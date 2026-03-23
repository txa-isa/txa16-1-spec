# 2. Register Model

## 2.1 General-Purpose Registers (GPR)

Eight 16-bit registers available to the programmer. All are architecturally equal.

| Register | Description               |
|----------|---------------------------|
| R0-R7    | General-purpose registers |

> **Calling convention (recommendation, not ISA requirement):**
> R0 — return value; R1-R3 — arguments; R4-R6 — callee-saved; R7 — scratch/temporary.

## 2.2 Special Registers

| Register | Size    | Description                                        |
|----------|---------|----------------------------------------------------|
| **PC**   | 16 bits | Program Counter — address of the current instruction |
| **SP**   | 16 bits | Stack Pointer — separate from GPR                  |
| **SR**   | 16 bits | Status Register — flags and mode bits              |

## 2.3 Status Register (SR)

```
Bit:  15  14  13  12  11  10   9   8   7   6   5   4   3   2   1   0
      [       Reserved (0)         ] [ IE ] [BANK] [SAT] [ 0   0 ] [OV] [N] [C] [Z]
```

| Bit  | Name       | Description                                          |
|------|------------|------------------------------------------------------|
| 0    | **Z**      | Zero — result is zero                                |
| 1    | **C**      | Carry — carry/borrow from the most significant bit   |
| 2    | **N**      | Negative — MSB of the result is 1                    |
| 3    | **OV**     | Overflow — signed overflow occurred                  |
| 4-5  | —          | Reserved, must be 0                                  |
| 6    | **SAT**    | Saturating arithmetic mode                           |
| 7    | **BANK**   | Active register bank (0 = User, 1 = IRQ)            |
| 8    | **IE**     | Interrupt Enable (1 = enabled, 0 = disabled)         |
| 9-15 | —          | Reserved, must be 0                                  |

### SAT (Saturating Mode)

When `SR[SAT] = 1`, arithmetic instructions (ADD, SUB, ADDI, SUBI) saturate instead of wrapping:

- Positive overflow → result clamped to `0x7FFF` (+32767)
- Negative overflow → result clamped to `0x8000` (-32768)

Flags C and OV are still set normally even when saturation occurs. MUL/MULH are unaffected by SAT mode.

### IE (Interrupt Enable)

When `SR[IE] = 0`, maskable interrupts (IRQ0-IRQ7) are not acknowledged. NMI is always acknowledged regardless of IE. On reset, IE = 0; software must explicitly enable interrupts after initialization.

## 2.4 Register Banks

TXA16-1 has **two banks** of 8 GPR registers each:

- **Bank 0 (User)** — active in normal mode (`SR[BANK] = 0`)
- **Bank 1 (IRQ)** — active during interrupt handling (`SR[BANK] = 1`)

Only R0-R7 are banked. PC, SP, and SR are **shared** across banks.

Bank switching occurs:
- **Automatically** on interrupt entry (hardware sets `SR[BANK] = 1`)
- **Automatically** on `RETI` (restores SR, including BANK bit)
- **Manually** via `WRSR` (software can manipulate `SR[BANK]` directly)

This allows interrupt handlers to use all 8 registers without saving/restoring the user context.

---

*[← Overview](01-overview.md) | [Next: Instruction Formats →](03-formats.md)*

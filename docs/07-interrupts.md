# 7. Interrupt System

## 7.1 Interrupt Vector Table

The IVT occupies the lowest 20 bytes of memory. Each entry is a 16-bit address (pointer to the handler).

| Address  | Vector | Description                     |
|----------|--------|---------------------------------|
| `0x0000` | RESET  | Reset vector (initial PC value) |
| `0x0002` | NMI    | Non-Maskable Interrupt          |
| `0x0004` | IRQ0   | Maskable interrupt 0            |
| `0x0006` | IRQ1   | Maskable interrupt 1            |
| `0x0008` | IRQ2   | Maskable interrupt 2            |
| `0x000A` | IRQ3   | Maskable interrupt 3            |
| `0x000C` | IRQ4   | Maskable interrupt 4            |
| `0x000E` | IRQ5   | Maskable interrupt 5            |
| `0x0010` | IRQ6   | Maskable interrupt 6            |
| `0x0012` | IRQ7   | Maskable interrupt 7            |

## 7.2 Reset Sequence

On power-on or hardware reset:

1. SR = `0x0000` (all flags clear, BANK = 0, IE = 0, SAT = 0)
2. SP = `0xFFFE`
3. PC = MEM16[`0x0000`] (load reset vector)
4. Execution begins at the loaded address

## 7.3 Interrupt Entry (Hardware)

When an interrupt is acknowledged (maskable with IE = 1, or NMI):

1. The current instruction completes
2. SP = SP - 2; MEM[SP] = PC_next (push return address)
3. SP = SP - 2; MEM[SP] = SR (push current status register)
4. SR[BANK] = 1 (switch to IRQ register bank)
5. SR[IE] = 0 (disable further maskable interrupts)
6. PC = MEM16[vector_address] (load handler address from IVT)

`PC_next` is the address of the instruction that would have executed next (the return point).

After entry, the stack contains:

```
        Higher addresses
        ┌──────────────┐
        │   PC_next    │  SP + 2
        ├──────────────┤
 SP --> │   SR_old     │  SP
        └──────────────┘
        Lower addresses
```

## 7.4 Interrupt Return

The `RETI` instruction reverses the entry sequence:

1. SR = MEM[SP]; SP = SP + 2 (pop and restore SR — restores BANK, IE, flags)
2. PC = MEM[SP]; SP = SP + 2 (pop and restore PC — returns to interrupted code)

RETI is **atomic**: no interrupt can be taken between the SR and PC restores.

## 7.5 Interrupt Priority

NMI has the highest priority and is always acknowledged, regardless of SR[IE].

Among maskable interrupts, **lower-numbered IRQs have higher priority**:

```
NMI > IRQ0 > IRQ1 > IRQ2 > IRQ3 > IRQ4 > IRQ5 > IRQ6 > IRQ7
```

When multiple interrupts are pending simultaneously, the highest-priority one is serviced first.

## 7.6 NMI Specifics

NMI (Non-Maskable Interrupt):
- Cannot be disabled via SR[IE]
- Uses the same entry mechanism as maskable interrupts
- Has priority over all maskable interrupts
- NMI during an NMI handler is **implementation-defined** (recommended: latch and defer until RETI)

## 7.7 Nested Interrupts

By default, maskable interrupts are disabled on entry (SR[IE] = 0). Nested interrupts are possible if the handler explicitly re-enables interrupts with `SEI`.

When using nested interrupts, the handler **must** save the SR and return address from the stack before calling SEI, because a new interrupt will overwrite the stack frame. The programmer is responsible for managing nested state.

## 7.8 Interrupt Latency

Minimum interrupt latency (from assertion to first handler instruction):

1. Complete current instruction (1 cycle, worst case varies by instruction)
2. Push PC (1 cycle)
3. Push SR (1 cycle)
4. Load vector (1 cycle)

**Minimum: 4 cycles** (plus current instruction completion).

---

*[← Memory Model](06-memory.md) | [Next: Flags →](08-flags.md)*

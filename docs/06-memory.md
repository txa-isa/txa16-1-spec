# 6. Memory Model

## 6.1 Address Space

TXA16-1 addresses **64 KB** of memory with 16-bit addresses (`0x0000`-`0xFFFF`). Memory is **byte-addressable**.

## 6.2 Byte Order

All multi-byte values are stored in **little-endian** order: the least significant byte is at the lower address.

Example: the value `0xABCD` stored at address `0x1000`:

| Address  | Byte   |
|----------|--------|
| `0x1000` | `0xCD` |
| `0x1001` | `0xAB` |

## 6.3 Alignment

- **Word** (16-bit) accesses via LDW, STW, PUSH, POP must be **2-byte aligned** (address is even).
- **Byte** accesses via LOADB, STOREB have no alignment restriction.
- Behavior on unaligned word access is **undefined** (implementation may trap, return incorrect data, or silently align the address).

## 6.4 Stack

The stack grows **downward** (from high addresses toward low addresses).

- `PUSH Rd`: SP = SP - 2; MEM[SP] = Rd
- `POP Rd`: Rd = MEM[SP]; SP = SP + 2

SP always points to the **topmost occupied** element. After reset, SP = `0xFFFE`.

```
        Higher addresses
        ┌──────────────┐
        │  (free)      │  SP + 4
        ├──────────────┤
        │  previous    │  SP + 2
        ├──────────────┤
 SP --> │  top of      │  SP
        │  stack       │
        ├──────────────┤
        │  (unused)    │
        Lower addresses
```

## 6.5 Memory Map

| Address Range     | Size     | Description                                  |
|-------------------|----------|----------------------------------------------|
| `0x0000`-`0x0013` | 20 bytes | Interrupt Vector Table (10 entries x 2 bytes) |
| `0x0014`-`0xFFFD` | ~65 KB   | General-purpose (code, data, stack)          |
| `0xFFFE`-`0xFFFF` | 2 bytes  | Initial SP location (top of stack)           |

The memory map describes the architectural convention. Implementations may define memory-mapped I/O regions within the general-purpose range. The IVT layout is described in detail in [Interrupt System](07-interrupts.md).

## 6.6 Memory Access Instructions

| Instruction | Width | Alignment | Description                   |
|-------------|-------|-----------|-------------------------------|
| LDW         | 16-bit| 2-byte    | Load word with signed offset  |
| STW         | 16-bit| 2-byte    | Store word with signed offset |
| LOADB       | 8-bit | None      | Load byte, zero-extend to 16  |
| STOREB      | 8-bit | None      | Store low byte of register    |
| PUSH        | 16-bit| 2-byte    | Push to stack (SP must be aligned) |
| POP         | 16-bit| 2-byte    | Pop from stack (SP must be aligned) |

---

*[← Instruction Set](05-instructions.md) | [Next: Interrupt System →](07-interrupts.md)*

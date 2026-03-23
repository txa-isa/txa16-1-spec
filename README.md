# TXA16-1 Specification

This repository contains the **formal specification** for TXA16-1 — one ISA from the **TXA (The XISC Architecture)** family.

TXA is a family of experimental instruction set architectures based on the XISC (eXtended ISC) philosophy. Different members of the family target different word sizes, use cases, and complexity tradeoffs while sharing common design principles.

**This repo covers only TXA16-1.** For other members of the TXA family, see their respective specification repositories.

## What is TXA16-1?

TXA16-1 is the 16-bit, revision 1 member of the family. It is a strict RISC core with fixed-width instructions and a load/store memory model, extended with a small number of "heavy" multi-operation instructions and a predication prefix for branchless conditional execution.

| Property         | Value                             |
|------------------|-----------------------------------|
| Word size        | 16 bits                           |
| Address space    | 64 KB, byte-addressable           |
| Byte order       | Little-endian                     |
| Registers        | 8 GPR + PC, SP, SR                |
| Instruction width| Fixed 16 bits                     |
| Register banks   | 2 (User, IRQ)                     |
| Instructions     | 51 across 8 encoding formats      |

### Highlights

- **COND predication prefix** — make up to 4 following instructions conditional without branching
- **Saturating arithmetic mode** — SR[SAT] toggles DSP-style clamping for ADD/SUB
- **Dual register banks** — automatic R0-R7 bank switch on interrupt, zero-overhead context save
- **MOVL/MOVH pair** — load any 16-bit constant in exactly 2 instructions
- **LDW/STW with offset** — signed 6-bit word-scaled offset for struct/stack access
- **8 maskable IRQs + NMI** — full interrupt system with vector table at 0x0000

### Opcode Map

```
0000  ALU       ADD, SUB, AND, OR, XOR, MUL, MULH, CMP
0001  ALU2      MOV, NEG, NOT, LOADB, STOREB
0010  ADDI      Add immediate
0011  SUBI      Subtract immediate
0100  ANDI      AND immediate
0101  ORI       OR immediate
0110  XORI      XOR immediate
0111  LDW       Load word [Rs + offset]
1000  STW       Store word [Rs + offset]
1001  SHIFT     SHL, SHR, SAR, ROL
1010  MOV-IMM   MOVL, MOVH
1011  Bcc       BEQ, BNE, BGT, BLT, BGE, BLE, BCS, BCC
1100  JMP/CALL  PC-relative jump and call
1101  SYS       PUSH, POP, JMP.R, CALL.R, RDSR, WRSR, RET, RETI
1110  CTL       NOP, HALT, SEI, CLI, SSAT, CSAT
1111  COND      Predication prefix
```

### Quick Example

```asm
; max(R1, R2) -> R0
    MOV   R0, R1
    CMP   R1, R2
    COND  LT, 1
      MOV R0, R2
```

## Specification

The full specification is in [`docs/`](docs/):

| Document | Description |
|----------|-------------|
| [Index](docs/00-index.md) | Table of contents |
| [Overview](docs/01-overview.md) | Architecture summary and design principles |
| [Registers](docs/02-registers.md) | GPR, special registers, SR layout, banks |
| [Formats](docs/03-formats.md) | 8 instruction encoding formats |
| [Opcode Map](docs/04-opcode-map.md) | Full opcode space allocation |
| [Instructions](docs/05-instructions.md) | Complete instruction descriptions |
| [Memory](docs/06-memory.md) | Address space, byte order, alignment, stack |
| [Interrupts](docs/07-interrupts.md) | Vector table, entry/return, priorities |
| [Flags](docs/08-flags.md) | Flag behavior and carry convention |
| [Pseudo-Instructions](docs/09-pseudo.md) | Assembler pseudo-instructions |
| [Assembly Syntax](docs/10-assembly.md) | Syntax conventions and directives |
| [Encoding Reference](docs/11-encoding-reference.md) | Compact encoding tables |
| [Examples](docs/12-examples.md) | Example programs |

## License

This work is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

## Author

**AnmiTaliDev**

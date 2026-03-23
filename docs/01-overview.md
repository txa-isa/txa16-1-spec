# 1. Overview

**TXA16-1** (The XISC Architecture, 16-bit, revision 1) is an experimental 16-bit instruction set architecture based on the **XISC** (eXtended ISC) model.

XISC philosophy: a strict RISC foundation (fixed-width instructions, load/store memory model, simple decoder) augmented with a small number of "heavy" instructions where they provide significant convenience without substantial implementation complexity.

## Key Parameters

| Parameter              | Value                              |
|------------------------|------------------------------------|
| Word size              | 16 bits                            |
| Instruction width      | Fixed 16 bits                      |
| Address space          | 64 KB (byte-addressable)           |
| GPR count              | 8 x 16-bit (R0-R7)                |
| Special registers      | PC, SP, SR                         |
| Register banks         | 2 (User, IRQ)                      |
| Byte order             | Little-endian                      |
| Alignment              | Word accesses must be 2-byte aligned |

## Design Principles

- **Fixed-width encoding** — all instructions are exactly 16 bits. No variable-length decoding.
- **Load/store architecture** — only dedicated memory instructions access RAM. ALU operates on registers.
- **Orthogonal registers** — all 8 GPR are architecturally equal. No zero register, no hardwired values.
- **Heavy instructions** — CALL, RET, RETI perform multiple micro-operations in a single instruction. Marked as "heavy" in the ISA.
- **Predication prefix** — the COND instruction makes up to 4 following instructions conditional, eliminating short branches.
- **Saturating arithmetic** — a mode bit in SR enables clamping instead of wrap-around overflow.
- **Dual register banks** — automatic bank switching on interrupt entry provides zero-overhead context save for IRQ handlers.

---

*[Next: Register Model →](02-registers.md)*

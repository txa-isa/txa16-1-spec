# 8. Flag Behavior

## 8.1 Flag-Setting Instructions

| Instruction Group        | Z | C | N | OV |
|--------------------------|---|---|---|----|
| ADD, SUB, ADDI, SUBI     | * | * | * | *  |
| CMP                      | * | * | * | *  |
| NEG                      | * | * | * | *  |
| AND, OR, XOR (reg & imm) | * | 0 | * | 0  |
| NOT                      | * | 0 | * | 0  |
| MUL, MULH                | * | 0 | * | 0  |
| SHL, SHR, SAR, ROL       | * | * | * | 0  |

Legend: `*` = set according to the result, `0` = cleared to zero.

### Flag definitions

- **Z** (Zero): set if the 16-bit result is `0x0000`
- **C** (Carry): set if there is a carry out of bit 15 (addition) or no borrow (subtraction)
- **N** (Negative): set if bit 15 of the result is 1
- **OV** (Overflow): set if signed overflow occurs (sign of result is inconsistent with signs of operands)

### Shift flag details

- **SHL**: C = last bit shifted out (original bit `16 - amount`)
- **SHR**: C = last bit shifted out (original bit `amount - 1`)
- **SAR**: C = last bit shifted out (original bit `amount - 1`)
- **ROL**: C = last bit rotated (original bit `16 - amount`)
- For amount = 0: C is **unchanged**, Z and N are set from the (unchanged) value

## 8.2 Flag-Neutral Instructions

The following instructions do **not** modify any flags:

| Category        | Instructions                                      |
|-----------------|---------------------------------------------------|
| Data movement   | MOV (register), MOVL, MOVH                       |
| Memory access   | LDW, STW, LOADB, STOREB                          |
| Stack           | PUSH, POP                                         |
| Control flow    | Bcc, JMP, CALL, JMP.R, CALL.R, RET              |
| System          | RDSR, WRSR, NOP, HALT, SEI, CLI, SSAT, CSAT     |
| Predication     | COND                                              |

Note: **RETI** restores the entire SR from the stack, which includes all flags. This is not "modifying" flags — it is restoring the interrupted context.

## 8.3 Carry Convention for Subtraction

For SUB, SUBI, and CMP, the C flag follows the **no-borrow** convention:

- **C = 1** if the subtraction did **not** borrow (unsigned: `Rs1 >= Rs2`)
- **C = 0** if the subtraction **did** borrow (unsigned: `Rs1 < Rs2`)

This convention (matching ARM) allows the branch conditions to work correctly:

| Branch | Condition | Meaning after CMP       |
|--------|-----------|-------------------------|
| BCS    | C = 1     | Unsigned Rs1 >= Rs2     |
| BCC    | C = 0     | Unsigned Rs1 < Rs2      |

For unsigned greater-than: check C = 1 and Z = 0 (no dedicated branch, use COND or two branches).

## 8.4 Signed Comparison Logic

The signed comparison conditions used in Bcc and COND:

| Condition | Flags tested   | True when (after CMP A, B) |
|-----------|----------------|---------------------------|
| EQ        | Z = 1          | A == B                    |
| NE        | Z = 0          | A != B                    |
| GT        | Z=0 and N=OV   | A > B (signed)            |
| LT        | N != OV        | A < B (signed)            |
| GE        | N = OV         | A >= B (signed)           |
| LE        | Z=1 or N!=OV   | A <= B (signed)           |
| CS        | C = 1          | A >= B (unsigned)         |
| CC        | C = 0          | A < B (unsigned)          |

---

*[← Interrupt System](07-interrupts.md) | [Next: Pseudo-Instructions →](09-pseudo.md)*

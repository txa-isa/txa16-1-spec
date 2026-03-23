# 4. Opcode Map

All 16 opcode slots are allocated. No reserved opcodes remain at the top level; future expansion uses reserved `func` codes within existing groups.

| Opcode | Binary | Name     | Format | Description                          |
|--------|--------|----------|--------|--------------------------------------|
| 0      | `0000` | ALU      | R      | 3-operand ALU operations             |
| 1      | `0001` | ALU2     | R      | 2-operand ALU and byte memory access |
| 2      | `0010` | ADDI     | I      | Add unsigned immediate               |
| 3      | `0011` | SUBI     | I      | Subtract unsigned immediate          |
| 4      | `0100` | ANDI     | I      | AND unsigned immediate               |
| 5      | `0101` | ORI      | I      | OR unsigned immediate                |
| 6      | `0110` | XORI     | I      | XOR unsigned immediate               |
| 7      | `0111` | LDW      | I      | Load word with signed offset         |
| 8      | `1000` | STW      | I      | Store word with signed offset        |
| 9      | `1001` | SHIFT    | I      | Shift/rotate with immediate amount   |
| 10     | `1010` | MOV-IMM  | S      | MOVL / MOVH                          |
| 11     | `1011` | Bcc      | B      | Conditional branch                   |
| 12     | `1100` | JMP/CALL | L      | PC-relative jump or call             |
| 13     | `1101` | SYS      | SYS    | Stack, indirect jumps, SR access     |
| 14     | `1110` | CTL      | CTL    | NOP, HALT, interrupt/SAT control     |
| 15     | `1111` | COND     | P      | Predication prefix                   |

## Sub-opcode Tables

### ALU (opcode `0000`) — `func` field

| func  | Instruction |
|-------|-------------|
| `000` | ADD         |
| `001` | SUB         |
| `010` | AND         |
| `011` | OR          |
| `100` | XOR         |
| `101` | MUL         |
| `110` | MULH        |
| `111` | CMP         |

### ALU2 (opcode `0001`) — `func` field

| func  | Instruction | Status   |
|-------|-------------|----------|
| `000` | MOV         | Active   |
| `001` | NEG         | Active   |
| `010` | NOT         | Active   |
| `011` | LOADB       | Active   |
| `100` | STOREB      | Active   |
| `101` | —           | Reserved |
| `110` | —           | Reserved |
| `111` | —           | Reserved |

### SHIFT (opcode `1001`) — `type` field (imm6 bits 5:4)

| type | Instruction |
|------|-------------|
| `00` | SHL         |
| `01` | SHR         |
| `10` | SAR         |
| `11` | ROL         |

### SYS (opcode `1101`) — `func` field

| func  | Instruction |
|-------|-------------|
| `000` | PUSH        |
| `001` | POP         |
| `010` | JMP.R       |
| `011` | CALL.R      |
| `100` | RDSR        |
| `101` | WRSR        |
| `110` | RET         |
| `111` | RETI        |

### CTL (opcode `1110`) — `func` field

| func  | Instruction | Status   |
|-------|-------------|----------|
| `000` | NOP         | Active   |
| `001` | HALT        | Active   |
| `010` | SEI         | Active   |
| `011` | CLI         | Active   |
| `100` | SSAT        | Active   |
| `101` | CSAT        | Active   |
| `110` | —           | Reserved |
| `111` | —           | Reserved |

---

*[← Instruction Formats](03-formats.md) | [Next: Instruction Set →](05-instructions.md)*

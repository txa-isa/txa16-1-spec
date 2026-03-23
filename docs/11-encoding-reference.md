# 11. Encoding Reference

Compact reference for all instruction encodings. Bit 15 is the MSB (leftmost), bit 0 is the LSB.

## 11.1 ALU (Opcode `0000`) — R-type

```
[0000] [Rd:3] [Rs1:3] [Rs2:3] [func:3]
```

| func | Instruction | Assembly            |
|------|-------------|---------------------|
| 000  | ADD         | `ADD Rd, Rs1, Rs2`  |
| 001  | SUB         | `SUB Rd, Rs1, Rs2`  |
| 010  | AND         | `AND Rd, Rs1, Rs2`  |
| 011  | OR          | `OR  Rd, Rs1, Rs2`  |
| 100  | XOR         | `XOR Rd, Rs1, Rs2`  |
| 101  | MUL         | `MUL Rd, Rs1, Rs2`  |
| 110  | MULH        | `MULH Rd, Rs1, Rs2` |
| 111  | CMP         | `CMP Rs1, Rs2`      |

## 11.2 ALU2 (Opcode `0001`) — R-type

```
[0001] [Rd:3] [Rs1:3] [Rs2:3] [func:3]
```

| func | Instruction | Assembly            | Unused fields  |
|------|-------------|---------------------|----------------|
| 000  | MOV         | `MOV Rd, Rs1`       | Rs2 = 000      |
| 001  | NEG         | `NEG Rd, Rs1`       | Rs2 = 000      |
| 010  | NOT         | `NOT Rd, Rs1`       | Rs2 = 000      |
| 011  | LOADB       | `LOADB Rd, [Rs1]`   | Rs2 = 000      |
| 100  | STOREB      | `STOREB [Rs1], Rs2` | Rd = 000       |

## 11.3 Immediate ALU (Opcodes `0010`-`0110`) — I-type

```
[opcode:4] [Rd:3] [Rs:3] [imm6:6]
```

| Opcode | Instruction | Assembly             | imm6       |
|--------|-------------|----------------------|------------|
| 0010   | ADDI        | `ADDI Rd, Rs, #imm`  | unsigned 0-63 |
| 0011   | SUBI        | `SUBI Rd, Rs, #imm`  | unsigned 0-63 |
| 0100   | ANDI        | `ANDI Rd, Rs, #imm`  | unsigned 0-63 |
| 0101   | ORI         | `ORI  Rd, Rs, #imm`  | unsigned 0-63 |
| 0110   | XORI        | `XORI Rd, Rs, #imm`  | unsigned 0-63 |

## 11.4 Load/Store Word (Opcodes `0111`-`1000`) — I-type

```
[opcode:4] [Rd:3] [Rs:3] [imm6:6]
```

| Opcode | Instruction | Assembly               | imm6              |
|--------|-------------|------------------------|-------------------|
| 0111   | LDW         | `LDW Rd, [Rs, #off]`   | signed, *2 scale |
| 1000   | STW         | `STW Rd, [Rs, #off]`   | signed, *2 scale |

Effective address = Rs + sign_extend(imm6) * 2. Byte offset range: -64 to +62.

## 11.5 Shift (Opcode `1001`) — I-type (modified)

```
[1001] [Rd:3] [Rs:3] [type:2] [amount:4]
```

| type | Instruction | Assembly             |
|------|-------------|----------------------|
| 00   | SHL         | `SHL Rd, Rs, #amt`   |
| 01   | SHR         | `SHR Rd, Rs, #amt`   |
| 10   | SAR         | `SAR Rd, Rs, #amt`   |
| 11   | ROL         | `ROL Rd, Rs, #amt`   |

Amount: unsigned 0-15.

## 11.6 Move Immediate (Opcode `1010`) — S-type

```
[1010] [Rd:3] [H:1] [imm8:8]
```

| H | Instruction | Assembly           |
|---|-------------|--------------------|
| 0 | MOVL        | `MOVL Rd, #imm8`   |
| 1 | MOVH        | `MOVH Rd, #imm8`   |

imm8: unsigned 0-255.

## 11.7 Conditional Branch (Opcode `1011`) — B-type

```
[1011] [cc:3] [offset9:9]
```

| cc  | Instruction | Condition              |
|-----|-------------|------------------------|
| 000 | BEQ         | Z = 1                  |
| 001 | BNE         | Z = 0                  |
| 010 | BGT         | Z = 0 and N = OV       |
| 011 | BLT         | N != OV                |
| 100 | BGE         | N = OV                 |
| 101 | BLE         | Z = 1 or N != OV       |
| 110 | BCS         | C = 1                  |
| 111 | BCC         | C = 0                  |

Target = (PC + 2) + sign_extend(offset9) * 2. Byte range: -512 to +510.

## 11.8 Jump/Call (Opcode `1100`) — L-type

```
[1100] [T:1] [offset11:11]
```

| T | Instruction | Assembly         |
|---|-------------|------------------|
| 0 | JMP         | `JMP #offset`    |
| 1 | CALL        | `CALL #offset`   |

Target = (PC + 2) + sign_extend(offset11) * 2. Byte range: -2048 to +2046.

## 11.9 System (Opcode `1101`) — SYS-type

```
[1101] [Rd:3] [func:3] [000000]
```

| func | Instruction | Assembly     | Rd usage     |
|------|-------------|--------------|--------------|
| 000  | PUSH        | `PUSH Rd`    | Source       |
| 001  | POP         | `POP Rd`     | Destination  |
| 010  | JMP.R       | `JMP.R Rd`   | Target addr  |
| 011  | CALL.R      | `CALL.R Rd`  | Target addr  |
| 100  | RDSR        | `RDSR Rd`    | Destination  |
| 101  | WRSR        | `WRSR Rd`    | Source       |
| 110  | RET         | `RET`        | Ignored (000)|
| 111  | RETI        | `RETI`       | Ignored (000)|

## 11.10 Control (Opcode `1110`) — CTL-type

```
[1110] [func:3] [000000000]
```

| func | Instruction | Binary encoding      |
|------|-------------|----------------------|
| 000  | NOP         | `1110 000 000000000` |
| 001  | HALT        | `1110 001 000000000` |
| 010  | SEI         | `1110 010 000000000` |
| 011  | CLI         | `1110 011 000000000` |
| 100  | SSAT        | `1110 100 000000000` |
| 101  | CSAT        | `1110 101 000000000` |

## 11.11 COND Prefix (Opcode `1111`) — P-type

```
[1111] [cc:3] [cnt:2] [0000000]
```

| cnt | Predicated count | Assembly example    |
|-----|------------------|---------------------|
| 00  | 1 instruction    | `COND EQ, 1`       |
| 01  | 2 instructions   | `COND NE, 2`       |
| 10  | 3 instructions   | `COND GT, 3`       |
| 11  | 4 instructions   | `COND LT, 4`       |

Condition codes: same as Bcc (Section 11.7).

---

*[← Assembly Syntax](10-assembly.md) | [Next: Examples →](12-examples.md)*

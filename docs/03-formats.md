# 3. Instruction Formats

All instructions are exactly **16 bits** wide. The ISA uses 8 encoding formats, determined by the 4-bit opcode.

## 3.1 R-type — Register-Register

Used by ALU and ALU2 instructions.

```
15      12  11    9   8     6   5     3   2     0
[ opcode  ] [  Rd   ] [ Rs1   ] [ Rs2   ] [ func  ]
    4          3          3          3          3
```

- **Rd** — destination register
- **Rs1** — first source register
- **Rs2** — second source register (unused in 2-operand instructions)
- **func** — operation selector within the opcode group

## 3.2 I-type — Immediate

Used by arithmetic/logic immediates, load/store with offset, and shifts.

```
15      12  11    9   8     6   5              0
[ opcode  ] [  Rd   ] [  Rs   ] [    imm6       ]
    4          3          3            6
```

- **Rd** — destination register
- **Rs** — source register
- **imm6** — 6-bit immediate (interpretation depends on instruction)

## 3.3 S-type — Short Immediate

Used by MOVL/MOVH.

```
15      12  11    9   8      7              0
[ opcode  ] [  Rd   ] [ H  ] [    imm8       ]
    4          3        1           8
```

- **Rd** — destination register
- **H** — high/low selector: 0 = MOVL, 1 = MOVH
- **imm8** — 8-bit unsigned immediate

## 3.4 B-type — Branch

Used by conditional branches (Bcc).

```
15      12  11    9   8                    0
[ opcode  ] [  cc   ] [     offset9          ]
    4          3              9
```

- **cc** — condition code (3 bits, 8 conditions)
- **offset9** — signed 9-bit word offset

Target address: `(PC + 2) + sign_extend(offset9) * 2`

## 3.5 L-type — Long Jump

Used by JMP and CALL (PC-relative).

```
15      12  11    10                       0
[ opcode  ] [ T  ] [       offset11          ]
    4         1             11
```

- **T** — type: 0 = JMP, 1 = CALL
- **offset11** — signed 11-bit word offset

Target address: `(PC + 2) + sign_extend(offset11) * 2`

## 3.6 SYS-type — System

Used by stack, register-indirect control flow, and SR access instructions.

```
15      12  11    9   8     6   5              0
[ opcode  ] [  Rd   ] [ func  ] [   unused (0)  ]
    4          3          3            6
```

- **Rd** — register operand (source or destination depending on func)
- **func** — operation selector
- **unused** — must be `000000`

## 3.7 CTL-type — Control

Used by NOP, HALT, and SR bit manipulation.

```
15      12  11    9   8                    0
[ opcode  ] [ func  ] [     unused (0)      ]
    4          3              9
```

- **func** — operation selector
- **unused** — must be `000000000`

## 3.8 P-type — Predication Prefix

Used by the COND prefix instruction.

```
15      12  11    9   8     7   6              0
[ opcode  ] [  cc   ] [ cnt  ] [   unused (0)  ]
    4          3         2            7
```

- **cc** — condition code (same encoding as B-type)
- **cnt** — number of predicated instructions minus 1 (0-3 → 1-4 instructions)
- **unused** — must be `0000000`

## 3.9 Format Summary

| Format | Opcodes     | Fields                     | Total bits |
|--------|-------------|----------------------------|------------|
| R      | 0000, 0001  | opcode, Rd, Rs1, Rs2, func | 4+3+3+3+3  |
| I      | 0010-1001   | opcode, Rd, Rs, imm6       | 4+3+3+6    |
| S      | 1010        | opcode, Rd, H, imm8        | 4+3+1+8    |
| B      | 1011        | opcode, cc, offset9        | 4+3+9      |
| L      | 1100        | opcode, T, offset11        | 4+1+11     |
| SYS    | 1101        | opcode, Rd, func, unused   | 4+3+3+6    |
| CTL    | 1110        | opcode, func, unused       | 4+3+9      |
| P      | 1111        | opcode, cc, cnt, unused    | 4+3+2+7    |

---

*[← Registers](02-registers.md) | [Next: Opcode Map →](04-opcode-map.md)*

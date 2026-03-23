# 9. Pseudo-Instructions

The assembler should recognize the following pseudo-instructions and expand them into real instructions:

| Pseudo-instruction     | Expansion                     | Notes                              |
|------------------------|-------------------------------|------------------------------------|
| `MOV Rd, Rs`           | ALU2 MOV (func `000`)        | Real instruction, no flag change   |
| `NOP`                  | CTL func `000`                | Real instruction                   |
| `LOAD Rd, [Rs]`        | `LDW Rd, [Rs, #0]`           | Word load, zero offset             |
| `STORE [Rs], Rd`       | `STW Rd, [Rs, #0]`           | Word store, zero offset            |
| `ROR Rd, Rs, #n`       | `ROL Rd, Rs, #(16 - n)`      | Rotate right via rotate left       |
| `INC Rd`               | `ADDI Rd, Rd, #1`            | Increment by 1                     |
| `DEC Rd`               | `SUBI Rd, Rd, #1`            | Decrement by 1                     |
| `CLR Rd`               | `MOVL Rd, #0`                | Clear register to zero             |
| `TST Rs`               | `CMP Rs, Rs`                 | Test if zero/negative (sets Z, N)  |
| `BRA offset`           | `JMP offset`                 | Alias for unconditional jump       |
| `BHI offset`           | `BCS offset`                 | Alias: unsigned higher or same     |
| `BLO offset`           | `BCC offset`                 | Alias: unsigned lower              |
| `BZ offset`            | `BEQ offset`                 | Alias: branch if zero              |
| `BNZ offset`           | `BNE offset`                 | Alias: branch if not zero          |

## Notes

**MOV and NOP** are listed here for completeness but are real instructions with dedicated encodings. The assembler maps them directly.

**TST Rs** uses `CMP Rs, Rs` which computes `Rs - Rs = 0`. This always sets Z = 1 and C = 1, with N = 0 and OV = 0. To test a register value for zero or sign, use `SUBI R7, Rs, #0` (which sets Z and N based on the value without subtracting anything) or `AND Rs, Rs, Rs` (which also sets Z and N, and clears C and OV). The recommended expansion depends on which flags are needed.

**Immediate compare** (`CMP Rs, #imm`) is not a single pseudo-instruction because it requires a scratch register. The recommended pattern:

```asm
; Compare R0 with constant 42
SUBI  R7, R0, #42    ; R7 = R0 - 42, sets flags (clobbers R7)
BEQ   equal_label    ; branch if R0 == 42
```

For constants outside the 0-63 range:

```asm
MOVL  R7, #0xAB
MOVH  R7, #0xCD      ; R7 = 0xCDAB
CMP   R0, R7         ; compare R0 with 0xCDAB
```

---

*[← Flags](08-flags.md) | [Next: Assembly Syntax →](10-assembly.md)*

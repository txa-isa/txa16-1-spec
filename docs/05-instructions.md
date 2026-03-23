# 5. Instruction Set

## 5.1 ALU — 3-Operand Register (Opcode `0000`)

R-type format: `[0000][Rd:3][Rs1:3][Rs2:3][func:3]`

| func  | Mnemonic | Operation             | Flags         |
|-------|----------|-----------------------|---------------|
| `000` | ADD      | Rd = Rs1 + Rs2        | Z, C, N, OV   |
| `001` | SUB      | Rd = Rs1 - Rs2        | Z, C, N, OV   |
| `010` | AND      | Rd = Rs1 & Rs2        | Z, N; C=0 OV=0|
| `011` | OR       | Rd = Rs1 \| Rs2       | Z, N; C=0 OV=0|
| `100` | XOR      | Rd = Rs1 ^ Rs2        | Z, N; C=0 OV=0|
| `101` | MUL      | Rd = (Rs1 * Rs2)[15:0]| Z, N; C=0 OV=0|
| `110` | MULH     | Rd = (Rs1 * Rs2)[31:16]| Z, N; C=0 OV=0|
| `111` | CMP      | flags(Rs1 - Rs2)      | Z, C, N, OV   |

**ADD/SUB** are affected by SAT mode.

**CMP** computes Rs1 - Rs2 and sets flags but does not write the result. Rd field should be `000` in the encoding.

**MUL** produces the lower 16 bits of the unsigned 32-bit product. **MULH** produces the upper 16 bits of the signed 32-bit product. To obtain a full 32-bit result, use both instructions. MUL and MULH are not affected by SAT mode.

## 5.2 ALU2 — 2-Operand and Byte Memory (Opcode `0001`)

R-type format: `[0001][Rd:3][Rs1:3][Rs2:3][func:3]`

| func  | Mnemonic | Operation                     | Flags          |
|-------|----------|-------------------------------|----------------|
| `000` | MOV      | Rd = Rs1                      | (none)         |
| `001` | NEG      | Rd = -Rs1 (two's complement)  | Z, C, N, OV    |
| `010` | NOT      | Rd = ~Rs1 (bitwise complement)| Z, N; C=0 OV=0 |
| `011` | LOADB    | Rd = zero_extend(MEM[Rs1], 8) | (none)         |
| `100` | STOREB   | MEM[Rs1] = Rs2[7:0]          | (none)         |

**MOV** copies Rs1 to Rd without modifying flags. Rs2 is ignored (should be `000`).

**NEG** computes the two's complement of Rs1. Rs2 is ignored.

**NOT** computes the bitwise complement of Rs1. Rs2 is ignored.

**LOADB** reads a single byte from memory at the address in Rs1 and zero-extends it to 16 bits. No alignment restriction. Rs2 is ignored.

**STOREB** writes the low byte of Rs2 to memory at the address in Rs1. No alignment restriction. Rd is ignored (should be `000`).

## 5.3 Arithmetic Immediate (Opcodes `0010`-`0011`)

I-type format: `[opcode:4][Rd:3][Rs:3][imm6:6]`

| Opcode | Mnemonic | Operation               | Flags       |
|--------|----------|-------------------------|-------------|
| `0010` | ADDI     | Rd = Rs + zero_ext(imm6) | Z, C, N, OV |
| `0011` | SUBI     | Rd = Rs - zero_ext(imm6) | Z, C, N, OV |

`imm6` is an **unsigned** 6-bit value (range 0-63). ADDI and SUBI together cover the range -63 to +63 without overlap.

ADDI and SUBI are affected by SAT mode.

## 5.4 Logic Immediate (Opcodes `0100`-`0110`)

I-type format: `[opcode:4][Rd:3][Rs:3][imm6:6]`

| Opcode | Mnemonic | Operation                | Flags          |
|--------|----------|--------------------------|----------------|
| `0100` | ANDI     | Rd = Rs & zero_ext(imm6)  | Z, N; C=0 OV=0 |
| `0101` | ORI      | Rd = Rs \| zero_ext(imm6) | Z, N; C=0 OV=0 |
| `0110` | XORI     | Rd = Rs ^ zero_ext(imm6)  | Z, N; C=0 OV=0 |

`imm6` is an **unsigned** 6-bit value (range 0-63).

## 5.5 Load/Store Word with Offset (Opcodes `0111`-`1000`)

I-type format: `[opcode:4][Rd:3][Rs:3][imm6:6]`

| Opcode | Mnemonic | Operation                           |
|--------|----------|-------------------------------------|
| `0111` | LDW      | Rd = MEM16[Rs + sign_ext(imm6) * 2] |
| `1000` | STW      | MEM16[Rs + sign_ext(imm6) * 2] = Rd |

`imm6` is a **signed** 6-bit value (range -32 to +31). The offset is scaled by 2 (word size), giving an effective byte offset range of **-64 to +62**.

The effective address must be 2-byte aligned. Behavior on unaligned access is **undefined**.

LDW and STW do **not** modify flags.

> **Pseudo-instructions:**
> - `LOAD Rd, [Rs]` assembles as `LDW Rd, [Rs, #0]`
> - `STORE [Rs], Rd` assembles as `STW Rd, [Rs, #0]`

## 5.6 Shifts and Rotates (Opcode `1001`)

I-type format: `[1001][Rd:3][Rs:3][type:2][amount:4]`

The 6-bit immediate encodes both shift type and amount:

```
imm6 = [ type (2 bits) ] [ amount (4 bits) ]
         bits 5:4            bits 3:0
```

| type | Mnemonic | Operation                              |
|------|----------|----------------------------------------|
| `00` | SHL      | Rd = Rs << amount (logical left)       |
| `01` | SHR      | Rd = Rs >>> amount (logical right)     |
| `10` | SAR      | Rd = Rs >> amount (arithmetic right)   |
| `11` | ROL      | Rd = Rs rotl amount (rotate left)      |

`amount` is an unsigned 4-bit value (0-15).

**Flags:** Z and N are set based on the result. C is set to the last bit shifted/rotated out. OV is cleared. For amount = 0: result is unchanged, Z and N set from the value, C is unchanged.

> **Pseudo-instruction:** `ROR Rd, Rs, #n` assembles as `ROL Rd, Rs, #(16 - n)`.

## 5.7 Move Immediate (Opcode `1010`)

S-type format: `[1010][Rd:3][H:1][imm8:8]`

| H | Mnemonic | Operation                                  |
|---|----------|--------------------------------------------|
| 0 | MOVL     | Rd[7:0] = imm8; Rd[15:8] = 0x00           |
| 1 | MOVH     | Rd[15:8] = imm8; Rd[7:0] unchanged        |

`imm8` is an unsigned 8-bit value (0-255).

MOVL and MOVH do **not** modify flags.

**Loading a full 16-bit constant** requires two instructions:

```asm
MOVL  R0, #0xCD      ; R0 = 0x00CD
MOVH  R0, #0xAB      ; R0 = 0xABCD
```

**Loading a small constant** (0-255) requires one instruction:

```asm
MOVL  R0, #42        ; R0 = 0x002A
```

## 5.8 Conditional Branch (Opcode `1011`)

B-type format: `[1011][cc:3][offset9:9]`

| cc    | Mnemonic | Condition             | Description             |
|-------|----------|-----------------------|-------------------------|
| `000` | BEQ      | Z = 1                 | Equal / zero            |
| `001` | BNE      | Z = 0                 | Not equal / not zero    |
| `010` | BGT      | Z = 0 and N = OV      | Signed greater than     |
| `011` | BLT      | N != OV               | Signed less than        |
| `100` | BGE      | N = OV                | Signed greater or equal |
| `101` | BLE      | Z = 1 or N != OV      | Signed less or equal    |
| `110` | BCS      | C = 1                 | Carry set / unsigned >= |
| `111` | BCC      | C = 0                 | Carry clear / unsigned < |

`offset9` is a signed 9-bit value. Target address:

```
target = (PC + 2) + sign_extend(offset9) * 2
```

Range: **-512 to +510 bytes** relative to the next instruction.

Branches do **not** modify flags.

> **Aliases:** `BHI` = `BCS`, `BLO` = `BCC`, `BZ` = `BEQ`, `BNZ` = `BNE`

## 5.9 Jump and Call (Opcode `1100`)

L-type format: `[1100][T:1][offset11:11]`

| T | Mnemonic | Operation                                     |
|---|----------|-----------------------------------------------|
| 0 | JMP      | PC = target                                   |
| 1 | CALL     | SP -= 2; MEM[SP] = PC + 2; PC = target        |

`offset11` is a signed 11-bit value. Target address:

```
target = (PC + 2) + sign_extend(offset11) * 2
```

Range: **-2048 to +2046 bytes** relative to the next instruction.

JMP and CALL do **not** modify flags.

CALL is a **heavy instruction**: it performs a stack push and a jump in a single instruction.

> For longer jumps, use register-indirect variants JMP.R and CALL.R (Section 5.10).

## 5.10 System Instructions (Opcode `1101`)

SYS-type format: `[1101][Rd:3][func:3][000000]`

| func  | Mnemonic | Operation                                 | Heavy? |
|-------|----------|-------------------------------------------|--------|
| `000` | PUSH     | SP -= 2; MEM[SP] = Rd                     | No     |
| `001` | POP      | Rd = MEM[SP]; SP += 2                     | No     |
| `010` | JMP.R    | PC = Rd                                   | No     |
| `011` | CALL.R   | SP -= 2; MEM[SP] = PC + 2; PC = Rd       | Yes    |
| `100` | RDSR     | Rd = SR                                   | No     |
| `101` | WRSR     | SR = Rd                                   | No     |
| `110` | RET      | PC = MEM[SP]; SP += 2                     | Yes    |
| `111` | RETI     | SR = MEM[SP]; SP += 2; PC = MEM[SP]; SP += 2 | Yes |

**PUSH/POP:** The stack grows downward. SP points to the topmost occupied slot. PUSH decrements SP by 2 then writes. POP reads then increments SP by 2.

**JMP.R / CALL.R:** Register-indirect jump/call. Used for long-range control flow (load target into Rd with MOVL/MOVH, then JMP.R).

**RDSR / WRSR:** Read and write the Status Register. These are the only way to directly access SR from software (besides RETI which restores SR from the stack).

**RET:** Pops the return address from the stack into PC.

**RETI:** Atomically restores SR and PC from the stack, in that order (SR is on top, PC below). This restores the BANK, IE, SAT, and flag bits to their pre-interrupt state.

For RET and RETI, the Rd field is ignored (should be `000`).

None of the system instructions modify flags directly (RETI restores the entire SR, including flags).

## 5.11 Control Instructions (Opcode `1110`)

CTL-type format: `[1110][func:3][000000000]`

| func  | Mnemonic | Operation                        |
|-------|----------|----------------------------------|
| `000` | NOP      | No operation                     |
| `001` | HALT     | Stop the processor clock         |
| `010` | SEI      | SR[IE] = 1 (enable interrupts)   |
| `011` | CLI      | SR[IE] = 0 (disable interrupts)  |
| `100` | SSAT     | SR[SAT] = 1 (enable saturation)  |
| `101` | CSAT     | SR[SAT] = 0 (disable saturation) |

**HALT** stops the processor. Execution resumes at the next instruction when an interrupt is received (if IE is enabled, or on NMI).

None of the control instructions modify flags.

## 5.12 Predication Prefix — COND (Opcode `1111`)

P-type format: `[1111][cc:3][cnt:2][0000000]`

**Condition codes (`cc`):** Same encoding as Bcc (Section 5.8).

| cc    | Mnemonic | Condition             |
|-------|----------|-----------------------|
| `000` | EQ       | Z = 1                 |
| `001` | NE       | Z = 0                 |
| `010` | GT       | Z = 0 and N = OV      |
| `011` | LT       | N != OV               |
| `100` | GE       | N = OV                |
| `101` | LE       | Z = 1 or N != OV      |
| `110` | CS       | C = 1                 |
| `111` | CC       | C = 0                 |

**Count field (`cnt`):**

| cnt  | Predicated instructions |
|------|------------------------|
| `00` | 1                      |
| `01` | 2                      |
| `10` | 3                      |
| `11` | 4                      |

### Semantics

The condition is evaluated once, when COND is decoded. The following `cnt` instructions are then either:
- **Executed normally** if the condition is true, or
- **Treated as NOPs** (skipped) if the condition is false.

In both cases, PC advances past all predicated instructions. The block has a predictable, fixed length.

### Restrictions

- COND **cannot** be nested (a COND within a predicated block is undefined behavior).
- Branch/jump instructions within a predicated block are permitted but require care.
- COND itself does **not** modify flags.

### Example

```asm
; if (R0 == R1) { R2 = R3; R4 = R5; }
CMP   R0, R1           ; compare, sets flags
COND  EQ, 2            ; next 2 instructions are conditional
  MOV R2, R3           ; executed only if Z = 1
  MOV R4, R5           ; executed only if Z = 1
```

---

*[← Opcode Map](04-opcode-map.md) | [Next: Memory Model →](06-memory.md)*

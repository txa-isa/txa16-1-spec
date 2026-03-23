# 10. Assembly Syntax

## 10.1 General Syntax

```
[label:]  MNEMONIC  [operands]   [; comment]
```

- Labels end with a colon and represent addresses
- Immediate values are prefixed with `#`
- Memory operands use bracket notation: `[Rs]`, `[Rs, #offset]`
- Register names: `R0`-`R7`, `SP`, `SR`, `PC`
- Mnemonics and register names are case-insensitive
- Comments start with `;` and extend to end of line

## 10.2 Operand Syntax

| Syntax             | Meaning                           | Example             |
|--------------------|-----------------------------------|---------------------|
| `Rd, Rs1, Rs2`     | Three registers                   | `ADD R0, R1, R2`    |
| `Rd, Rs`           | Two registers                     | `MOV R0, R1`        |
| `Rd, Rs, #imm`     | Register and immediate            | `ADDI R0, R1, #5`   |
| `Rd, #imm`         | Register and immediate            | `MOVL R0, #0xFF`    |
| `Rd, [Rs]`         | Register and memory indirect      | `LOAD R0, [R1]`     |
| `Rd, [Rs, #off]`   | Register and memory with offset   | `LDW R0, [R1, #4]`  |
| `[Rs], Rd`         | Memory indirect and register      | `STORE [R1], R0`    |
| `#offset`          | Branch/jump target                | `BEQ #-4`           |
| `label`            | Symbolic target (resolved by assembler) | `BEQ loop`    |

## 10.3 Numeric Literals

| Format     | Example    | Value  |
|------------|------------|--------|
| Decimal    | `#42`      | 42     |
| Hexadecimal| `#0xFF`    | 255    |
| Binary     | `#0b1010`  | 10     |
| Octal      | `#0o77`    | 63     |
| Negative   | `#-5`      | -5     |
| Character  | `#'A'`     | 65     |

## 10.4 Directives

| Directive          | Description                              | Example                   |
|--------------------|------------------------------------------|---------------------------|
| `.org addr`        | Set the assembly origin address          | `.org 0x0100`             |
| `.dw value [,...]` | Emit one or more 16-bit words            | `.dw 0x1234, 0x5678`      |
| `.db value [,...]` | Emit one or more bytes                   | `.db 0x41, 0x42, 0x00`    |
| `.ds count`        | Reserve `count` bytes of uninitialized space | `.ds 64`              |
| `.equ name, value` | Define a named constant                  | `.equ BUFFER_SIZE, 256`   |
| `.ascii "string"`  | Emit a string as bytes (no terminator)   | `.ascii "Hello"`          |
| `.asciiz "string"` | Emit a null-terminated string            | `.asciiz "Hello"`         |

## 10.5 Example Program Structure

```asm
; =========================================
; Example: LED blinker
; =========================================

.equ IO_PORT, 0xFF00
.equ DELAY_COUNT, 1000

    .org 0x0000

; --- Interrupt Vector Table ---
    .dw _start          ; RESET
    .dw nmi_handler     ; NMI
    .dw default_irq     ; IRQ0
    .dw default_irq     ; IRQ1
    .dw default_irq     ; IRQ2
    .dw default_irq     ; IRQ3
    .dw default_irq     ; IRQ4
    .dw default_irq     ; IRQ5
    .dw default_irq     ; IRQ6
    .dw default_irq     ; IRQ7

; --- Code ---
_start:
    ; Toggle LED bit
    MOVL  R0, #0x00
    MOVH  R0, #0xFF       ; R0 = IO_PORT
    LOADB R1, [R0]        ; read current state
    XORI  R1, R1, #0x01   ; toggle bit 0
    STOREB [R0], R1       ; write back

    ; Delay loop
    MOVL  R2, #0xE8
    MOVH  R2, #0x03       ; R2 = 1000
delay:
    SUBI  R2, R2, #1
    BNE   delay

    JMP   _start

default_irq:
    RETI

nmi_handler:
    RETI
```

---

*[← Pseudo-Instructions](09-pseudo.md) | [Next: Encoding Reference →](11-encoding-reference.md)*

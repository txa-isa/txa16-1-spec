# 12. Example Programs

## 12.1 Array Sum

Compute the sum of N 16-bit elements.

```asm
; Input:  R0 = pointer to array, R1 = N (element count)
; Output: R2 = sum
; Clobbers: R3

array_sum:
    CLR   R2                ; sum = 0
.loop:
    TST   R1                ; set flags from R1
    BEQ   .done             ; if N == 0, exit
    LOAD  R3, [R0]          ; R3 = MEM[R0]
    ADD   R2, R2, R3        ; sum += R3
    ADDI  R0, R0, #2        ; pointer += 2 (16-bit elements)
    DEC   R1                ; N--
    JMP   .loop
.done:
    RET
```

## 12.2 Conditional Assignment with COND

```asm
; R0 = max(R1, R2)

    MOV   R0, R1            ; R0 = R1 (assume R1 >= R2)
    CMP   R1, R2            ; compare R1 and R2
    COND  LT, 1             ; if R1 < R2:
      MOV R0, R2            ;   R0 = R2
```

## 12.3 Absolute Value

```asm
; R0 = abs(R0)

    SUBI  R7, R0, #0        ; set flags from R0 (Z, N)
    COND  LT, 1             ; if R0 < 0:
      NEG R0, R0            ;   R0 = -R0
```

## 12.4 16-bit Multiply and Accumulate

```asm
; R0 += R1 * R2 (lower 16 bits)

    MUL   R3, R1, R2        ; R3 = (R1 * R2)[15:0]
    ADD   R0, R0, R3        ; R0 += R3
```

## 12.5 Memory Copy (Word-Aligned)

```asm
; Copy N words from [R0] to [R1]
; Input:  R0 = src, R1 = dst, R2 = N (word count)
; Clobbers: R3

memcpy_w:
    TST   R2
    BEQ   .done
.loop:
    LOAD  R3, [R0]          ; R3 = MEM[src]
    STORE [R1], R3          ; MEM[dst] = R3
    ADDI  R0, R0, #2        ; src += 2
    ADDI  R1, R1, #2        ; dst += 2
    DEC   R2                ; N--
    BNE   .loop
.done:
    RET
```

## 12.6 Saturating DSP-Style Accumulation

```asm
; Saturating sum of 4 signed samples
; Input:  R0 = pointer to 4-element signed array
; Output: R1 = clamped sum

    SSAT                    ; enable saturation mode
    CLR   R1                ; sum = 0
    LDW   R2, [R0, #0]     ; sample[0]
    ADD   R1, R1, R2
    LDW   R2, [R0, #1]     ; sample[1] (offset 1 word = 2 bytes)
    ADD   R1, R1, R2
    LDW   R2, [R0, #2]     ; sample[2]
    ADD   R1, R1, R2
    LDW   R2, [R0, #3]     ; sample[3]
    ADD   R1, R1, R2
    CSAT                    ; disable saturation mode
    RET
```

## 12.7 Function Call with Stack Frame

```asm
; int add_and_square(int a, int b)
;   return (a + b) * (a + b)
; R1 = a, R2 = b, R0 = return value

add_and_square:
    PUSH  R4                ; save callee-saved register
    ADD   R4, R1, R2        ; R4 = a + b
    MUL   R0, R4, R4        ; R0 = (a + b)^2 (low 16 bits)
    POP   R4                ; restore R4
    RET
```

## 12.8 Interrupt Handler

```asm
; Minimal IRQ0 handler
; Bank 1 is active, so R0-R7 are IRQ-private

irq0_handler:
    ; Read status from I/O device (memory-mapped at 0xFF00)
    MOVL  R0, #0x00
    MOVH  R0, #0xFF         ; R0 = 0xFF00
    LOADB R1, [R0]          ; R1 = device status byte

    ; Acknowledge interrupt
    MOVL  R2, #0x01
    STOREB [R0], R2         ; write ack byte

    RETI                    ; return, restore SR and bank
```

## 12.9 Full System Initialization

```asm
    .org 0x0000

; --- Interrupt Vector Table ---
    .dw _start              ; RESET vector
    .dw nmi_handler         ; NMI vector
    .dw irq0_handler        ; IRQ0 vector
    .dw default_handler     ; IRQ1
    .dw default_handler     ; IRQ2
    .dw default_handler     ; IRQ3
    .dw default_handler     ; IRQ4
    .dw default_handler     ; IRQ5
    .dw default_handler     ; IRQ6
    .dw default_handler     ; IRQ7

; --- Entry Point ---
_start:
    ; SP is already 0xFFFE from reset

    ; Enable interrupts
    SEI

    ; Jump to main program
    JMP   main

; --- Default Handlers ---
default_handler:
    RETI

nmi_handler:
    RETI

irq0_handler:
    ; ... (see example 12.8)
    RETI

; --- Main Program ---
main:
    ; Application code here
    CLR   R0
.loop:
    INC   R0
    JMP   .loop
```

## 12.10 Loading Arbitrary 16-bit Constants

```asm
; Load 0x0000 - 0x00FF (0-255): one instruction
    MOVL  R0, #0x42         ; R0 = 0x0042

; Load 0xXX00 (high byte only): two instructions
    CLR   R0                ; R0 = 0x0000
    MOVH  R0, #0xAB         ; R0 = 0xAB00

; Load arbitrary 16-bit value: two instructions
    MOVL  R0, #0xCD         ; R0 = 0x00CD
    MOVH  R0, #0xAB         ; R0 = 0xABCD

; Load -1 (0xFFFF): two instructions
    MOVL  R0, #0xFF         ; R0 = 0x00FF
    MOVH  R0, #0xFF         ; R0 = 0xFFFF

; Alternative -1 via NEG:
    MOVL  R0, #1            ; R0 = 0x0001
    NEG   R0, R0            ; R0 = 0xFFFF
```

---

*[← Encoding Reference](11-encoding-reference.md) | [Back to Index](00-index.md)*

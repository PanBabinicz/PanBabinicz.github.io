---
title: ARM Assembly
date: 2024-10-17 12:00:00 +0200
categories: [embedded,software,hardware,arm,assembly]
tags: [embedded,software,hardware,arm,assembly]
---

# ARM-ASSEMBLY

## ❓ What is ARM-ASSEMBLY for?

> This repository contains example programs written in ARM assembly for ARM-based processors,
> along with low-level driver implementations such as I2C, SPI, and UART. It is intended as a
> practical reference for understanding hardware interaction and bare-metal development.

## License

> The MIT License (MIT)
>
> Copyright (c) 2011-2024 The Bootstrap Authors
>
> Permission is hereby granted, free of charge, to any person obtaining a copy
> of this software and associated documentation files (the "Software"), to deal
> in the Software without restriction, including without limitation the rights
> to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
> copies of the Software, and to permit persons to whom the Software is
> furnished to do so, subject to the following conditions:
>
> The above copyright notice and this permission notice shall be included in
> all copies or substantial portions of the Software.
>
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
> IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
> FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
> AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
> LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
> OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
> THE SOFTWARE.

## Blink program

```asm
        .syntax unified
        .cpu cortex-m4
        .thumb

        .include "stm32f446re_rcc.inc"
        .include "stm32f446re_gpio.inc"

        delay_number_of_iterations = 0x00100000

        .text
        .balign 4
        .global _reset_handler
        .type _reset_handler, %function
_reset_handler:
        bl _rcc_setup
        bl _gpio_setup
        ldr r0, =gpioa_base                     @ _blink function argument 0
        ldr r1, =gpiox_bsrr_bs5                 @ _blink function argument 1
        ldr r2, =gpiox_bsrr_br5                 @ _blink function argument 2
        ldr r3, =delay_number_of_iterations     @ _blink function argument 3
        bl _blink
        b .                                     @ Never return.

        .text
        .balign 4
        .type _rcc_setup, %function
_rcc_setup:
        ldr r0, =rcc_base
        ldr r1, [r0, #rcc_ahb1enr_offset]
        orr r1, #rcc_ahb1enr_gpioaen
        str r1, [r0, #rcc_ahb1enr_offset]
        bx lr

        .text
        .balign 4
        .type _gpio_setup, %function
_gpio_setup:
        ldr r0, =gpioa_base
        ldr r1, [r0, #gpiox_moder_offset]
        orr r1, #gpiox_moder_moder5_bit0
        str r1, [r0, #gpiox_moder_offset]
        bx lr

        .text
        .balign 4
        .type _blink, %function
_blink:
        push { r4, lr }
        mov r4, r0
blink_loop:
        str r1, [r4, #gpiox_bsrr_offset]
        mov r0, r3
        bl _blink_delay
        str r2, [r4, #gpiox_bsrr_offset]
        mov r0, r3
        bl _blink_delay
        b blink_loop

        .text
        .balign 4
        .type _blink_delay, %function
_blink_delay:
loop:
        subs r0, #1
        cmp r0, #0
        bne loop
        bx lr

        .end
```

## Hello World program

```asm
        .syntax unified
        .cpu cortex-m4
        .thumb

        .include "stm32f446re_rcc.inc"
        .include "stm32f446re_gpio.inc"
        .include "stm32f446re_usart.inc"

        .global _usart_init
        .global _usart_send_data

        .section .rodata
        .balign 4
        .type hello_world_message, %object
hello_world_message:
        .asciz "Hello, World!"

        .text
        .balign 4
        .global _reset_handler
        .type _reset_handler, %function
_reset_handler:
        bl _rcc_setup
        bl _gpio_setup
        ldr r0, =usart2_base
        bl _usart_init
        bl _main
        b .                     @ Never return.

        .text
        .balign 4
        .type _rcc_setup, %function
_rcc_setup:
        ldr r0, =rcc_base
        ldr r1, [r0, #rcc_ahb1enr_offset]
        orr r1, #rcc_ahb1enr_gpioaen
        str r1, [r0, #rcc_ahb1enr_offset]
        ldr r1, [r0, #rcc_apb1enr_offset]
        orr r1, #rcc_apb1enr_usart2en
        str r1, [r0, #rcc_apb1enr_offset]
        bx lr

        .text
        .balign 4
        .type _gpio_setup, %function
_gpio_setup:
        ldr r0, =gpioa_base
        ldr r1, [r0, #gpiox_moder_offset]
        orr r1, #(gpiox_moder_moder2_bit1 | gpiox_moder_moder3_bit1)
        str r1, [r0, #gpiox_moder_offset]
        ldr r1, [r0, #gpiox_afrl_offset]
        orr r1, #(gpiox_afrl_afrl2_bit0 | gpiox_afrl_afrl2_bit1 | gpiox_afrl_afrl2_bit2)
        orr r1, #(gpiox_afrl_afrl3_bit0 | gpiox_afrl_afrl3_bit1 | gpiox_afrl_afrl3_bit2)
        str r1, [r0, #gpiox_afrl_offset]
        bx lr

        .text
        .balign 4
        .type _main, %function
_main:
        push { lr }             @ Push only 4 bytes.
        sub sp, sp, #4          @ Adjust to maintain 8-byte alignment.
        ldr r0, =usart2_base
        ldr r1, =hello_world_message
        bl _usart_send_data
1:
        wfi
        b 1b
        add sp, sp, #4          @ Restore stack pointer before returning.
        pop { lr }              @ Should never go here.
        bx lr

        .end
```

## Usage

- **Debugger**  
  `arm-none-eabi-gdb <binary>`

- **OpenOCD**  
  `openocd -f <interface-config> -f <board-config>`

- **Assembler**  
  `arm-none-eabi-as -g -I <include-path> <source-file> -o <object-file>`

- **Linker**  
  `arm-none-eabi-ld <object-files> -o <binary-file> -T <linker-script>`
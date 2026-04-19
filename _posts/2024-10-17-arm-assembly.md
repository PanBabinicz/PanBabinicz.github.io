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

## USART driver

```asm
        .syntax unified
        .cpu cortex-m4
        .thumb

        .include "stm32f446re_usart.inc"

        .text
        .balign 4
        .type _usart_enable, %function
        @ Argument in r0: usart base address
_usart_enable:
        ldr r1, [r0, #usart_cr1_offset]
        orr r1, #usart_cr1_ue
        str r1, [r0, #usart_cr1_offset]
        bx lr

        .text
        .balign 4
        .type _usart_set_m_bit, %function
        @ Argument in r0: usart base address
        @ Argument in r1: m bit value
_usart_set_m_bit:
        ldr r2, [r0, #usart_cr1_offset]
        orr r2, r1
        str r2, [r0, #usart_cr1_offset]
        bx lr

        .text
        .balign 4
        .type _usart_set_stop_bits, %function
        @ Argument in r0: usart base address
        @ Argument in r1: set bits value
_usart_set_stop_bits:
        ldr r2, [r0, #usart_cr2_offset]
        orr r2, r1
        str r2, [r0, #usart_cr2_offset]
        bx lr

        .text
        .balign 4
        .type _usart_set_baud_rate, %function
        @ Argument in r0: usart base address
        @ Argument in r1: fraction value
        @ Argument in r2: mantissa value
        @ The reset value of sysclk is 96MHz
        @ It is calculated like this, values are in RCC_PLLCFGR register:
        @ 1. Check which oscilator is used (on reset it is HSI which is 16MHz)
        @ 2. Divide PLL input clock by PLLM which is 16
        @ 3. The divided input clock is 1MHz, multiply it by PLLN (192)
        @ 4. The VCO is now 192MHz, lets divide it by PLLP (which is 2)
        @ 5. The main system clock is now set to 96MHz
        @ On the clock tree you can see that APB1 gets sysclk which is
        @ main system clock
        @ The last thing is to check if there are any prescalers in
        @ RCC_CFGR register.
        @ For 9600 baudrate and 96MHz clock the mantissa has to be
        @ set to 625, it is calculated in the following way:
        @ 1. USARTDIV = fpclk1/(16*9600) = 96MHz/(16*9600) = 625
        @ NOTE: the PLL can be not turned on, so SYSCLK will be 16MHz
        @ 1. USARTDIV = 16MHz/(16*9600) = 104.1667
        @ 2. mantissa = 104 (0x68)
        @ 3. fraction = 0.1667 * 16 = 2.667 = 3 (0x03)
_usart_set_baud_rate:
        lsl r2, #usart_brr_divmantissa_position
        orr r2, r1
        str r2, [r0, #usart_brr_offset]
        bx lr

        .text
        .balign 4
        .type _usart_set_idle_frame, %function
_usart_set_idle_frame:
        ldr r1, [r0, #usart_cr1_offset]
        orr r1, #usart_cr1_te
        str r1, [r0, #usart_cr1_offset]
        bx lr

        .text
        .balign 4
        .global _usart_send_data
        .type _usart_send_data, %function
        @ Argument in r0: usart base address
        @ Argument in r1: address of the message
_usart_send_data:
1:
        ldrb r2, [r1], #1
        cmp r2, #0x00
        beq usart_send_end
        strb r2, [r0, #usart_dr_offset]
2:
        ldr r3, [r0, #usart_sr_offset]
        tst r3, #usart_sr_txe
        beq 2b
        b 1b
usart_send_end:
        ldr r3, [r0, #usart_sr_offset]
        tst r3, #usart_sr_tc
        beq usart_send_end
        bx lr

        .text
        .balign 4
        .global _usart_init
        .type _usart_init, %function
        @ Argument in r0: usart base address
_usart_init:
        push { lr }             @ Push only 4 bytes.
        sub sp, sp, #4          @ Adjust to maintain 8-byte alignment.
        bl _usart_enable        @ Argument in r0 is usart base address.
        mov r1, #0x00
        bl _usart_set_m_bit
        mov r1, #0x00
        bl _usart_set_stop_bits
        mov r1, #0x03
        mov r2, #0x68
        bl _usart_set_baud_rate
        bl _usart_set_idle_frame
        add sp, sp, #4          @ Restore stack pointer before returning.
        pop { lr }
        bx lr

        .end

```

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

## BME280 I2C example

```asm
        .syntax unified
        .cpu cortex-m4
        .thumb

        .include "stm32f446re_rcc.inc"
        .include "stm32f446re_gpio.inc"
        .include "stm32f446re_i2c.inc"

        .global _i2c_init
        .global _i2c_master_transmit

        i2c1_base_address       = i2c1_base
        i2c1_clock_frequency    = 0x00f42400    @ 16MHz
        i2c1_scl_frequency      = 0x000493e0    @ 300kHz

        i2c_slave_address       = 0x76          @ Slave address
        i2c_frame_reset_size    = 0x02          @ Size of the reset frame
        i2c_frame_id_size       = 0x01          @ Size of the id frame

        .section .rodata
        .balign 4
        .type i2c_frame_reset_start, %object
        @ This frame resets the bme280 sensor
i2c_frame_reset_start:
        .byte 0xe0
        .byte 0xb6

        .section .rodata
        .balign 4
        .type i2c_frame_id_start, %object
        @ This frame gets the bme280 sensor id
i2c_frame_id_start:
        .byte 0xd0

        .data
        .balign 4
        .type i2c_receive_data_buffer_start, %object
        @ This frame gets the bme280 sensor id
i2c_receive_data_buffer_start:
        .byte 0x00

        .text
        .balign 4
        .global _reset_handler
        .type _reset_handler, %function
_reset_handler:
        bl _rcc_setup
        bl _gpio_setup
        bl _main
        b .                                     @ Never return

        .text
        .balign 4
        .type _rcc_setup, %function
_rcc_setup:
        ldr r0, =rcc_base
        ldr r1, [r0, #rcc_apb1enr_offset]
        orr r1, #rcc_apb1enr_i2c1en
        str r1, [r0, #rcc_apb1enr_offset]       @ Set the clock on i2c1
        ldr r1, [r0, #rcc_ahb1enr_offset]
        orr r1, #rcc_ahb1enr_gpioben
        str r1, [r0, #rcc_ahb1enr_offset]       @ Set the clock on gpiob
        bx lr

        .text
        .balign 4
        .type _gpio_setup, %function
_gpio_setup:
        ldr r0, =gpiob_base
        ldr r1, [r0, #gpiox_moder_offset]
        orr r1, #(gpiox_moder_moder8_bit1 | gpiox_moder_moder9_bit1)
        str r1, [r0, #gpiox_moder_offset]
        ldr r1, [r0, #gpiox_pupdr_offset]
        orr r1, #(gpiox_pupdr_pupdr8_bit0 | gpiox_pupdr_pupdr9_bit0)
        str r1, [r0, #gpiox_pupdr_offset]
        ldr r1, [r0, #gpiox_afrh_offset]
        orr r1, #(gpiox_afrh_afrh8_bit2 | gpiox_afrh_afrh9_bit2)
        str r1, [r0, #gpiox_afrh_offset]
        bx lr

        .text
        .balign 4
        .type _main, %function
_main:
        push { lr }
        sub sp, sp, #0x04
        ldr r0, =i2c1_base_address
        ldr r1, =i2c1_clock_frequency
        ldr r2, =i2c1_scl_frequency
        bl _i2c_init
        ldr r1, =i2c_frame_reset_start
        ldr r2, =i2c_frame_reset_size
        ldr r3, =i2c_slave_address
        bl _i2c_master_transmit
        ldr r1, =i2c_frame_id_start
        mov r2, #i2c_frame_id_size
        ldr r3, =i2c_slave_address
        bl _i2c_master_transmit                 @ Write the address of the register
        ldr r1, =i2c_receive_data_buffer_start
        mov r2, #0x01
        ldr r3, =i2c_slave_address
        bl _i2c_master_receive                  @ Read from this register
1:
        wfi
        b 1b
        add sp, sp, #0x04                       @ Should never go here
        pop { lr }
        bx lr

        .end

```

## I2C Driver

```asm
        .syntax unified
        .cpu cortex-m4
        .thumb

        .include "stm32f446re_i2c.inc"

        .text
        .balign 4
        .type _i2c_software_reset, %function
_i2c_software_reset:
1:
        ldr r1, [r0, #i2c_cr1_offset]
        orr r1, #i2c_cr1_swrst
        str r1, [r0, #i2c_cr1_offset]
        ldr r1, [r0, #i2c_cr1_offset]
        tst r1, #i2c_cr1_swrst
        it eq
        beq 1b
        bic r1, #i2c_cr1_swrst
        str r1, [r0, #i2c_cr1_offset]
        bx lr

        .text
        .balign 4
        .type _i2c_set_peripheral_input_clock, %function
        @ Argument in r0: i2c base address
        @ Argument in r1: i2c clock frequency (2MHz, 50MHz)
_i2c_set_peripheral_input_clock:
        ldr r3, =0x000f4240             @ 1000000
        udiv r1, r1, r3
        str r1, [r0, #i2c_cr2_offset]
        bx lr

        .text
        .balign 4
        .type _i2c_configure_clock, %function
        @ Argument in r0: i2c base address
        @ Argument in r1: i2c clock frequency (2MHz, 50MHz)
        @ Argument in r2: i2c scl frequency (0, 100kHz) or (100kHz, 400kHz)
_i2c_configure_clock:                   @ TODO: Check the math here
        push { r4-r7 }
        lsl r4, r2, #0x01               @ 2*fscl
        ldr r3, =0x000186a0             @ 100kHz
        cmp r2, r3
        itte hi
        addhi r4, r2                    @ 3*fscl
        movhi r5, #0x01                 @ Fast mode
        movls r5, #0x00                 @ Standard mode
        udiv r6, r1, r4                 @ fpclk / x*fscl
        lsl r5, #0x0f                   @ shift f/s bit on 15th position
        orr r6, r5
        bic r6, #(0x01 << 14)           @ duty is 0
        str r6, [r0, #i2c_ccr_offset]
i2c_configure_clock_exit:
        pop { r4-r7 }
        bx lr

        .text
        .balign 4
        .type _i2c_configure_rise_time, %function
        @ Argument in r0: i2c base address
        @ Argument in r1: i2c clock frequency (2MHz, 50MHz)
        @ Argument in r2: i2c scl frequency (0, 100kHz) or (100kHz, 400kHz)
_i2c_configure_rise_time:               @ TODO: Bug in this function
        push { r4 }
        sub sp, sp, #0x04
        ldr r3, =0x000186a0             @ 100kHz
        cmp r2, r3
        ittt hi
        movhi r3, #0x012c               @ 300ns
        mulhi r4, r1, r3                @ trise = clock_frequency * r3
        lsrhi r4, r4, #0x0a             @ divide by 1024
        itt ls
        movls r3, #0x01                 @ 1us
        mulls r4, r1, r3                @ trise = clock_frequency * r3
        add r4, r4, #0x01
        str r4, [r0, #i2c_trise_offset]
        add sp, sp, #0x04
        pop { r4 }
        bx lr

        .text
        .balign 4
        .type _i2c_enable, %function
        @ Argument in r0: i2c base address
_i2c_enable:
        ldr r1, [r0, #i2c_cr1_offset]
        orr r1, #i2c_cr1_pe
        str r1, [r0, #i2c_cr1_offset]
        bx lr

        .text
        .balign 4
        .type _i2c_check_arguments, %function
        @ Argument in r0: i2c check argument status
        @ Argument in r1: i2c clock frequency (2MHz, 50MHz)
        @ Argument in r2: i2c scl frequency (0, 100kHz) or (100kHz, 400kHz)
_i2c_check_arguments:
        ldr r3, =0x001e8480             @ 2MHz
        cmp r1, r3
        itt lo
        movlo r0, #0x01                 @ Error: clock frequency is less than 2MHz
        blo i2c_check_arguments_exit
        ldr r3, =0x02faf080             @ 50MHz
        cmp r1, r3
        itt hi
        movhi r0, #0x01                 @ Error: clock frequency is greater than 50MHz
        bhi i2c_check_arguments_exit
        ldr r3, =0x00                   @ 0Hz
        cmp r2, r3
        itt eq
        moveq r0, #0x01                 @ Error: scl frequency is equal 0Hz
        beq i2c_check_arguments_exit
        ldr r3, =0x00061a80             @ 400kHz
        cmp r2, r3
        itt hi
        movhi r0, #0x01                 @ Error: scl frequency is greater than 400kHz
        bhi i2c_check_arguments_exit
        mov r0, #0x00                   @ Success: clock and scl frequencies are okey
i2c_check_arguments_exit:
        bx lr

        .text
        .balign 4
        .type _i2c_transmit_byte, %function
        @ Argument in r0: i2c base address
        @ Argument in r1: data byte
_i2c_transmit_byte:
        ldrb r2, [r1], #0x01
        strb r2, [r0, #i2c_dr_offset]
        bx lr

        .text
        .balign 4
        .type _i2c_receive_byte, %function
        @ Argument in r0: i2c base address
        @ Argument in r1: data buffer address
_i2c_receive_byte:
        ldrb r2, [r0, #i2c_dr_offset]
        strb r2, [r1], #0x01
        bx lr

        .text
        .balign 4
        .global _i2c_master_transmit
        .type _i2c_master_transmit, %function
        @ Argument in r0: i2c base address
        @ Argument in r1: data buffer address
        @ Argument in r2: data size size
        @ Argument in r3: slave address
_i2c_master_transmit:
        push { r4-r5, lr }
        mov r5, r2
        sub sp, sp, #0x04
        ldr r4, [r0, #i2c_cr1_offset]
        orr r4, #i2c_cr1_start
        str r4, [r0, #i2c_cr1_offset]   @ Start generation
i2c_master_transmit_check_sb:
        ldr r4, [r0, #i2c_sr1_offset]
        tst r4, #i2c_sr1_sb
        it eq
        beq i2c_master_transmit_check_sb
        lsl r3, #0x01                   @ Shift the address, lsb signals the mode
        bic r3, #0x01                   @ Enter the transmitter mode
        strb r3, [r0, #i2c_dr_offset]
i2c_master_transmit_check_addr:
        ldr r4, [r0, #i2c_sr1_offset]
        tst r4, #i2c_sr1_addr
        it eq
        beq i2c_master_transmit_check_addr
        ldr r4, [r0, #i2c_sr2_offset]   @ Read sr2 after sr1 to clear the addr
i2c_master_transmit_next_byte:
        bl _i2c_transmit_byte           @ Don't need to save r2 because I dont use it anymore
        subs r5, #0x01
        it eq
        beq i2c_master_transmit_check_btf
i2c_master_transmit_is_empty:
        ldr r4, [r0, #i2c_sr1_offset]
        tst r4, #i2c_sr1_txe
        it ne
        bne i2c_master_transmit_next_byte
        it eq
        beq i2c_master_transmit_is_empty
i2c_master_transmit_check_btf:
        ldr r4, [r0, #i2c_sr1_offset]
        tst r4, #i2c_sr1_btf
        it eq
        beq i2c_master_transmit_check_btf
        ldr r4, [r0, #i2c_cr1_offset]
        orr r4, #i2c_cr1_stop
        str r4, [r0, #i2c_cr1_offset]
        add sp, sp, #0x04
        pop { r4-r5, lr }
        bx lr

        .text
        .balign 4
        .global _i2c_master_receive
        .type _i2c_master_receive, %function
        @ Argument in r0: i2c base address
        @ Argument in r1: data buffer address
        @ Argument in r2: data buffer size
        @ Argument in r3: slave address
_i2c_master_receive:
        push { r4-r5, lr }
        sub sp, sp, #0x04
        mov r5, r2                      @ Save the data buffer size
        ldr r4, [r0, #i2c_cr1_offset]
        orr r4, #i2c_cr1_start
        str r4, [r0, #i2c_cr1_offset]   @ Start generation
i2c_master_receive_check_sb:
        ldr r4, [r0, #i2c_sr1_offset]
        tst r4, #i2c_sr1_sb
        it eq
        beq i2c_master_receive_check_sb
        lsl r3, #0x01                   @ Shift the address, lsb signals the mode
        orr r3, #0x01                   @ Enter the receiver mode
        strb r3, [r0, #i2c_dr_offset]
i2c_master_receive_check_addr:
        ldr r4, [r0, #i2c_sr1_offset]
        tst r4, #i2c_sr1_addr
        it eq
        beq i2c_master_receive_check_addr
        ldr r4, [r0, #i2c_sr2_offset]   @ Read sr2 after sr1 to clear the addr
i2c_master_receive_next_byte:
1:
        ldr r4, [r0, #i2c_sr1_offset]
        tst r4, #i2c_sr1_rxne
        it eq
        beq 1b                          @ Wait for the data to be available in buffer
        subs r5, #0x01
        bl _i2c_receive_byte
        ite ne
        cmpne r5, #0x01
        beq 3f
        it ne
        bne i2c_master_receive_next_byte
        ldr r4, [r0, #i2c_cr1_offset]   @ Prepare the NACK
        bic r4, #i2c_cr1_ack
        str r4, [r0, #i2c_cr1_offset]
2:
        ldr r4, [r0, #i2c_sr1_offset]
        tst r4, #i2c_sr1_rxne
        it eq
        beq 2b                          @ Wait for the data to be available in buffer
        bl _i2c_receive_byte            @ Receive the last byte
3:
        ldr r4, [r0, #i2c_cr1_offset]
        orr r4, #i2c_cr1_stop           @ Set STOP
        str r4, [r0, #i2c_cr1_offset]
        add sp, sp, #0x04
        pop { r4-r5, lr }
        bx lr

        .text
        .balign 4
        .global _i2c_slave_transmit
        .type _i2c_slave_transmit, %function
        @ Argument in r0: i2c base address
        @ Argument in r1: data buffer
        @ Argument in r2: data size
_i2c_slave_transmit:
        push { r4, lr }
        mov r4, r2
1:
        ldr r3, [r0, #i2c_sr1_offset]
        tst r3, #i2c_sr1_addr           @ Check if addr flag is set (the received address matched)
        it eq
        beq 1b
        ldr r3, [r0, #i2c_sr2_offset]   @ Clear the addr flag
i2c_slave_transmit_next_byte:
2:
        ldr r3, [r0, #i2c_sr1_offset]
        tst r3, #i2c_sr1_txe
        it eq
        beq 2b
        subs r4, #0x01
        bl _i2c_transmit_byte
        it ne
        bne i2c_slave_transmit_next_byte
i2c_slave_transmit_check_stopf_af:
        ldr r3, [r0 ,#i2c_sr1_offset]
        tst r3, #i2c_sr1_stopf          @ Check if stop flag is set
        it ne
        bne i2c_slave_transmit_stopf_detected
        tst r3, #i2c_sr1_af
        it eq
        beq i2c_slave_transmit_check_stopf_af
i2c_slave_transmit_stopf_detected:
        ldr r3, [r0, #i2c_cr1_offset]
        str r3, [r0, #i2c_cr1_offset]   @ Clear the STOPF flag
        mov r3, #0x00
        str r3, [r0, #i2c_sr1_offset]   @ Clear the AF flag
        pop { r4, lr }
        bx lr

        .text
        .balign 4
        .global _i2c_slave_receive
        .type _i2c_slave_receive, %function
        @ Argument in r0: i2c base address
        @ Argument in r1: data buffer address
        @ Argument in r2: data buffer size
_i2c_slave_receive:
        push { r4, lr }
        mov r4, r2
1:
        ldr r3, [r0, #i2c_sr1_offset]
        tst r3, #i2c_sr1_addr           @ Check if addr flag is set (the received address matched)
        it eq
        beq 1b
        ldr r3, [r0, #i2c_sr2_offset]   @ Clear the addr flag
i2c_slave_receive_next_byte:
2:
        ldr r3, [r0, #i2c_sr1_offset]
        tst r3, #i2c_sr1_rxne
        it eq
        beq 2b
        subs r4, #0x01
        bl _i2c_receive_byte
        it ne
        bne i2c_slave_receive_next_byte
i2c_slave_receive_check_stopf:
        ldr r3, [r0, #i2c_sr1_offset]
        tst r3, #i2c_sr1_stopf
        it eq
        beq i2c_slave_receive_check_stopf
        ldr r3, [r0, #i2c_cr1_offset]
        str r3, [r0, #i2c_cr1_offset]
        pop { r4, lr }
        bx lr

        .text
        .balign 4
        .global _i2c_init
        .type _i2c_init, %function
        @ Argument in r0: i2c base address / i2c init status
        @ Argument in r1: i2c clock frequency (2MHz, 50MHz)
        @ Argument in r2: i2c scl frequency (0, 100kHz) or (100kHz, 400kHz)
_i2c_init:
        push { r4-r5, lr }
        sub sp, sp, #0x04
        mov r4, r0
        mov r5, r1
        bl _i2c_check_arguments
        cmp r0, #0x00
        it ne
        bne i2c_init_exit
        mov r0, r4
        bl _i2c_software_reset
        mov r0, r4
        mov r1, r5
        bl _i2c_set_peripheral_input_clock
        mov r1, r5
        bl _i2c_configure_clock
        bl _i2c_configure_rise_time
        bl _i2c_enable
i2c_init_exit:
        add sp, sp, #0x04
        pop { r4-r5, lr }
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

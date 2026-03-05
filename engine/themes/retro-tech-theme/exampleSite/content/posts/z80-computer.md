---
title: "Building a Z80 Computer from Scratch"
date: 2026-01-15
author: "jurek"
tags: ["z80", "retro-computing", "hardware", "assembly"]
image: "/images/z80-board.jpg"
---

Ever wondered what it's like to build a computer from the ground up? Join me as I document my journey creating a Z80-based computer with nothing but datasheets and determination.

## The Beginning

The Zilog Z80 processor, released in 1976, powered countless computers including the ZX Spectrum, MSX, and even the original Game Boy. Its elegant instruction set and affordable price made it perfect for hobbyists.

### Parts List

- Zilog Z80 CPU (CMOS version)
- 32KB SRAM (62256)
- 32KB EEPROM (AT28C256)
- Address decoder logic
- Clock circuit (1MHz oscillator)

## The Memory Map

```
0x0000 - 0x7FFF : ROM (32KB)
0x8000 - 0xFFFF : RAM (32KB)
```

This gives us a clean split between program storage and working memory.

## First Boot

When I first powered it on, nothing happened. Classic! After triple-checking my connections with a multimeter, I found a loose wire on the address bus. One solder joint later, and the address lines started dancing on my logic analyzer.

```asm
; First program - blink an LED
ORG 0000h
START:
    LD A, 01h      ; Load 1 into accumulator
    OUT (80h), A   ; Output to port 80h
    CALL DELAY
    XOR A          ; Clear accumulator
    OUT (80h), A
    CALL DELAY
    JP START
```

## Next Steps

- Add a UART for serial communication
- Implement a simple monitor program
- Port Tiny BASIC (ambitious, I know!)

The best part about retro computing? When something works, you *know* exactly why it works. Every bit, every clock cycle is yours to command.

More updates coming soon! 💾

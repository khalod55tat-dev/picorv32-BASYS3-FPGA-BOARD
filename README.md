
# PicoRV32 RISC-V SoC on Basys3 FPGA

A RISC-V based System-on-Chip implemented on the Xilinx Artix-7 (Basys3) FPGA board, using the PicoRV32 32-bit RISC-V processor core.

## Overview

This project integrates the PicoRV32 RV32I core into a minimal SoC with memory-mapped peripherals including GPIO and UART, synthesised and verified on Xilinx Basys3 hardware using Vivado.

## Features

- PicoRV32 32-bit RISC-V (RV32I) processor core
- Memory-mapped GPIO — LED outputs and slide switch inputs
- UART interface at 115200 baud for serial communication
- On-chip SRAM for program and data storage
- Synthesised, placed, routed and verified using Vivado
- Bare-metal C firmware compiled with riscv32-unknown-elf-gcc

## Hardware

| Item | Details |
|------|---------|
| FPGA Board | Digilent Basys3 |
| FPGA Device | Xilinx Artix-7 XC7A35T |
| Clock | 100 MHz onboard oscillator |
| Target Frequency | 50 MHz |

## Memory Map

| Address | Peripheral |
|---------|-----------|
| 0x00000000 | Program ROM / SRAM |
| 0x02000000 | GPIO — LEDs (write) |
| 0x02000004 | GPIO — Switches (read) |
| 0x03000000 | UART data register |
| 0x03000004 | UART clock divider |

## Project Structure

```
picorv32-basys3-soc/
├── rtl/
│   ├── top.v              — Top-level SoC module
│   ├── picorv32.v         — PicoRV32 core
│   ├── memory.v           — On-chip SRAM
│   ├── gpio.v             — Memory-mapped GPIO
│   └── uart.v             — UART peripheral
├── firmware/
│   ├── main.c             — Bare-metal C firmware
│   └── sections.lds       — Linker script
├── constraints/
│   └── basys3.xdc         — Pin assignments for Basys3
├── progmem.v              — Generated program memory
└── README.md
```

## Tools Required

- Vivado 2020.x or later (Xilinx/AMD)
- RISC-V toolchain — riscv32-unknown-elf-gcc
- Python 3 — for generating program memory hex file

## How to Build

**1 — Compile firmware**
```bash
riscv32-unknown-elf-gcc -march=rv32i -mabi=ilp32 -Os \
  -Tlinker/sections.lds firmware/main.c -o firmware.elf
riscv32-unknown-elf-objcopy -O binary firmware.elf firmware.bin
python3 make_hex_mem.py firmware.bin > progmem.v
```

**2 — Synthesise in Vivado**
Open Vivado, create a new project targeting XC7A35T (Basys3), add all source files from `rtl/`, add `constraints/basys3.xdc`, then run Synthesis, Implementation, and Generate Bitstream.

**3 — Program the board**
Connect Basys3 via USB, open Hardware Manager in Vivado, and program the generated bitstream.

**4 — Verify via UART**
Open a serial terminal (PuTTY or Tera Term) at 115200 baud on the Basys3 COM port. You should see firmware output printed over UART.

## What It Does

Once programmed, the firmware runs on the PicoRV32 core and demonstrates:
- LED patterns controlled via memory-mapped writes to the GPIO register
- Switch state read back via the GPIO input register and reflected on LEDs
- UART output printing system status to a connected serial terminal

## Based On

Adapted from [celuk/picosoc-port-basys3](https://github.com/celuk/picosoc-port-basys3) and the original [YosysHQ/picorv32](https://github.com/YosysHQ/picorv32) by Clifford Wolf.

## License

This project is released under the ISC License, consistent with the original PicoRV32 core.

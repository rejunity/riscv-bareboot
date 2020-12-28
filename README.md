# RISC-V Bare Metal Tiny OS
Coding RISC-V bare metal boot loader and exokernel OS for education purposes.

## Goal
Learn how to build minimal OS for RISC-V architecture:
- Source code should be small and self-explanatory. Aim at 2-3 files.
- No external dependencies except the RISC-V GNU toolchain.
- Load and execture ELF binary in User Space. Trap invalid operations without crashing the whole machine.
- Small RAM memory footprint. Aim below 16KB.
- Provide basic syscalls.
- Run on real hardware such as SiFive E board.
- Run small user space "hello world" program. Playable "Tetris" as a stretch goal!

## Understanding RISC-V
To start building the bare metal OS, we need an absolute minimal bootable sample code for RISC-V system. The very first stop would be pure RISC-V assembly example that does exactly that: https://github.com/noteed/riscv-hello-asm. Boots and prints "Hello".

RISC-V Instruction Set Architecture (ISA) is very elegant and easy to learn. One could turbocharge into RISC-V assembly with a single double-sided page https://github.com/jameslzhu/riscv-card/blob/master/riscv-card.pdf and RISC-V Assembly Programmer's Manual https://github.com/riscv/riscv-asm-manual/blob/master/riscv-asm.md

Reset vector, ROM, RAM addresses.

M/S/U modes

Flattened Device Tree (FDT)

Physical Memory Protection (PMP)

Syscalls


## Steps
- Second Stage Bootloader. CPU is in M-mode (Machine mode).
- Handle hardware threads (HARTs) and setup stack for each.
- Handle miniam device initialization: UART. Use Flattened Device Tree (FDT) when available.
- Transition from M-mode (machine) to S-mode (supervisor).
- Setup memory protection.
- Setup interrupt handlers.
- Basic syscalls.
- Load ELF.

## Build & Run
### Toolchain
### QEMU

## Inspiration


## Useful links
- Hardware Abstraction Layer for several physical RISC-V boards out there  https://github.com/sifive/freedom-e-sdk

## Inspiration
- OSDev -- hugely useful forum and wiki, but mostly for x86
   - PC Boot sequence https://wiki.osdev.org/Boot_Sequence
   - Simple x86 kernel https://wiki.osdev.org/Bare_bones
   - Tutorial in small steps https://wiki.osdev.org/Babystep1
- General OS tutorials & examples:
   - Roll your own toy UNIX-clone OS  http://www.jamesmolloy.co.uk/tutorial_html/index.html
- Small & Embedded OSes
   * 64 bit BareMetal OS https://github.com/ReturnInfinity/BareMetal-OS with x86_64 bootloader https://gitlab.com/ReturnInfinity/Pure64
   * 
- RISC-V Bootloaders
   * Berkeley Boot Loader
   * u-boot — Standard bootloader for embedded Linux systems
   * Coreboot
   * RISC-V FPGA SmartFusion2 M2S150 Devkit bootloader https://github.com/RISCV-on-Microsemi-FPGA/RVBM-BootLoader
- Interviews:
   * BareMetal OS https://www.osnews.com/story/24815/interview-with-baremetal-os-ian-seyler/

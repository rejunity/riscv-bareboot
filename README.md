# RISC-V Bare Metal Tiny OS
Coding RISC-V bare metal boot loader and exokernel OS for education purposes.

## The Goal
The goal of this project is to learn how to build minimal OS for RISC-V architecture.
- Source code should be small and self-explanatory. Aim at 2-3 files.
- No external dependencies except the RISC-V GNU toolchain.
- Load and execture ELF binary in User Space. Trap invalid operations without crashing the whole machine.
- Small RAM memory footprint. Aim below 16KB.
- Provide basic syscalls.
- Run on real hardware such as SiFive E development board.
- Run small user space "hello world" program. Playable "Tetris" as a stretch goal!

## Build & Run
### Toolchain
### QEMU

## Understanding RISC-V
To start building the bare metal OS, we need an absolute minimal bootable sample code for RISC-V system. The very first stop would be pure RISC-V assembly example that does exactly that: https://github.com/noteed/riscv-hello-asm. Boots and prints "Hello".

RISC-V Instruction Set Architecture (ISA) is very elegant and easy to learn. One could turbocharge into RISC-V assembly with a [cheat-sheet card](https://github.com/jameslzhu/riscv-card/blob/master/riscv-card.pdf) and [RISC-V Assembly Programmer's Manual](https://github.com/riscv/riscv-asm-manual/blob/master/riscv-asm.md)

As of writing there are several physical development boards and virtual environments that sport RISC-V cores. QEMU emulates two physical profiles [`sifive_e`](https://github.com/qemu/qemu/blob/master/hw/riscv/sifive_e.c) and [`sifive_u`](https://github.com/qemu/qemu/blob/master/hw/riscv/sifive_u.c) resembling *HiFive1* and *HiFive Unleashed* development boards respectively.

[HiFive1 revB](https://www.sifive.com/boards/hifive1-rev-b) board is based on FE310-G002 System-on-Chip:
- Manual: https://sifive.cdn.prismic.io/sifive%2F59a1f74e-d918-41c5-b837-3fe01ba7eaa1_fe310-g002-manual-v19p05.pdf
- 32-bit E31 RISC‐V core
- 16KiB RAM (DTIM) + 8KiB instruction cache that can be configured into code scratchpad (ITIM)
- Privelege modes: Machine, User
- PMP: 8 regions with a minimum region size of 4 bytes.

[HiFive Unleashed](https://www.sifive.com/boards/hifive-unleashed) board is based on FU540-C000 System-on-Chip:
- Manual: https://static.dev.sifive.com/FU540-C000-v1.0.pdf
- 4 64-bit U54 RISC‐V cores
- 8 GB DDR4 RAM
- Sv39 virtual memory, 38-bit physical address space, and a 32-entry TLB.
- Privelege modes: Machine, Supervisor, User
- PMP: 8 regions with a minimum region size of 4 bytes.
- 1 E51 RISC‐V core
- 8 KiB DTIM + 8KiB instruction cache that can be configured into code scratchpad (ITIM)
- Privelege modes: Machine, User
- PMP: 8 regions with a minimum region size of 4 bytes.

Control and Status Registers (CSR)
   Starts with letter describing level of privilege.
	12-bit encoding space (csr[11:0]) for up to 4,096 CSRs.
   CSRR/CSRW (read/write) pseudoinstructions that are based on CSRRW, CSRRS, CSRRC instructions
   SYSTEM major opcode = encodes all privileged instructions
   CSR include info:
   - Machine information (vendor/arch/impl/isa/threadid)
   - Exception/interrupt delegates & trap handler (base address), interrupt enable. Delegation will redirect trap to handler in less-protected mode.
   - Trap handling
   - (Only M) Physical memory protection : pmpcfg0..3, pmpaddr0..15 
   - (Only S) Supervisor adder translation and protection: satp
   - ...

## Booting RISC-V and Priveleged Execution

### M/S/U modes
   - (U)ser/Application
   - (S)upervisor
   - (M)achine
   - Switching to S and U with FreedomSDK - https://github.com/sifive/example-privilege-level


### Reset vector, ROM, RAM addresses.
   - All harts are set to M mode. mstatus.mie = 0, mstatus.mprv = 0
   - mcause = cause of reset (implementation specific)
   - pc = reset vector (implementation specific)


### Memory / Physical Memory Protection (PMP)
	- Physical Memory Attributes (PMA) below stay (mostly) fixed during the execution.
	- The physical memory map for a complete system includes various address ranges, some correspond- ing to memory regions, some to memory-mapped control registers, and some to empty holes in the address space.
	- Some memory regions might not support reads, writes, or execution; some might not support subword or subblock accesses; some might not support atomic operations; and some might not support cache coherence or might have different memory models. 
	- PMAs are checked for any access to physical memory, including accesses that have undergone virtual to physical memory translation. 
	- Precisely trapped PMA violations manifest as load, store, or instruction-fetch access exceptions, distinct from virtual- memory page-fault exceptions. 
	- An optional Physical Memory Protection (PMP) unit provides per-hart machine-mode control registers to allow physical memory access 	privileges (read, write, execute) to be specified for each physical memory region. The PMP values are checked in parallel with the PMA checks.
	- PMP checks are applied to all accesses when the hart is running in S or U modes, and for loads and stores when the MPRV bit is set in the mstatus register and the MPP field in the mstatus register contains S or U. 
	- PMP checks are also applied to page-table accesses for virtual-address translation, for which the effective privilege mode is S.
	- PMP can grant permissions to S and U modes, which by default have none, and can revoke permissions from M-mode, which by default has full permissions. 
	- Failed accesses generate a load, store, or instruction access exception. 
	- 16 entrees: pmpcfg0..3 (densely packed mapping for 16 entrees: RV32 - 4 in each reg, RV64 - 8 in each even reg), 8 bits each LxxAAXWR + pmpaddr0..15. AA=0 (off),  AA=1 (TOR, address is “end of the segment”) AA=2 (4 byte segments), AA=3(power-2 size segments). L==locked.
	- If L bit is clear, the R/W/X permissions apply only to U-mode, if L is set - to both M & U.

### Memory in Supervisor mode
   - Supervisor Address Translation and Protection (satp) Register 
      - PPN=physical page number of the root page table
      - MODE=Bare | Sv32 (RV32 only) | Sv39 (RV64 only) | Sv48 (RV64 only) | where SvBB == Page-based BB-bit virtual addressing.
      - Sv32=4GB, Sv48 = 256TB.
      - page = 4KB
   - SFENCE.VMA instruction 

### Interrupts
   - Supports Machine Mode interrupts - local and global. 
      - Local interrupts are signaled directly to an individual hart with a dedicated interrupt value. Local are generated by CLINT. 
      - Global interrupts - PLIC.

   - Timer and software interrupts via the Core-Local Interruptor (CLINT)
   - External interrupts - platform-level interrupt controller (PLIC)

   - Interrupts are enabled by setting the mstatus.MIE bit and by enabling the desired individual interrupt in the mie register (mie.MSIE/MTIE/MEIE - software/timer/external aka global)
   - Trap mtvec provides a table for exceptions: mtvec.BASE + 4 × mcause.EXCCODE (EXCCODE = bits[9:0]). All machine external interrupts (global interrupts) are mapped to exception code of 11. Thus, when interrupt vectoring is enabled, the pc is set to address mtvec.BASE + 0x2C for any global interrupt.
   - mtval is written with the faulting effective address (breakpoint, instruction-fetch, load, or store address-misaligned, access, or page-fault occurs). On an illegal instruction trap, mtval is written with the first XLEN bits of the faulting instruction as described below.
   - mscratch holds a pointer to a hart-local context space and swapped with a user register upon entry to a trap handle.
      - (All the above applies to m/s/u registers/modes)


### Bare metal boatloaders:
* RISC-V FPGA SmartFusion2  M2S150 Devkit bootloader  https://github.com/RISCV-on-Microsemi-FPGA/RVBM-BootLoader

### Bootlloaders:
* Berkeley Boot Loader
* u-boot — Standard bootloader for embedded Linux systems
* Coreboot

### ProxyKernel
* https://github.com/riscv/riscv-pk The RISC-V Proxy Kernel, pk, is a lightweight application execution environment that can host statically-linked RISC-V ELF binaries

## Devices
Flattened Device Tree (FDT)

Syscalls


## Steps
- Second Stage Bootloader. CPU starts in M-mode (Machine mode).
- Handle hardware threads (HARTs) and setup stack for each.
- Handle miniam device initialization: UART. Use Flattened Device Tree (FDT) when available.
- Transition from M-mode (machine) to S-mode (supervisor).
- Setup memory protection.
- Setup interrupt handlers.
- Basic syscalls.
- Load ELF.

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

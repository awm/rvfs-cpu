# RISC-V from Scratch

The goal of this project is to explore basic CPU design and learn about the RISC-V architecture in particular.  As this
is a hobby project progress may not proceed in a particularly smooth or linear fashion 😃.

## Sub-projects

Construction of the processor is the primary project, however ancillary projects exist for the hardware peripherals,
programming/debugging interface, and simulator:

 * Peripheral Modules: *[TBD]*
 * Simulation Environment: *[TBD]*
 * Debugger/Programmer/Test Interface: [rvfs-testbed](https://github.com/awm/rvfs-testbed)

## Phases

This project is broken into phases, with the requirements for each phase defined in
[doc/requirements.md](docs/requirements.md).

### Phase 0 - Infrastructure

 * Define initial requirements and high level design, constraining some hardware interfaces.
 * Create a basic simulation environment to test out logic blocks.
 * Design and build processor backplane.
 * Design and build programming and debug interface.

### Phase 1 - RV32E Decode and Register Bank

 * Design and build RV32E instruction decode pipeline processor module.
 * Design and build RV32E register bank processor module.

### Phase 2 - Execute and Register Write-back

 * Design and build execution pipeline processor module.
 * Design and build write-back pipeline processor module.

### Phase 3 - System Memory

 * Design and build RAM peripheral module.
 * Design and build Flash peripheral module.

### Phase 4 - Instruction Fetch and Memory Access

 * Design and build instruction fetch pipeline module.
 * Design and build memory access pipeline module.

### Phase 5 - Real World Output

 * Design and build a UART peripheral module.
 * Design and build a GPIO peripheral module.
 * Build a basic "Hello World" application using a standard RV32E toolchain, and run it on the processor.
 * Build a basic "Blinky" application using a standard RV32E toolchain, and run it on the processor.

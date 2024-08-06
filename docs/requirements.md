# Requirements

This document covers requirements for the CPU core and backplane.  For the requirements of other parts of the system see
their respective requirements documents:

 * Simulation Environment: *[TBD]*
 * Peripheral Modules: *[TBD]*
 * Debugger/Programmer/Test Interface Requirements: *[TBD]*

## Fundamental

These are basic requirements that define bounds for the system design space.

 * The CPU and backplane shall be constructed from commercially available integrated circuits.
    * *Rationale: Obtaining the components should be as easy as placing orders to Digi-Key, Mouser, etc.*
    * In general, generic programmable elements shall not be used (i.e. no FPGAs, CPLDs, microcontrollers, ROM look-up
      tables, etc).
      * *Rationale: The objective is to construct the system from discrete components in order to gain knowledge, rather
        than to program it into existence.*
      * General purpose memory (i.e. SRAM and Flash) may be used for the system and instruction memory.
    * BGA packages shall not be used.
      * *Rationale: BGA packages are hard to test for soldering issues and hard to rework in a home lab environment due
        the contacts hidden under the chip.*
    * Surface mount components should be preferred over through hole.
      * *Rationale: Surface mount components generally take up less total board space than through hole.*
      * Through hole components may be used when no surface mount equivalent is available, or when the added mechanical
        stability is desirable (such as for some connectors).
 * The maximum main system clock (instruction execution) speed shall be 1 MHz.
    * *Rationale: This provides a good balance between performance and being able to easily source compatible
      components.*
 * The maximum reference clock speed shall be 8 MHz.
 * It shall be possible to run at lower clock speeds.
 * The power supply voltage for the processor shall be 3.3 V.
    * *Rationale: This voltage node is extremely common and permits use of a wide range of components.  Many parts
      operate at 3.3 V or lower, or at 2-3 V and higher, placing this in a sweet spot for compatibility.*

## Phase 0

 * It shall be possible to externally cycle the reference clock via the test/debug interface.
 * The backplane shall provide at least the following signals to each CPU module:
    * 32-bit memory data bus.
    * 28-bit memory address bus.
    * Memory read/write enable lines.
    * 4-bit socket ID.
    * 16 interrupt lines (one dedicated to each socket).
    * Main clock.
    * Reference clock.
    * Power supply (3.3 V and ground).
    * 32-bit register data bus.
    * 12-bit register selector.
 * CPU modules shall use a 164-contact PCI Express connector to attach to the backplane. **Note: the interface will not
   be electrically compatible with PCIe.**
   * *Rationale: This provides an inexpensive and widely available connector type as the interface point.*
 * The backplane shall provide at least the following signals to each peripheral module:
    * 32-bit memory data bus.
    * 28-bit memory address bus.
    * Memory read/write enable lines.
    * 4-bit socket ID.
    * 16 interrupt lines (one dedicated to each socket).
    * Main clock.
    * Power supply (3.3 V and ground).
 * Peripheral modules shall use a 98-contact PCI Express connector to attach to the backplane. **Note: the interface
   will not be electrically compatible with PCIe.**
   * *Rationale: This provides an inexpensive and widely available connector type as the interface point.*
 * Backplane signals shall be assigned such that a peripheral card may be legally inserted into a CPU card slot, but a
   CPU card may not be inserted into a peripheral slot.

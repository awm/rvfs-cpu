# Bus Architecture

## Main Memory Interface

The main memory interface connects all CPU modules and all peripheral modules.  It is the only way to transfer data
between the CPU and the peripherals.

The data bus `D[31:0]` is 32 bits wide and is used to transfer values from memory to the CPU or vice versa.  The actual
size of the data transferred is determined by the `HALF[1:0]` bits:

| `HALF[1]` | `HALF[0]` | Effect                                      |
| --------: | --------: | ------------------------------------------- |
|       `0` |       `0` | Full 32-bit word transferred  in `D[0:31]`. |
|       `0` |       `1` | 16-bit half word transferred  in `D[0:15]`. |
|       `1` |       `0` | 16-bit half word transferred  in `D[0:15]`. |
|       `1` |       `1` | 8-bit byte transferred  in `D[0:7]`.        |

The address bus `A[27:0]` is 28 bits wide and is used by the CPU to set the memory address of the transfer (source for a
read or destination for a write).  The upper 4 bits (`A[27:24]`) correspond to the fixed socket ID, thus leaving the
remaining 24 bits for addressing within that module.  This gives each module up to 16 MB of the address space.

The write line `/WR` determines if the operation is a read or write:

| `/WR` | Effect                                                               |
| ----: | -------------------------------------------------------------------- |
|   `0` | Data on `D` is being transferred to the address on `A` from the CPU. |
|   `1` | Data on `D` is being transferred from the address on `A` to the CPU. |

The `/DREADY` signal indicates when the value on `D` is ready to be consumed.  This line will be driven by the CPU when
performing a write, and by the peripheral when a read is requested.

The `/AREADY` signal indicates when the value on `A` is ready to be consumed.  This line will be driven by the CPU once
a valid address is present.

Finally, the `/FAULT` line is used by the peripheral to indicate that a memory fault has occurred.  This could indicate
an access to an invalid address, or an unsupported transfer size, for example.

### Summary

| Signal(s)   | Driven By       | Description                                                      |
| ----------- | --------------- | ---------------------------------------------------------------- |
| `/AREADY`   | CPU             | Address bus has settled                                          |
| `/DREADY`   | CPU, Peripheral | Data bus has settled; from CPU on write, from peripheral on read |
| `/FAULT`    | Peripheral      | Memory fault has occurred                                        |
| `/WR`       | CPU             | Transfer direction                                               |
| `A[28:0]`   | CPU             | Memory address                                                   |
| `D[31:0]`   | CPU, Peripheral | Transfer data; from CPU on write, from peripheral on read        |
| `HALF[1:0]` | CPU             | Transfer size                                                    |

**Signal Count:** 66

## Register Interface

The register interface is exclusive to the CPU modules.  It provides the transfer path between CPU registers (all
general purpose registers and CSRs) and the pipeline.

The data bus `D[31:0]` is 32 bits wide and is used to transfer values from registers to the CPU pipeline or vice versa.
All such transfers are 32-bit.

The register selector bus `R[11:0]` is 12 bits wide and selects the index of the register to be accessed.  The general
purpose registers are mapped into the CSR address space at offset `0x800`, which allows each to be accessed at
`0x800 + <register index>`.  The offset of 0x800 corresponds to the beginning of the unprivileged custom read/write
CSRs.

The `/WR` signal determines if the operation is a read or a write:

| `/WR` | Effect                                                         |
| ----: | -------------------------------------------------------------- |
|   `0` | Data on `D` is being written to the register indicated by `R`. |
|   `1` | Data on `D` is being read from the register indicated by `R`.  |

### Summary

| Signal(s) | Driven By     | Description                                             |
| --------- | ------------- | ------------------------------------------------------- |
| `/WR`     | CPU           | Transfer direction                                      |
| `R[11:0]` | CPU           | Register address                                        |
| `D[31:0]` | CPU, Register | Transfer data; from CPU on write, from register on read |

**Signal Count:** 45

## Interrupt Requests

Each module is connected to 16 IRQ lines, `/IRQ[15:0]`, but only drives one line as an output.  The index of the IRQ
line for a module is selected by the ID of the backplane slot which the module is inserted into.  For example, the
peripheral in slot 14 drives only `/IRQ[14]`.

CPU modules are also restricted to only driving the IRQ line corresponding to their slot ID, however they may also read
the value of any IRQ line as necessary.

### Summary

| Signal(s)    | Driven By       | Description                                     |
| ------------ | --------------- | ----------------------------------------------- |
| `/IRQ[15:0]` | CPU, Peripheral | Interrupt request lines for each backplane slot |

**Signal Count:** 16

## Clocking

All modules are provided with `MCLK` which is the main system clock, representing the normal tick at which instructions
are retired (assuming no stalls or other exceptional behaviour).

Up to two memory accesses may occur over the main memory interface for each cycle of `MCLK`: an instruction read in the
first half, and a memory read or write in the second half.

CPU modules are also provided with `RCLK` which is the reference clock for operations such as register accesses or
internal stages of instruction execution.  This clock runs at a frequency of 8 times the `MCLK`.

The clocks may be driven directly from the main clock source at their nominal maximum rate, or may be generated by an
attached debugger at any frequency between 0 Hz and the maximum.

### Summary

| Signal(s) | Driven By                 | Maximum Frequency | Description        |
| --------- | ------------------------- | ----------------: | ------------------ |
| `MCLK`    | Clock Generator, Debugger |             1 MHz | Main system clock  |
| `RCLK`    | Clock Generator, Debugger |             8 MHz | CPU register clock |

**Signal Count:** 2

## Reset Lines

All modules are provided with the main system reset signal `/MRST`.  When asserted low, all modules stop processing and
clear their internal state.  The `MCLK` signal is also gated by `/MRST`, so the clock will stop ticking while in reset.

CPU modules are additionally provided with the CPU reset line `/CRST`.  When asserted low, all CPU modules stop
processing and clear their internal state.  The `RCLK` signal is also gated by `/CRST`, so the clock will stop ticking
while in reset.

Asserting `/CRST` will **not** reset peripheral modules.  This allows the CPU to be held in reset while a programmer or
debugger reads and writes the peripherals, for example to write a new program to flash.  Asserting the `/MRST` signal
also causes `/CRST` to be asserted.

If it is desireable for a specific peripheral to have its own unique reset control, then that can be implemented by
providing a register with an appropriate reset control in the address space of the peripheral.

### Summary

| Signal(s)  | Driven By                 | Description       |
| --------- | -------------------------- | ----------------- |
| `/MRST`   | CPU, Debugger, POR, Button | Main system reset |
| `/CRST`   | CPU, Debugger, `/MRST`     | CPU reset         |

**Signal Count:** 2

## Power Supply

All modules are supplied with +3.3 V and ground via multiple connections.  At least some of these supply connections use
the main power and ground pins on the PCIe connectors, as those are generally rated for higher current.

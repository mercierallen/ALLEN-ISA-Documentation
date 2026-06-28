# Technical Specification: Allen-8 ISA Multi Purpose Unit (MPU)
This document establishes the hardware routing architecture, bit-field segmentation, and behavioral constraints for the Multi Purpose Unit (MPU).

## 1. Instruction Encoding (Type 10)
All instructions routed toward the **`MPU`** must satisfy the core 2-bit **`TYPE`** field constraint (`10`).

```text
Bit: 7 6 5 4 3 2 1 0
[ 1 0 ] [ ? ] [ ? ? ] [ ? ? ? ]
[ TYPE ] [ S ] [ Unit ] [ Opcode ]
```


Bits 7–6 (**`TYPE`**): `10` (Native **`MPU`** Allocation).
Bit 5 (S / Sub-Type Routing Gate):
`0`: Native **`MPU`** Sub-type. The instruction remains within the **`MPU`** pipeline for single-cycle execution.
`1`: EXT Sub-type Deviation. The **`MPU`** decoder is forced into a high-impedance state. The 32 operational slots are routed to the multi-cycle extension infrastructure.
Bits 4–3 (**`UNIT`** Selector): Maps the instruction to one of four hardware blocks.
Bits 2–0 (**`OPCODE`**): 3-bit operational code passed to the target hardware execution unit.

## 2. Hierarchical Decoding Tree
When **`S`** is set to `0`, the state of bits 4–3 branches execution paths into four isolated functional matrices:

### Unit 00: Mono Branch
Execution Unit: Control Flow Matrix
Functional Description: Evaluates conditional flags using a 8-bit ground reference comparator against register **`R3`**.

### Unit 01: Dual Branch
Execution Unit: Behavioral Compare Array
Functional Description: Evaluates parallel hardware comparators hooked directly to inputs **`R1`** and **`R2`**. These bypass the **`ALU`** to prevents any corruption.

### Unit 10: RAM & Subroutine Space
Execution Unit: Memory Execution Logic
Functional Description: Orchestrates data transfers with the **`RAM`** chip and the hardware stack.

### Unit 11: Multi-Byte Memory Operations
Execution Unit: Internal Bus Controls
Functional Description: Reserved for multi-byte sequential memory access hardware.

## 3. Hardware Behavioral Rules
1. **Branch Target Source** : When an **`MPU`** branch execution signal is validated, the address is read from **`R4`** and copy-translated into **`R6`** (**`PC`**).
2. **`ALU` Isolation Rule ** : During the multi-cycle execution of multi-byte memory operations, **`ALU`** is kept inactive. This hardware isolation prevents any corruption.

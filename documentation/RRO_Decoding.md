# Technical Specification: Allen-8 ISA Register to Register Unit (RRO)
This document establishes the hardware routing architecture, bit-field segmentation, and behavioral constraints for the Register to Register Operations (**`RRO`**).

## 1. Instruction Encoding (Type 01)
All instructions routed toward the **`RRO`** must satisfy the core 2-bit **`TYPE`** field constraint (`01`).

```text
Bit: 7 6 5 4 3 2 1 0
[ 0 1 ] [ ? ? ? ] [ ? ? ? ]
[ TYPE ] [ S S S ] [ D D D ]
```

Bits 7–6 (**`TYPE`**): `01` (Native **`RRO`** Allocation).
Bits 5–3 (**`S`**): ???
Bits 2–0 (**`D`**): ???

## 2. Hierarchical Decoding Tree
The hardware decoder evaluates the **`S`** and **`D`** fields using a strict priority routing matrix to branch into four isolated functional zones:

### Zone 1: Core ALU Operations
Routing Gate: **`D`** == `0`
Execution Unit: Arithmetic Logic Unit (**`ALU`**)
Functional Description: Interprets **`S`** field as an **`ALU`** opcode, execution reads from **`R1`** and **`R2`**, and automatically stores the result in **`R3`**.

### Zone 2: Synchronous Clear Operations
Routing Gate: **`S`** == `0` (Evaluated only if Zone 1 is false)
Execution Unit: Constant Zero Routing Logic
Functional Description: Forces a constant value of `0` into the targeted destination register **`D`**.

> [!WARNING]
> **Register 7 Behavior:** Forcing a Synchronous Clear with a destination of `111` (Stack Pointer) will overwrite its current address.
> Clearing **`SP`** to `0` disrupts the descending stack model (which starts at `0xFF`) and may cause immediate memory collisions or application failure if stack execution is in progress.
> To recover or safely initialize, use the **`DSP`** (**`MPU`**) opcode to manually set **`SP`** to `0xFF`.

### Zone 3: Unary and Control Operations
Routing Gate: **`S`** == **`D`** (Evaluated only if Zones 1 and 2 are false)
Execution Unit: Internal Unary Intercept Decoder
Functional Description: Intercepts the redundant **`S`** and **`D`** states to drive unary arithmetic or processor breakpoints without using **`ALU`**.

### Zone 4: Universal Copy Operation (MOV)
Routing Gate: Default / Fallthrough (Evaluated only if Zones 1, 2, and 3 are false)
Execution Unit: General-Purpose Register File Bus
Functional Description: Executes a standard register-to-register copy, transferring the 8-bit value from **`S`** directly into **`D`**.
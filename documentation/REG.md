# Register Architecture of the Allen-8 [ISA](Base_ISA.md)
This document provides a detailed breakdown of the internal register file of the Allen-8 [ISA](Base_ISA.md).
The architecture features 8 internal 8-bit registers, addressed via a 3-bit index. 
To maximize instruction density within a strict 1-byte encoding constraint, each register is primarily mapped to a specific, specialized function within the processor's data path.
Except for **`R0`**, all registers remain permanently interconnected via a shared general-purpose register bus.

## Register Map Reference Table
| Index (Binary) | Name | Hardware Function | Detailed Description |
| :--- | :--- | :--- | :--- |
| `000` | **`R0`** / **`ZR`** | Hardwired Zero | Permanently wired to ground. Read returns `0`. Writes are discarded or intercepted by the instruction decoder. |
| `001` | **`R1`** / **`S1`** | **`ALU`** Source 1 | Serves as the primary data source for Input A of the **`ALU`**. |
| `010` | **`R2`** / **`S2`** | **`ALU`** Source 2 | Serves as the primary data source for Input B of the **`ALU`**. |
| `011` | **`R3`** / **`RE`** | **`ALU`** Result | Default target for **`ALU`** calculation outputs. |
| `100` | **`R4`** / **`IS`** | Immediate Storage | Default landing register for all **`IMM`** type instructions. |
| `101` | **`R5`** / **`MA`** | Memory Address | Primarily holds the 8-bit memory address for all data transfer operations with the **`RAM`** space. |
| `110` | **`R6`** / **`PC`** | Program Counter | Holds the memory address of the next instruction to be fetched. Increments automatically after each cycle unless overwritten. |
| `111` | **`R7`** / **`SP`** | Stack Pointer | Primarily interfaced with the stack sub-system. |

## Hardware Behavioral Rules
1. Branch Testing: Register 3 (**`R3`**) is the primary reference for mono conditional branch testing. The mono condition is always evaluated against the value `0` (e.g., EQU verifies if **`R3`** == `0`).
2. Branching Constraint: When a branch instruction executes, the new target address is always read from **`R4`** and copied directly into **`R6`** (PC).
3. Data Width: All registers are 8 bits wide, except for **`R0`** which remains frozen at `0`.
# Hardware Operation Map: Allen-8 ISA Multi Purpose Unit (MPU)
This document catalogs the functional specifications and mnemonics for all operational slots within the Multi Purpose Unit (**`MPU`**).

## UNIT 00: Mono Branch [8 operations - Control Flow]
This hardware block manages absolute control flow based on the single evaluation of register **`R3`** (**`RE`**) against a hardwired `0` ground reference.

| Opcode | Mnemonic | Condition | Hardware Description |
| :--- | :--- | :--- | :--- |
| `000` | **`EQU`** | **`R3`** == `0` | Branch if zero (Equal). |
| `001` | **`NEQ`** | **`R3`** != `0` | Branch if not zero (Not Equal). |
| `010` | **`LES`** | **`R3`** < `0` | Branch if negative (Less Than). |
| `011` | **`LEQ`** | **`R3`** <= `0` | Branch if negative or zero (Less or Equal). |
| `100` | **`GRE`** | **`R3`** > `0` | Branch if positive (Greater Than). |
| `101` | **`GEQ`** | **`R3`** >= `0` | Branch if positive or zero (Greater or Equal). |
| `110` | **`ALW`** | True | Unconditional branch (Always). Forces **`R4`** -> **`R6`**. |
| `111` | **`NEV`** | False | Never branch (NOP / No Operation). |

## UNIT 01: Dual Branch [8 operations - Control Flow]
This hardware block manages dual control flow by evaluating behavioral hardware comparators hooked directly to inputs **`R1`** and **`R2`**.

| Opcode | Mnemonic | Condition | Hardware Description |
| :--- | :--- | :--- | :--- |
| `000` | **`EQR`** | **`R1`** == **`R2`** | Branch if Equal. |
| `001` | **`NQR`** | **`R1`** != **`R2`** | Branch if Not Equal. |
| `010` | **`LSR`** | **`R1`** < **`R2`** | Branch if Less Than (Signed). |
| `011` | **`LQR`** | **`R1`** <= **`R2`** | Branch if Less or Equal (Signed). |
| `100` | **`GRR`** | **`R1`** > **`R2`** | Branch if Greater Than (Signed). |
| `101` | **`GQR`** | **`R1`** >= **`R2`** | Branch if Greater or Equal (Signed). |
| `110` | **`CAR`** | (**`R1`** + **`R2`**) > 255 | Branch on Carry (ALU unsigned overflow). |
| `111` | **`OVR`** | (R1[7] == R2[7]) && (R3[7] != R1[7]) | Branch on Overflow (2's complement test). |

## UNIT 10: RAM & Subroutine Space [8 operations - Memory]
This hardware block manages native 8-bit memory data exchange, hardware stack interactions, and execution contexts.

| Opcode | Mnemonic | Hardware Operation | Functional Description |
| :--- | :--- | :--- | :--- |
| `000` | **`STR`** | **`R3`** -> RAM[R5] | Store: Writes **`R3`** into **`RAM`** at address **`R5`**. |
| `001` | **`LOA`** | RAM[R5] -> **`R3`** | Load: Reads **`RAM`** at address **`R5`** into **`R3`**. |
| `010` | **`PSH`** | SP - 1 -> SP; **`R3`** -> RAM[SP] | Push: Pushes **`R3`** onto the hardware stack. |
| `011` | **`POP`** | RAM[SP] -> **`R3`**; SP + 1 -> SP | Pop: Pops the top stack value into **`R3`**. |
| `100` | **`CAL`** | SP - 1 -> SP; **`R6`** + 1 -> RAM[SP]; **`R4`** -> **`R6`** | Call: Pushes next PC to stack and jumps to absolute **`R4`**. |
| `101` | **`RET`** | RAM[SP] -> **`R6`**; SP + 1 -> SP | Return: Pops the saved address from stack back into PC (**`R6`**). |
| `110` | **`PKL`** | Stack[Top] -> **`R3`** | Peek Last: Copies top stack value into **`R3`** without deletion. |
| `111` | **`PKF`** | RAM[0] -> **`R3`** | Peek First: Inspects absolute address `0` and copies into **`R3`**. |

## UNIT 11: Multi-Byte Memory Operations [8 operations - Memory]
This hardware block expands the memory subsystem to handle 16-bit half-words and 32-bit words over sequential byte-access hardware.

| Opcode | Mnemonic | Hardware Operation | Functional Description |
| :--- | :--- | :--- | :--- |
| `000` | **`LHW`** | RAM[R5:R5+1] -> **`R1`**:**`R2`** | Load Half-Word: Loads 16 bits into **`R1`** (High) and **`R2`** (Low). |
| `001` | **`SHW`** | **`R1`**:**`R2`** -> RAM[R5:R5+1] | Store Half-Word: Writes 16 bits from **`R1`**:**`R2`** into **`RAM`** at **`R5`**. |
| `010` | **`LWD`** | RAM[R5:R5+3] -> **`R1`**:**`R4`** | Load Word: Loads 32 bits from **`RAM`** into **`R1`**, **`R2`**, **`R3`**, and **`R4`**. |
| `011` | **`SWD`** | **`R1`**:**`R4`** -> RAM[R5:R5+3] | Store Word: Writes 32 bits from **`R1`**:**`R4`** into **`RAM`** at **`R5`**. |
| `100` | **`PUH`** | SP - 2 -> SP; **`R1`**:**`R2`** -> RAM[SP:SP+1] | Push Half-Word: Decrements SP by `2`, pushes 16 bits from **`R1`**:**`R2`**. |
| `101` | **`POH`** | RAM[SP:SP+1] -> **`R1`**:**`R2`**; SP + 2 -> SP | Pop Half-Word: Pops 16 bits from stack into **`R1`**:**`R2`**, increments SP by `2`. |
| `110` | **`PUW`** | SP - 4 -> SP; **`R1`**:**`R4`** -> RAM[SP:SP+3] | Push Word: Decrements SP by `4`, pushes 32 bits from **`R1`**:**`R4`**. |
| `111` | **`POW`** | RAM[SP:SP+3] -> **`R1`**:**`R4`**; SP + 4 -> SP | Pop Word: Pops 32 bits from stack into **`R1`**:**`R4`**, increments SP by `4`. |
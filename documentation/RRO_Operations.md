# Hardware Operation Map: Register to Register Unit (RRO)
This document catalogs the complete functional specifications and mnemonics for all operational slots within the Register to Register Operations (**`RRO`**) unit.

## Zone 1: Core ALU Operations
| Opcode | Mnemonic | Operation | Functional Description |
| :--- | :--- | :--- | :--- |
| `000` | **`AND`** | **`R1`** & **`R2`** -> **`R3`** | Bitwise logical **`AND`** |
| `001` | **`OR`** | **`R1`** \| **`R2`** -> **`R3`** | Bitwise logical **`OR`** |
| `010` | **`ADD`** | **`R1`** + **`R2`** -> **`R3`** | Arithmetic addition |
| `011` | **`SUB`** | **`R1`** - **`R2`** -> **`R3`** | Arithmetic subtraction |
| `100` | **`NOT`** | ~**`R1`** -> **`R3`** | Bitwise inversion of Input A (**`R2`** is ignored) |
| `101` | **`XOR`** | **`R1`** ^ **`R2`** -> **`R3`** | Bitwise logical **`Exclusive OR`** |
| `110` | **`SHL`** | **`R1`** << **`R2`** -> **`R3`** | Bitwise logical shift left of **`R1`** by **`R2`** bits |
| `111` | **`SHR`** | **`R1`** >> **`R2`** -> **`R3`** | Bitwise logical shift right of **`R1`** by **`R2`** bits |

## Zone 2: Synchronous Clear Operations
| Mnemonic | Data Path | Functional Description |
| :--- | :--- | :--- |
| CLR **`DST`** | `0` -> **`DST`** | Clears the specified destination register to 0 |

## Zone 3: Unary and Control Operations
| Index | Mnemonic | Hardware Operation | Functional Description |
| :--- | :--- | :--- | :--- |
| `001` | NG1 | -**`R1`** -> **`R1`** | Two's complement arithmetic negation of Input A (**`R1`**) |
| `010` | NG2 | -**`R2`** -> **`R2`** | Two's complement arithmetic negation of Input B (**`R2`**) |
| `011` | NG3 | -**`R3`** -> **`R3`** | Two's complement arithmetic negation of **`ALU`** result (**`R3`**) |
| `100` | NG4 | -**`R4`** -> **`R4`** | Two's complement arithmetic negation of constants (**`R4`**) |
| `101` | IMA | **`R5`** + `1` -> **`R5`** | Increment by `1` the **`RAM`** address register (**`R5`**)|
| `110` | BRK | Halt **`R6`** | Signals software breakpoint and halts the program counter (**`R6`**) |
| `111` | ISP | **`R7`** + `1` -> **`R7`** | Increment by `1` the stack pointer register (**`R7`**)|

## Zone 4: Universal Copy Operation (MOV)
| Mnemonic | Data Path | Functional Description |
| :--- | :--- | :--- |
| **`MOV`** **`S`**, **`D`** | **`S`** -> **`D`** | Copies the 8-bit value from **`S`** directly into **`D`** |
# Technical Specification of the Allen-8 ISA (RISC/Harvard Architecture)
The Allen-8 ISA is an 8-bit **`RISC`**/**`Harvard`** architecture with fully isolated instruction and data memory paths.
It features 8-bit wide registers holding strictly sign-agnostic data across all hardware components. 
Consequently, execution units treat data as neutral bit-fields by default, dynamically switching between unsigned boundaries and signed two's complement decoding based entirely on the chosen operational opcode.

## 1. Register Architecture
See [REG.md](REG.md) for details.

## 2. Instruction Encoding (8-bit)
Every instruction is 1 byte, with bits 7-6 defining the **`TYPE`**.

```text
Bit: 7 6 5 4 3 2 1 0
[ TYPE ] [ Opcode/Constant ]
```

*See [Decoding_Tree.md](Decoding_Tree.md) for a complete structural visualization of the hardware execution paths.*

### Type 00: IMM (Immediate Storage)
Loads a 6-bit constant (0-63) directly into register **`R4`**.

### Type 01: RRO (Register to Register Operations)
Activates the **`RRO`** unit, using 6 bits to define **`SRC`** and **`DST`** registers.
*See [RRO_Decoding.md](RRO_Decoding.md) for details.*

### Type 10: MPU (Multi Purpose Unit)
Activates the **`MPU`** for control flow, **`RAM`**, and stack operations using 3-bit Opcode/Zone fields.
*See [MPU_Decoding.md](MPU_Decoding.md) for details.*

### Type 11: EXT (Extensions)
Handles extended multi-byte instructions.
*See [EXT.md](EXT.md) for details.*
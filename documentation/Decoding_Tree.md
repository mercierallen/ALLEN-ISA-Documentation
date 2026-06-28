# Hardware Decoding Tree of the Allen-8 ISA

This document provides a structural visualization of the hardware decoding logic. The instruction decoder evaluates the 8-bit instruction opcode sequentially, branching through hardware routing gates to activate the correct execution units.

```mermaid
graph TD
%% Styling
classDef root fill:#f9f,stroke:#333,stroke-width:2px;
classDef type fill:#bbf,stroke:#333,stroke-width:2px;
classDef gate fill:#ffe3cc,stroke:#333,stroke-width:1px;
classDef unit fill:#d4edda,stroke:#333,stroke-width:2px;

%% Instruction Root
Instruction[Fetched Instruction <br> 1 Byte / Bits 7-0] --> TypeSplit{Evaluate TYPE <br> Bits 7-6}
class Instruction root;

%% =========================================================================
%% TYPE 00: IMM
%% =========================================================================
TypeSplit -- "00" --> IMM_Unit[IMM Unit <br> Load Bits 5-0 into **`R4`**]
class IMM_Unit unit;

%% =========================================================================
%% TYPE 01: RRO
%% =========================================================================
TypeSplit -- "01" --> RRO_Tree{RRO Priority Matrix <br> Evaluate SRC & DST}
class RRO_Tree type;

RRO_Tree-->|1.D==0|RRO_Z1[Zone 1: Core ALU <br> Save in R3]
RRO_Tree-->|2.S==0|RRO_Z2[Zone 2: Sync Clear <br> 0 -> DST]
RRO_Tree-->|3.S==D|RRO_Z3[Zone 3: Unary & Control <br> NG1-NG4, IMA, BRK, DSP]
RRO_Tree-->|4.Fallthrough|RRO_Z4[Zone 4: Universal MOV <br> SRC -> DST]

class RRO_Z1,RRO_Z2,RRO_Z3,RRO_Z4 unit;

%% =========================================================================
%% TYPE 10: MPU
%% =========================================================================
TypeSplit -- "10" --> MPU_S{Evaluate Sub-Type <br> Bit 5}
class MPU_S type;

MPU_S -- "S == 1" --> EXT_Deviation[EXT Sub-type Deviation <br> Multi-cycle Infrastructure]
class EXT_Deviation unit;

MPU_S -- "S == 0" --> MPU_Unit{Evaluate UNIT <br> Bits 4-3}
class MPU_Unit type;

MPU_Unit -- "00" --> MPU_U00[Unit 00: Mono Branch <br> Compare **`R3`** against 0]
MPU_Unit -- "01" --> MPU_U01[Unit 01: Dual Branch <br> Parallel Comparators **`R1`** & **`R2`**]
MPU_Unit -- "10" --> MPU_U10[Unit 10: Native 8-bit MEM <br> RAM & Stack Operations via **`R5`**]
MPU_Unit -- "11" --> MPU_U11[Unit 11: Multi-Byte MEM <br> 16-bit / 32-bit Hardware Access]

class MPU_U00,MPU_U01,MPU_U10,MPU_U11 unit;

%% =========================================================================
%% TYPE 11: EXT
%% =========================================================================
TypeSplit -- "11" --> EXT_Unit[EXT Unit <br> Multi-byte Instruction Extensions]
class EXT_Unit unit;
```

---
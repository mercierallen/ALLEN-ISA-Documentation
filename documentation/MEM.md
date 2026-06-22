# Technical Specification: Allen-8 ISA Memory Architecture (MEM)
This document establishes the hardware memory layout, behavioral constraints, and the standard convention map (Memory Map) for the Allen-8 ISA plat shared data space.

## 1. Physical Architecture and Isolation
The Allen-8 ISA implements a **`Harvard`** Architecture layout for bus routing during execution, featuring fully isolated instruction (ROM) and data (**`RAM`**) memory paths. 

However, within the Data **`RAM`** subsystem, no physical hardware separation exists. The Unified **`RAM`** space is concurrently shared by:
Direct **`RAM`** Operations: STR and LOA (addressed via register **`R5`** / MA)
Stack Operations: PSH, POP, CAL, and RET (addressed via register **`R7`** / SP)

All operations interact with the exact same physical silicon bank.

---

## 2. Standard 8-bit Memory Map Convention
Since the hardware does not enforce memory protection or boundaries, a strict logical Memory Map is required to prevent data corruption between variables and the hardware stack.
By standard convention, the 256-byte **`RAM`** space (0x00 to 0xFF) is segmented as follows:

| Address Range | Segment Name | Access Type | Functional Description |
| :--- | :--- | :--- | :--- |
| 0x00 | ZR / Peek First | Read Only | Hardwired to ground reference. Intercepted by PKF to verify hardware alignment. |
| 0x01 – 0x1F | Hardware I/O & MMIO | Read / Write | Reserved for Memory-Mapped Input/Output devices (timers, UART, peripherals). |
| 0x20 – 0x7F | Global Data Space | Read / Write | Standard variable storage accessed via STR and LOA. Grows upward (incrementing). |
| 0x80 – 0xBF | Extended Heap / Temp | Read / Write | Buffer zone for large data processing or temporary structural structures. |
| 0xC0 – 0xFF | Hardware Stack Space | Push / Pop | Dedicated to subroutine contexts. Grows downward (decrementing) toward 0x80. |

### Stack Initialization Rule
At system boot or execution reset, software MUST initialize the Stack Pointer (**`R7`** / SP) to 0xFF before any PSH or CAL instruction is executed.

---

## 3. Hardware Behavioral Rules & Side-Effects
1. Stack and Subroutine Interaction: When a CAL instruction executes, the return address (PC + 1) is written into the data **`RAM`** at RAM[SP]. This bridges the **`Harvard`** gap, making the Data **`RAM`** a temporary custodian of control flow pointer metadata.
2. Stack Pointer Visibility: Because **`R7`** (SP) is wired to the general register bus, its value can be copied directly into **`R5`** (MA). This allows the processor to execute raw LOA or STR commands directly inside the stack frame to modify parameters without performing a POP.
3. Collision Hazard (Stack Overflow): If the Stack grows past 0xC0 downward into the 0x7F space due to excessive recursive CAL or PSH loops, it will silently overwrite active Global Variables. Hardware does not generate a fault; memory integrity is purely software-enforced.

---

## 4. Multi-Byte Alignment & Extension Map (16/32/64-bit)
When operating the Multi-Byte Memory block (Unit 11) or utilizing the EXT infrastructure, data width expands beyond the native 8-bit boundaries. The system utilizes a Big-Endian sequence layout by default.

### Layout Sequences:
Half-Word (16-bit / LHW, SHW): 
Uses 2 bytes. If base address in **`R5`** is N, RAM[N] holds bits 15-8 (High Byte) and RAM[N+1] holds bits 7-0 (Low Byte).
Word (32-bit / LWD, SWD): 
Uses 4 bytes sequentially from N to N+3.
Double Word (64-bit / EXT Mode): 
Uses 8 bytes sequentially from N to N+7.

### Structural Alignment Warning
When executing a 32-bit Word transfer (LWD/SWD) or 64-bit operations, the base address in **`R5`** must never be positioned within 4 or 8 bytes of the active Stack Pointer boundary (SP), otherwise, the multi-cycle execution loop will corrupt the stack context mid-transfer.
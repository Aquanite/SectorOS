1.	DESCRIPTION

BSlash (abbreviated as B/) is a modern 32-bit processor architecture designed to succeed and supersede the original BARC architecture. B/ introduces a scalable and modular instruction format, a richer and more extensible register model, and a robust execution model that aligns with the demands of real-world software and hardware systems.

Developed with performance, clarity, and extensibility in mind, B/ is equally suitable for embedded systems, operating system development, educational instruction, and experimental CPU designs. It is structured to balance simplicity in decoding with a comprehensive feature set necessary for general-purpose computing.

2.	PURPOSE

The original BARC design, while minimal and instructive, proved insufficient for scalable or real-world applications due to its rigid 8-bit instruction width and lack of operand encoding. B/ addresses these limitations by adopting a modular instruction structure wherein operands follow opcodes and can vary in length and type.

This design allows for efficient register-register operations, 32-bit immediate loads, robust control flow mechanisms, and low-level memory access — all while maintaining a clean and efficient instruction decoding path. By enabling a rich instruction set without compromising simplicity, B/ strikes a pragmatic balance between academic readability and industrial applicability.

<!-- table -->
| | |
|---|---|
| **Architecture Name** | BSlash (B/) |
| **Word Size** | 32 bits |
| **Instruction Width** | Variable (8-bit opcode + variable-length operands) |
| **Instruction Alignment** | Byte-aligned (8-bit) |
| **Endianness** | Little-endian |
| **Register Set** | 16 general-purpose registers (B0–B15), special registers (IP, STK, FR, STAT), control registers (CTL0–CTL7: MODE, VECBASE, CYCOUNT, INFO, SYSCALLNO, FCSR, MKMR, CAUSE) |
| **Address Space** | 4 GiB (32-bit addressing) |
| **Stack Architecture** | Full descending stack |
| **Calling Support** | Yes (stack-based) |
| **I/O Mechanisms** | Memory-mapped I/O; Port-mapped I/O
| **Addressing Modes** | Register, immediate, direct, indirect, indexed |
| **STAT Register** | Zero (Z), Sign (S), Carry (C), Overflow (O), Parity (P), Interrupt Enable (IE) |

1.	ARCHITECTURAL INTENT

The BSlash (B/) architecture was conceived to deliver a compact, performant, and extensible instruction set suitable for both theoretical and practical computing systems. B/ achieves this by balancing structured simplicity with flexible execution, enabling both low-level control and high-level expressiveness.

2.1.	DESIGN OBJECTIVES

- **Simplicity and Clarity**: B/ maintains a straightforward instruction format that is easy to decode and understand, facilitating educational use and rapid implementation.

- **Extensibility**: The architecture is designed to accommodate future enhancements, allowing for the addition of new instructions and features without disrupting existing functionality.

- **Performance**: B/ aims to deliver efficient execution through a well-defined instruction set and optimized register usage, supporting both high-level language constructs and low-level operations.

- **Versatility**: The architecture supports a wide range of applications, from embedded systems to operating systems, making it a versatile choice for various computing needs.

2.2.	DESIGN OBJECTIVES (Cont.)
- **I/O and Memory Flexibility**:
    1. B/ supports both memory-mapped and port-mapped I/O, providing flexibility in interfacing with peripheral devices.
    2. The architecture includes multiple addressing modes to facilitate diverse memory access patterns.

- **Simplicity of Implementation**:
The architecture prioritizes implementation clarity:
1. Opcodes are uniquely identifiable without complex prefix trees.
2. Operand formats are predictable and follow a small number of patterns.
3. Instruction decoding logic can be implemented with minimal state. 

- **Compiler and OS Friendliness**:
The architecture’s clean structure is optimized for:
1. Code generation by C-like compilers
2. Symbolic debugging and disassembly
3. Low-level OS features like context switching, interrupt handling, and task isolation


1.	SYSTEM ARCHITECTURE OVERVIEW

The BSlash (B/) architecture defines a clean, minimalistic system model designed for extensibility and practical implementation in both software and hardware. This section outlines the core structural components that make up a typical B/ system and how they interact. At its core, a B/ system is composed of the following subsystems:

1.1.	Central Processing Unit (CPU)
The CPU is the heart of the B/ architecture, responsible for executing instructions and managing data flow. It consists of:
- **Registers**: A set of 16 general-purpose registers (B0–B15)
- **Instruction Pointer (IP)**: Points to the next instruction to be executed
- **Stack Top (STK)**: Manages the call stack for function calls and local variables
- **Frame Register (FR)**: Used for stack frame management
- **Status Register (STAT)**: Holds status flags (Zero, Sign, Carry, Overflow, Parity, Interrupt Enable)
- **Control Registers (CTL0–CTL7)**: Used for system configuration and control
    1. CTL0 (MODE): System Mode Register, defines operating modes (bit 0 = 0 for kernel, bit 0 = 1 for user)
    2. CTL1 (VECBASE): Interrupt Vector Base, sets the base address for interrupt vectors
    3. CTL2 (CYCOUNT): Cycle Counter, used for performance monitoring, increments with each CPU cycle
    4. CTL3 (INFO): CPU Information Request Register, provides details about CPU features and capabilities. After writing to CTL3, it will reset to 0 and populate registers B0–B3 based on the value written:
        4.1. If CTL3 is set to 1, the CPU sets its brand string in registers B0–B3 as follows: (example for a hypothetical CPU)
            - B0: "BSlh"
            - B1: "32  "
            - B2: "CPU "
            - B3: "2025"
    4.2. If CTL3 is set to 2, the CPU will set its date of manufacture in register B0 in the format YYYYMMDD as a 32-bit unsigned integer (binary). For example, a CPU manufactured on June 15, 2025, sets B0 to the integer value 20250615.
        4.3. If CTL3 is set to 3, the CPU will set its unique serial number in registers B0 and B1. The serial number is a 64-bit value, with the higher 32 bits in B0 and the lower 32 bits in B1.
        4.4. If CTL3 is set to 4, the CPU will return feature bitmap group 0 in B0 (B1-B3 reserved as 0). Defined bits:
            - B0 bit 0: CICPY instruction available
            - B0 bits 1-31: Reserved for future feature flags
        4.5. More functions may be defined for CTL3 in future revisions of the architecture.
    4.6. CTL3 brand string encoding (selector=1): B0–B3 together return exactly 16 ASCII characters. Within each 32-bit register, bytes are ordered little-endian; i.e., the least-significant byte holds the first character of that 4-character block. If B0–B3 are stored to memory as four consecutive 32-bit little-endian words, the resulting 16 bytes in memory form the brand string left-to-right.
    4.7. CTL3 clobbers: Writing CTL3 for any selector may clobber B0–B3 as described. Software must treat B0–B3 as volatile across a CTL3 transaction (including the CPUID instruction).
    5. CTL4 (SYSCALLNO): Syscall Number Register, used to specify the syscall number when making a system call (first 8 bits are used to indicate the syscall number; remaining bits are reserved for future use). **Kernel-only write**; user mode may not modify CTL4.
    6. CTL5 (FCSR): FP Status Control Register, manages floating-point unit settings such as rounding modes and exception flags
        - Bits 0-2: Rounding Mode
            - 000: Round to Nearest (even)
            - 001: Round toward Zero
            - 010: Round toward Positive Infinity
            - 011: Round toward Negative Infinity
            - 100-111: Reserved
        - Bits 3-7: Exception Flags (sticky)
            - Bit 3: Invalid Operation
            - Bit 4: Division by Zero
            - Bit 5: Overflow
            - Bit 6: Underflow
            - Bit 7: Inexact Result
    7. CTL6 (MKMR): Memory Key Mask Register (MKMR). Used by keyed memory access instructions to authorize access to memory regions.
        - Bits 0-7: Key mask (implementation-defined semantics for region-to-key mapping)
        - Bits 8-31: Reserved (0)
    8. CTL7 (CAUSE): Exception Cause Register. Readable in all modes; writable in kernel mode. Encodes the most recent synchronous fault/exception cause and optional subcodes.
        - Bits 0–7: Cause code
            - 0x00: None
            - 0x01: Protection fault (key or privilege)
            - 0x02: Illegal instruction/encoding
            - 0x03: Alignment fault (unaligned access)
            - 0x04: Page fault (not present)
            - 0x05: Divide by zero (integer or FP)
            - 0x06: Breakpoint (BRK)
            - 0x07: System call (trace)
            - 0x08–0xFF: Reserved
        - Bits 8–31: Reserved (0)
    A separate read-only BADADDR register is exposed via RDBAD (see MMU/Debug ops) to report the faulting address when applicable.

1.2.	Memory Subsystem
The memory subsystem in a B/ system is designed to provide efficient data storage and retrieval. It includes:
- **Main Memory**: A flat 4 GiB address space, byte-addressable
- **Stack Memory**: A dedicated area for function call management, growing downwards from higher, which is managed by the STK and FR registers
- **I/O Memory**: Supports both memory-mapped and port-mapped I/O for peripheral device interaction

1.2.1.   Alignment and Unaligned Access Policy
- Word (32-bit) and halfword (16-bit) accesses must be naturally aligned (addr % 4 == 0 for 32-bit, addr % 2 == 0 for 16-bit). Byte accesses have no alignment restrictions.
- Unaligned 32-bit or 16-bit accesses raise an Alignment Fault and set CTL7 (CAUSE) = 0x03. RDBAD returns the faulting address.
- Implementations may optionally support unaligned accesses with automatic fix-up; if so, they must advertise this via a platform-specific CPUID selector and guarantee atomicity only at the natural alignment granularity.

1.3.	Input/Output (I/O) Subsystem
The I/O subsystem facilitates communication between the CPU and external devices. It includes:
- **Memory-Mapped I/O**: Allows peripherals to be accessed as if they were memory locations
- **Port-Mapped I/O**: Provides a separate address space for I/O operations.

1.4.    Instruction Execution Model
The B/ architecture employs a straightforward instruction execution model:
- **Fetch**: The CPU retrieves the next instruction from memory using the IP.
- **Decode**: The instruction is decoded to determine the operation and operands.
- **Execute**: The operation is performed, and results are written back to registers or memory
- **Update IP**: The IP is updated to point to the next instruction, unless modified by control flow instructions

1.5.    Interrupt Handling
The B/ architecture includes a robust interrupt handling mechanism:
    The saved STK value recorded in the frame is the pre-interrupt STK (before any hardware pushes); implementations may use an internal shadow stack to perform the push sequence.
1.5.1.    Vector Table Layout (recommended)

While handler selection is OS-defined, B/ recommends a simple vector table layout indexed by a vector ID at VECBASE (CTL1):

- Table format: At address `VECBASE + 4*id`, store a 32-bit absolute handler address for vector `id`.
- Recommended IDs:
        - 0x00: Reset (initial entry point on power-on)
        - 0x01: Protection fault (CAUSE=0x01)
        - 0x02: Illegal instruction/encoding (CAUSE=0x02)
        - 0x03: Alignment fault (CAUSE=0x03)
        - 0x04: Page fault (CAUSE=0x04)
        - 0x05: Divide by zero (CAUSE=0x05)
        - 0x06: Breakpoint (CAUSE=0x06)
        - 0x07: System call dispatcher (optional single-vector ABI)
        - 0x80–0x8F: External IRQ lines 0–15

On trap/interrupt, hardware sets CTL7 (CAUSE) and pushes the context as specified. Software may consult CTL7 and dispatch, or jump directly to the vector derived from the event source. Platforms are free to extend the table with additional IDs.

1.6.    System Modes
The B/ architecture supports multiple operating modes to enhance security and stability:
- **User Mode**: Restricted access to system resources, suitable for application-level code
- **Kernel Mode**: Full access to all system resources, intended for operating system code
The current mode is determined by the System Mode Register (CTL0).

1.7.    Reset and Boot State

Unless overridden by platform firmware, a conforming implementation resets architectural state as follows:

- General-purpose registers B0–B15: undefined
- IP: loaded from `[CTL1.VECBASE + 0x00]` (Reset vector)
- STK: implementation-defined; firmware should initialize before enabling user code
- FR: 0x00000000
- STAT: 0x00 (all flags cleared; IE=0)
- CTL0 (MODE): kernel mode
- CTL1 (VECBASE): implementation-defined (commonly 0x00000000)
- CTL2 (CYCOUNT): 0x00000000
- CTL3 (INFO): 0x00000000
- CTL4 (SYSCALLNO): 0x00000000
- CTL5 (FCSR): 0x00000000 (round-to-nearest-even; all FP exception flags cleared)
- CTL6 (MKMR): 0x00000000 (no keys enabled)
- CTL7 (CAUSE): 0x00000000 (no cause)

Firmware is responsible for installing a valid vector table at VECBASE prior to transferring control to the OS or application code.

1.8.    Multi-core model

B/ permits implementations to expose multiple symmetric cores that share the same 4 GiB address space, MMIO map, and architectural register state layout. All CTL registers, STAT, and general-purpose registers are per-core; memory is shared and coherence is the responsibility of the implementation (the reference emulator models fully coherent shared memory).

- **Core identity**: Core IDs are zero-based. Core 0 is the bootstrap/original core that executes the reset vector. Additional cores (IDs 1..N-1) are auxiliary cores that begin parked with an implementation-defined IP and STK until released by firmware or the OS. The architected way to discover the core topology is the EXT/MTC* instruction family (see CPU/System Instructions).
- **Privilege**: Bringing up or reconfiguring another core is privileged. User-mode code may query counts/identity, but only kernel-mode code (CTL0.MODE bit 0 == 0) may change another core’s entry address or send inter-processor interrupts.
- **Shared memory**: All cores observe the same memory image; synchronization follows the architectural memory model and uses the existing FENCE/FENCE.R/FENCE.W/FENCE.I and LL/SC primitives. Software must use the usual ordering rules when handing off work to another core.
- **Remote entry programming**: Kernel code may program an auxiliary core’s entry IP/context address using EXT.MTSETIP and then wake it with an inter-processor interrupt (EXT.MTI). Implementations should ensure the remote core observes a consistent instruction stream (issue FENCE.I after writing code that the target will execute).


## Program load at 0xA000_0000 and user stack setup

This section describes a practical, platform-agnostic flow for starting code linked at virtual address 0xA000_0000 from ROM/FLASH and how the initial stack is established by user code.

Assembler note: When emitting a flat binary, the `bas` assembler now supports setting the load origin directly inside the source via `%origin #0xA0000000` (or any 32-bit constant/expression). This is equivalent to passing `--origin 0xA0000000` on the command line, and the directive is rejected if it conflicts with a CLI-provided origin. Only one `%origin` may appear per translation unit, and it has no effect when assembling relocatable `.bso` objects.

### Memory model expectations

- The platform must map the code region at virtual address 0xA000_0000 as readable and executable at reset (either directly XIP from ROM/FLASH or copied into RAM prior to execution).
- If an MMU or simple MPU is present, the firmware must arrange page/region attributes so that instruction fetches from 0xA000_0000 succeed (no page/protection faults).
- Data RAM used for the initial stack must be mapped writable and word-aligned (4-byte aligned as required by the ABI; higher alignment is optional).

### Two common boot models

1) Execute-In-Place (XIP) from ROM/FLASH at VA 0xA000_0000
- Link the application text segment with VMA = 0xA000_0000.
- Reset vector points to a small ROM stub which sets up the stack and jumps to the application entry at 0xA000_0000.

Boot ROM stub (XIP) — example
```
// Assumptions:
//  - VECBASE reset vector jumps here
//  - Top-of-RAM for stack is 0x2000_0000 (example)
//  - Application entry is linked at 0xA000_0000

    MOVI32  B0, #0x20000000   // desired initial STK (must be 4-byte aligned by policy)
    WRSTK   B0                 // STK = 0x2000_0000
    MOVI32  B1, #0x00000000    // optional: clear B1.. as policy
    WRFR    B1                 // FR = 0
    // Jump to absolute 0xA000_0000 using J32 with REL32 = target - next_ip
    // Assembler computes REL32; label form is preferred in practice.
    J32     #((0xA0000000 - (IP+5)) & 0xFFFFFFFF)
```

Notes
- Because J32 uses a 32-bit signed-relative displacement added to the next IP, it can reach any 32-bit address; assemblers should compute the correct REL32 from labels or absolute constants.

2) Copy-to-RAM then execute at VA 0xA000_0000
- Firmware/bootloader copies the image from its ROM/FLASH load address (LMA) to RAM at VMA 0xA000_0000.
- After the copy, it sets the stack and jumps to the entry as above.

Boot ROM stub (copy) — example
```
// Copy [src=ROM_LMA .. src+size) to [dst=0xA0000000 ..)
    MOVI32  B0, #ROM_LMA       // source pointer
    MOVI32  B1, #0xA0000000    // destination pointer
    MOVI32  B2, #IMAGE_SIZE    // byte count (multiple of 4 recommended)
copy_loop:
    LDBU    B3, [B0 + #0]
    STB     [B1 + #0], B3
    ADDI8   B0, #1
    ADDI8   B1, #1
    SUBI8   B2, #1
    BNZ     copy_loop

// Set initial stack and jump to entry
    MOVI32  B4, #STACK_TOP
    WRSTK   B4
    WRFR    B1                 // FR = 0 (reuse cleared reg) or explicit 0
    J32     #((0xA0000000 - (IP+5)) & 0xFFFFFFFF)
```

### User-controlled stack initialization

If the design requires the application to set its own stack on early entry (e.g., entry runs with an arbitrary STK from firmware), user code may do:

```
    MOVI32  B0, #STACK_TOP     // choose an aligned stack top (e.g., end of RAM)
    WRSTK   B0                 // set STK directly
    WRFR    B1                 // optional: set FR to 0 or a chosen base
    ENTERI16 #frame_bytes      // establish a canonical frame if desired
    // ... proceed with program
```

Requirements
- Maintain 4-byte alignment at call boundaries (round frame_bytes accordingly).
- Ensure STACK_TOP obeys the platform’s RAM map and doesn’t collide with program/data sections.

### Linking and image layout

- Link the application with VMA = 0xA000_0000 for .text (code) and place .data/.bss in RAM as appropriate.
- When using XIP, the LMA of .text equals its VMA (executed in place). For copy-to-RAM, LMA is the ROM/FLASH address, VMA is 0xA000_0000.
- Toolchains should emit relocations so the bootloader doesn’t need to relocate code at runtime; absolute references within the image should be encoded via MOVI32 or resolved as RIP-relative loads (LDRIP) when possible.

Troubleshooting
- If a page/protection fault occurs on first fetch, verify that the MMU/MPU maps 0xA000_0000 as executable and that CTL7 (CAUSE) reports 0x04 (page fault) or 0x01 (protection fault). Use RDBAD to read the faulting address.
- After self-modifying code or copying code into its final location, issue FENCE.I before jumping into it to synchronize the instruction stream.


1.	TERMINOLOGY & NOTATION

This section defines the standard symbols, formatting rules, and terminology used throughout the BSlash (B/) architecture specification. By establishing strict conventions, we ensure consistency across instruction listings, diagrams, assembler syntax, and documentation tooling.

| Symbol/Term | Definition |
|-------------|------------|
| **B0–B15** | General-purpose registers numbered from 0 to 15. |
| **IP** | Instruction Pointer register, holds the address of the next instruction to execute. |
| **STK** | Stack Top register, points to the top of the stack. |
| **FR** | Frame Register, used for stack frame management. |
| **STAT** | Status register containing condition flags (Zero, Sign, Carry, Overflow, Parity, Interrupt Enable). |
| **CTL0–CTL7** | Control registers (MODE, VECBASE, CYCOUNT, INFO, SYSCALLNO, FCSR, MKMR, CAUSE) used for system configuration and control. |
| **[address]** | Memory access at the specified address. |
| **#immediate** | Immediate value used as an operand in instructions. |
| **opcode operand1, operand2** | General instruction format where `opcode` is the operation code and `operand1`, `operand2` are the instruction operands. |
| **;** | Statement terminator in assembly language syntax. |
| **// comment** | Single-line comment in assembly language syntax. |
| **/* comment */** | Multi-line comment in assembly language syntax. |
| **< >** | Angle brackets used to denote optional elements in syntax diagrams. |
| **[ ]** | Square brackets used to denote memory access in assembly language syntax. |
| **0xNN** | Hexadecimal notation for numeric values. |
| **Flag-setting forms** | Flag-setting instructions are separate mnemonics (e.g., `ADC.F`) with their own opcodes. See “Flag-Setting Instructions.” The base forms do not accept a `.F` suffix. |

1.1.    Global Semantic Conventions

- Parity flag (P): When updated, P reflects even parity of the least significant 8 bits of the computed value (P=1 for even parity). Higher bits are ignored for parity.
- Sign flag (S): When updated, S reflects bit 31 of the computed 32-bit value (1 means negative in two’s complement).
- Carry flag (C): For addition-family instructions, C reflects the carry-out of the unsigned addition (1 means a carry out of bit 31 occurred). For subtraction and compare, C reflects a borrow (1 means a borrow occurred).
- IP-relative immediates: All relative immediates (e.g., branch REL8) are signed values in bytes and are added to the address of the next instruction (after the complete current instruction including its operands).
- Register width: All general-purpose registers are 32-bit. Untyped moves preserve bit patterns unchanged.
- Flag preservation: Unless an instruction is documented to update STAT (e.g., CMP/CMPI32, FCMP/FCMPI, or the flag-setting forms in this spec), STAT is left unchanged.

Aliases (compatibility): In earlier drafts and some examples, the following synonyms may appear. Treat them as exact aliases of the new canonical names:
- Rn ≡ Bn, PC ≡ IP, SP ≡ STK, BP ≡ FR, FLAGS ≡ STAT, CTRLn ≡ CTLn.

1.    INSTRUCTION SET

### ADC rD, rS
Add with Carry: Adds the value in source register rS and the carry flag to the destination register rD. Updates STAT.

#### Syntax
```
ADC rD, rS
```

#### Operation
```
rD = rD + rS + (STAT.C ? 1 : 0)
// Does not modify STAT. For a flag-setting form, see ADC.F.
```

#### Usage
```
ADC B1, B2       // B1 = B1 + B2 + Carry
// For a flag-setting add-with-carry, use ADC.F (see Flag-Setting Instructions)
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |` <br>
Opcode:  `00000000b`

#### Summary
16 bits total: 8 bits opcode + 8 bits operands

### ADCI32 rD, #IMM32
Add with Carry Immediate 32-bit: Adds a 32-bit immediate value and the carry flag to the destination register rD. Updates STAT.

#### Syntax
```
ADCI32 rD, #IMM32
```

#### Operation
```
rD = rD + IMM32 + (STAT.C ? 1 : 0)
// Does not modify STAT. For a flag-setting form, see ADCI32.F.
```

#### Usage
```
ADCI32 B1, #0x00000010       // B1 = B1 + 0x00000010 + Carry
// For a flag-setting form, use ADCI32.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM32(32) |` <br>
Opcode:  `00000001b`

#### Summary
48 bits total: 8 bits opcode + 4 bits rD + 4 bits unused + 32 bits immediate

### ADCI16 rD, #IMM16
Add with Carry Immediate 16-bit: Adds a 16-bit immediate value and the carry flag to the destination register rD. Updates STAT.

#### Syntax
```
ADCI16 rD, #IMM16
```

#### Operation
```
rD = rD + IMM16 + (STAT.C ? 1 : 0)
// Does not modify STAT. For a flag-setting form, see ADCI16.F.
```

#### Usage
```
ADCI16 B1, #0x0010       // B1 = B1 + 0x0010 + Carry
// For a flag-setting form, use ADCI16.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM16(16) |` <br>
Opcode:  `00000010b`

#### Summary
32 bits total: 8 bits opcode + 4 bits rD + 4 bits unused + 16 bits immediate

### ADCI8 rD, #IMM8
Add with Carry Immediate 8-bit: Adds an 8-bit immediate value and the carry flag
to the destination register rD. Updates STAT.

#### Syntax
```
ADCI8 rD, #IMM8
```

#### Operation
```
rD = rD + IMM8 + (STAT.C ? 1 : 0)
// Does not modify STAT. For a flag-setting form, see ADCI8.F.
```

#### Usage
```
ADCI8 B1, #0x10       // B1 = B1 + 0x10 + Carry
// For a flag-setting form, use ADCI8.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM8(8) |` <br>
Opcode:  `00000011b`

#### Summary
24 bits total: 8 bits opcode + 4 bits rD + 4 bits unused + 8 bits immediate

### ADD rD, rS
Add: Adds the value in source register rS to the destination register rD. Updates STAT.

#### Syntax
```
ADD rD, rS
```

#### Operation
```
rD = rD + rS
// Does not modify STAT. For a flag-setting form, see ADD.F.
```

#### Usage
```
ADD B1, B2       // B1 = B1 + B2
// For a flag-setting add, use ADD.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |` <br>
Opcode:  `00000100b`

#### Summary
16 bits total: 8 bits opcode + 8 bits operands

### ADDI32 rD, #IMM32
Add Immediate 32-bit: Adds a 32-bit immediate value to the destination register rD. Updates STAT.

#### Syntax
```
ADDI32 rD, #IMM32
```

#### Operation
```
rD = rD + IMM32
// Does not modify STAT. For a flag-setting form, see ADDI32.F.
```

#### Usage
```
ADDI32 B1, #0x00000010       // B1 = B1 + 0x00000010
// For a flag-setting form, use ADDI32.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM32(32) |` <br>
Opcode:  `00000101b`

#### Summary
48 bits total: 8 bits opcode + 4 bits rD + 4 bits unused + 32 bits immediate

### ADDI16 rD, #IMM16
Add Immediate 16-bit: Adds a 16-bit immediate value to the destination register rD. Updates STAT.

#### Syntax
```
ADDI16 rD, #IMM16
```

#### Operation
```
rD = rD + IMM16
// Does not modify STAT. For a flag-setting form, see ADDI16.F.
```

#### Usage
```
ADDI16 B1, #0x0010       // B1 = B1 + 0x0010
// For a flag-setting form, use ADDI16.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM16(16) |` <br>
Opcode:  `00000110b`

#### Summary
32 bits total: 8 bits opcode + 4 bits rD + 4 bits unused + 16 bits immediate

### ADDI8 rD, #IMM8
Add Immediate 8-bit: Adds an 8-bit immediate value to the destination register rD. Updates STAT.

#### Syntax
```
ADDI8 rD, #IMM8
```

#### Operation
```
rD = rD + IMM8
// Does not modify STAT. For a flag-setting form, see ADDI8.F.
```

#### Usage
```
ADDI8 B1, #0x10       // B1 = B1 + 0x10
// For a flag-setting form, use ADDI8.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM8(8) |` <br>
Opcode:  `00000111b`

#### Summary
24 bits total: 8 bits opcode + 4 bits rD + 4 bits unused + 8 bits immediate

### AND rD, rS
Bitwise AND: Performs a bitwise AND operation between the value in source register rS and the destination register rD. Updates STAT.

#### Syntax
```
AND rD, rS
```

#### Operation
```
rD = rD & rS
// Does not modify STAT. For a flag-setting form, see AND.F.
```

#### Usage
```
AND B1, B2       // B1 = B1 & B2
// For a flag-setting AND, use AND.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |` <br>
Opcode:  `00001000b`

#### Summary
16 bits total: 8 bits opcode + 8 bits operands

### ANDI32 rD, #IMM32
Bitwise AND Immediate 32-bit: Performs a bitwise AND operation between a 32-bit immediate value and the destination register rD. Updates STAT.

#### Syntax
```
ANDI32 rD, #IMM32
```

#### Operation
```
rD = rD & IMM32
// Does not modify STAT. For a flag-setting form, see ANDI32.F.
```

#### Usage
```
ANDI32 B1, #0xFFFFFFFF       // B1 = B1 & 0xFFFFFFFF
// For a flag-setting form, use ANDI32.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM32(32) |` <br>
Opcode:  `00001001b`

#### Summary
48 bits total: 8 bits opcode + 4 bits rD + 4 bits unused + 32 bits immediate

### ANDI16 rD, #IMM16
Bitwise AND Immediate 16-bit: Performs a bitwise AND operation between a 16-bit immediate value and the destination register rD. Updates STAT.

#### Syntax
```
ANDI16 rD, #IMM16
```

#### Operation
```
rD = rD & IMM16
// Does not modify STAT. For a flag-setting form, see ANDI16.F.
```

#### Usage
```
ANDI16 B1, #0xFFFF       // B1 = B1 & 0xFFFF
// For a flag-setting form, use ANDI16.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM16(16) |` <br>
Opcode:  `00001010b`

#### Summary
32 bits total: 8 bits opcode + 4 bits rD + 4 bits unused + 16 bits immediate

### ANDI8 rD, #IMM8
Bitwise AND Immediate 8-bit: Performs a bitwise AND operation between an 8-bit immediate value and the destination register rD. Updates STAT.

#### Syntax
```
ANDI8 rD, #IMM8
```

#### Operation
```
rD = rD & IMM8
// Does not modify STAT. For a flag-setting form, see ANDI8.F.
```

#### Usage
```
ANDI8 B1, #0xFF       // B1 = B1 & 0xFF
// For a flag-setting form, use ANDI8.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM8(8) |` <br>
Opcode:  `00001011b`

#### Summary
24 bits total: 8 bits opcode + 4 bits rD + 4 bits unused + 8 bits immediate


## Floating-Point Extension (B/FP32) and CPU-System Instructions

This section extends B/ with single-precision floating-point operations and explicit CPU/system control instructions. It preserves the base decoding model (8-bit opcode + variable-length operands). Flag-setting forms are defined as separate instructions (see “Flag-Setting Instructions”).

Additional notation used below:
- #f32(x): A typed immediate literal encoded as IEEE-754 single-precision (32-bit). Assembler must encode the exact bit pattern into IMM32.
- CTL[n]: Denotes control register index n (0–7) when referenced in system instructions.
- FP rounding and exceptions: Unless stated otherwise, floating-point rounding and exception behavior are controlled via CTL5 (FCSR). See note at the end of this section.

### FADD rD, rS
Floating Add (single-precision): Interprets rD and rS as IEEE-754 float32, computes rD = rD + rS. Does not modify STAT; see FADD.F for the flag-setting form.

#### Syntax
```
FADD rD, rS
```

#### Operation
```
// Interpret bit-patterns of rD and rS as float32
fD = f32(rD)
fS = f32(rS)
fR = fD + fS  // rounding mode from CTL5
rD = bits(fR)
// Does not modify STAT. For a flag-setting form, see FADD.F in Flag-Setting Instructions.
```

#### Usage
```
FADD B1, B2       // B1 = float(B1) + float(B2)
// For a flag-setting FP add, use FADD.F
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `10000000b`

#### Summary
16 bits total: 8 bits opcode + 8 bits operands

### FSUB rD, rS
Floating Subtract: rD = float(rD) - float(rS).

#### Syntax
```
FSUB rD, rS
```

#### Operation
```
fR = f32(rD) - f32(rS)
rD = bits(fR)
// Does not modify STAT. For a flag-setting form, see FSUB.F.
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `10000001b`

#### Summary
16 bits total: 8 bits opcode + 8 bits operands

### FMUL rD, rS
Floating Multiply: rD = float(rD) * float(rS).

#### Syntax
```
FMUL rD, rS
```

#### Operation
```
fR = f32(rD) * f32(rS)
rD = bits(fR)
// Does not modify STAT. For a flag-setting form, see FMUL.F.
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `10000010b`

#### Summary
16 bits total

### FDIV rD, rS
Floating Divide: rD = float(rD) / float(rS).

#### Syntax
```
FDIV rD, rS
```

#### Operation
```
fR = f32(rD) / f32(rS)
rD = bits(fR)
// Does not modify STAT. For a flag-setting form, see FDIV.F.
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `10000011b`

#### Summary
16 bits total

### FFMA rD, rA, rB
Fused Multiply-Add: rD = bits(f32(rD) + f32(rA) * f32(rB)) using a single rounding step.

#### Syntax
```
FFMA rD, rA, rB
```

#### Operation
```
fR = fma(f32(rA), f32(rB), f32(rD)) // one rounding step
rD = bits(fR)
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rA(4) | rB(4) | UNUSED(4) |`  
Opcode:  `10000100b`

#### Summary
24 bits total: 8 bits opcode + 12 bits regs + 4 bits unused (byte-aligned)

### FINV rD, rS
Floating Reciprocal: rD = bits(1.0 / f32(rS)).

#### Syntax
```
FINV rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `10000101b`

#### Summary
16 bits total
// For a flag-setting form, see FINV.F.

### FSQRT rD, rS
Floating Square Root: rD = bits(sqrt(f32(rS))).

#### Syntax
```
FSQRT rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `10000110b`

#### Summary
16 bits total
// For a flag-setting form, see FSQRT.F.

### FCMP rA, rB
Floating Compare: Sets STAT based on comparison of f32(rA) vs f32(rB).

#### Syntax
```
FCMP rA, rB
```

#### Operation
```
// Unordered comparisons (NaN) set O=1 and Z=0
if (isnan(f32(rA)) || isnan(f32(rB))) {
    STAT.O = 1; STAT.Z = 0; STAT.S = 0; STAT.C = 0; STAT.P = 0;
} else {
    STAT.O = 0; STAT.Z = (f32(rA) == f32(rB)) ? 1 : 0;
    STAT.S = (f32(rA) <  f32(rB)) ? 1 : 0; // "less-than" encodes in S
    STAT.C = 0; STAT.P = 0;
}
```

#### Bits
Format:  `| OPCODE(8) | rA(4) | rB(4) |`  
Opcode:  `10000111b`

#### Summary
16 bits total

#### Notes
- Unordered comparisons (NaN involvement) set O=1 and Z=0 as specified; S and other flags are cleared.
- Derived relations for branching after FCMP (with O=0):
    - Equal: Z==1
    - Less-than: S==1
    - Greater-than: (Z==0 && S==0)
    - Less-or-equal: (Z==1 || S==1)
    - Greater-or-equal: (Z==1 || (Z==0 && S==0))

### FCMPI rA, #IMM32
Floating Compare Immediate: Compare f32(rA) with immediate float32.

#### Syntax
```
FCMPI rA, #f32(imm)
```

#### Operation
```
fImm = f32_from_bits(IMM32)
// Then same flag rules as FCMP
```

#### Bits
Format:  `| OPCODE(8) | rA(4) | UNUSED(4) | IMM32(32) |`  
Opcode:  `10001000b`

#### Summary
48 bits total

### ITOF rD, rS
Integer-to-Float Conversion: rD = bits(convert_s32_to_f32((int32)rS)). Rounding follows CTL5.

#### Syntax
```
ITOF rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `10001001b`

#### Summary
16 bits total

### FTOI rD, rS
Float-to-Integer Conversion: rD = (int32)round(f32(rS)) per CTL5 rounding mode.

#### Syntax
```
FTOI rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `10001010b`

#### Summary
16 bits total

### FMOV rD, rS
Floating Move (bit-preserving): rD = rS (intended for moving float payloads between registers).

#### Syntax
```
FMOV rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `10001011b`

#### Summary
16 bits total

### FMOVI32 rD, #IMM32
Floating Move Immediate: rD = IMM32 (raw IEEE-754 bits). Use #f32(x) for readability.

#### Syntax
```
FMOVI32 rD, #f32(imm)
FMOVI32 rD, #0xXXXXXXXX
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM32(32) |`  
Opcode:  `10001100b`

#### Summary
48 bits total

---

## CPU/System Instructions

These instructions directly interact with CPU control registers and execution state. They are designed to expose low-level control in a compact, readable way.

### SYSRD rD, CTL[n]
System Read Control Register: Read CTL[n] into rD (n in 0–7).

#### Syntax
```
SYSRD rD, CTL[n]
```

#### Operation
```
rD = CTL[n]
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | CTLIDX(3) | UNUSED(1) |`  
Opcode:  `11100000b`

#### Summary
16 bits total

#### Privilege and Effects
- Allowed in kernel mode for all CTL[n].
- In user mode: SYSRD CTL2 (cycle counter) and CTL5 (FCSR) are allowed; others raise a protection fault.
- Does not modify STAT.

### SYSWR CTL[n], rS
System Write Control Register: Write rS to CTL[n] (n in 0–7).

#### Syntax
```
SYSWR CTL[n], rS
```

#### Operation
```
CTL[n] = rS
// Note: Writing CTL3 triggers the info-query side effects defined earlier.
```

#### Bits
Format:  `| OPCODE(8) | rS(4) | CTLIDX(3) | UNUSED(1) |`  
Opcode:  `11100001b`

#### Summary
16 bits total

#### Privilege and Effects
- Allowed in kernel mode for all CTL[n].
- In user mode: SYSWR CTL5 (FCSR) is allowed; all other CTL writes raise a protection fault. CTL4 (SYSCALLNO) is kernel-only.
- Writing CTL3 triggers the documented side effects and may clobber B0–B3.
- Does not modify STAT.

### SYSCALL
System Call: Triggers a system call using CTL4 low 8 bits as the syscall number. Arguments are conventionally passed in B0–B5; return in B0. Traps to kernel mode.

#### Syntax
```
SYSCALL
```

#### Operation
```
// Implementation-defined transfer to kernel handler
trap_to_kernel(CTL4 & 0xFF)
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `11100010b`

#### Summary
8 bits total

#### Execution Semantics
- Traps to kernel mode. Context saving is identical to interrupts (see Interrupt Handling order).
- Handler selection and entry address are OS-defined using CTL1 (vector base). The syscall number is `CTL4 & 0xFF` (kernel-preconfigured; user mode cannot modify CTL4).
- Calling convention: Arguments in B0–B5; return value in B0. Other registers are callee-saved by OS policy.
- Does not modify STAT beyond those changed by the handler upon return.

### WFI
Wait For Interrupt: Enter low-power state until an interrupt is taken (STAT.IE must be set to receive interrupts).

#### Syntax
```
WFI
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `11100011b`

#### Summary
8 bits total

#### Execution Semantics
- Privileged (kernel mode only). In user mode, raises a protection fault.
- Enters low-power state until any interrupt is pending; returns regardless of STAT.IE. Delivery to software still follows STAT.IE policy.

### IRET
Interrupt Return: Pops saved context and resumes interrupted execution. Restores registers in the order specified in Interrupt Handling.

#### Syntax
```
IRET
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `11100100b`

#### Summary
8 bits total

#### Execution Semantics
- Privileged (kernel mode only). Restores context from the stack exactly as specified in Interrupt Handling.

### ENI
Enable Interrupts: Sets STAT.IE = 1.

#### Syntax
```
ENI
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `11100101b`

#### Summary
8 bits total

#### Execution Semantics
- Privileged (kernel mode only). Sets STAT.IE = 1.

### DIS
Disable Interrupts: Clears STAT.IE = 0.

#### Syntax
```
DIS
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `11100110b`

#### Summary
8 bits total

#### Execution Semantics
- Privileged (kernel mode only). Sets STAT.IE = 0.

### FENCE
Full Memory Barrier: Ensures all prior loads/stores complete before any subsequent memory operations.

#### Syntax
```
FENCE
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `11100111b`

#### Summary
8 bits total

#### Execution Semantics
- Provides a full, sequentially-consistent barrier for memory and I/O operations.

### RDTSC rD
Read Timestamp Counter: Reads cycle counter (CTL2) into rD.

#### Syntax
```
RDTSC rD
```

#### Operation
```
rD = CTL2
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) |`  
Opcode:  `11101000b`

#### Summary
16 bits total

#### Execution Semantics
- Reads CTL2 (cycle counter) as a 32-bit value with wrap-around on overflow. Allowed in all modes.

### RDBAD rD
Read Bad Address: Reads the most recent faulting address associated with CTL7.CAUSE into rD.

#### Syntax
```
RDBAD rD
```

#### Operation
```
rD = BADADDR // read-only architectural register tracking last synchronous fault address
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) |`  
Opcode:  `11010011b`

#### Summary
16 bits total

#### Execution Semantics
- Returns an implementation-maintained read-only register that records the faulting virtual address for the most recent synchronous address-related fault (e.g., alignment, page fault, protection fault). Cleared or overwritten on the next such fault. Accessible in all modes.

### EXT rD, rS, #SUBOP8 (Extension / Multi-core)
Multi-purpose extension opcode. The low 8-bit SUBOP selects a sub-operation; rD/rS semantics depend on the sub-op. The defined sub-ops are reserved for multi-core coordination.

#### Syntax
```
EXT rD, rS, #subop8
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) | SUBOP(8) |`  
Opcode:  `11001011b`

#### Summary
24 bits total

#### Defined SUBOPs (Multi-core, shared memory)
- **0x00 — EXT.MTCC (Multi-Task Core Count)**: rD := number of architecturally visible cores. All modes. STAT unchanged.
- **0x01 — EXT.MTID (Current Core ID)**: rD := zero-based core ID of the executing core. All modes. STAT unchanged.
- **0x02 — EXT.MTSTS rD, rS (Core Status)**: rS supplies the target core ID. rD receives a status bitmap: bit0=online, bit1=bootstrap/original core, bit2=pending IPI. Protection fault (CAUSE=0x01) if the core ID is out of range.
- **0x03 — EXT.MTSETIP (Set remote entry/context address)**: Privileged (kernel mode only). rS supplies target core ID, rD supplies a 32-bit address that the implementation treats as the remote core’s next entry/context address. If the target core is parked, it becomes eligible to run and is marked online. Protection fault if in user mode or the core ID is invalid. Software should FENCE.I after writing code that the remote core will execute.
- **0x04 — EXT.MTI (Send inter-processor interrupt)**: Privileged. rS supplies target core ID, rD’s low 8 bits supply the vector ID. If the target equals the current core, delivery is immediate and identical to SICF using that vector. If the target is another core, the interrupt is latched as pending for that core; delivery timing is implementation-defined but must be observed before the next instruction boundary once the target core executes. Protection fault if in user mode or the core ID is invalid.

Undefined SUBOP values raise an illegal-instruction fault.

### CPUID #SEL8
CPU Information Query: Writes selector to CTL3 and reads back CPU info as defined (B0–B3 and/or B0–B1 depending on selector).

#### Syntax
```
CPUID #0xNN      // 8-bit selector written to CTL3
```

#### Operation
```
CTL3 = SEL8
// The CPU then populates B0–B3 or B0–B1 per CTL3 rules described earlier.
```

#### Bits
Format:  `| OPCODE(8) | SEL8(8) |`  
Opcode:  `11101001b`

#### Summary
16 bits total

#### Execution Semantics
- Accessible in all modes. Clobbers B0–B3 with results according to CTL3 selector rules. Does not modify STAT.

---

### Note: CTL5 (FCSR) — Floating-Point Control/Status

CTL5 (FCSR) is defined as the FP control/status register:
- Bits 0–2: RND (Rounding mode)
    - 000: Round to Nearest (even)
    - 001: Round toward Zero
    - 010: Round toward +Infinity
    - 011: Round toward −Infinity
    - 100–111: Reserved
- Bits 3–7: Exception Flags (sticky)
    - Bit 3: Invalid Operation
    - Bit 4: Division by Zero
    - Bit 5: Overflow
    - Bit 6: Underflow
    - Bit 7: Inexact Result
- Bits 8–31: Reserved for future use

Exception flags are sticky and use write-1-to-clear semantics via `SYSWR`.
Rounding mode is configured by writing the desired value into bits 0–2 of CTL5.

Software can configure rounding via `SYSWR CTL[5], rS` and read/clear exception flags via `SYSRD`/`SYSWR` sequences.

---

## Conditional Branches (flag-based)

Short forward/backward branches based on STAT. Offsets are signed 8-bit values relative to the next instruction.

### BZ #REL8
Branch if Zero: Branches if STAT.Z == 1.

#### Syntax
```
BZ #rel8
```

#### Operation
```
if (STAT.Z == 1) IP = IP + sign_extend(REL8);
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `00100000b`

#### Summary
16 bits total

### BNZ #REL8
Branch if Not Zero: Branches if STAT.Z == 0.

#### Syntax
```
BNZ #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `00100001b`

#### Summary
16 bits total

### BS #REL8
Branch if Sign: Branches if STAT.S == 1.

#### Syntax
```
BS #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `00100010b`

#### Summary
16 bits total

### BNS #REL8
Branch if Not Sign: Branches if STAT.S == 0.

#### Syntax
```
BNS #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `00100011b`

#### Summary
16 bits total

### BO #REL8
Branch if Overflow: Branches if STAT.O == 1.

#### Syntax
```
BO #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `00100100b`

#### Summary
16 bits total

### BNO #REL8
Branch if Not Overflow: Branches if STAT.O == 0.

#### Syntax
```
BNO #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `00100101b`

#### Summary
16 bits total

### J #REL8
Unconditional short jump: IP += sign_extend(REL8).

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `00100111b`

#### Summary
16 bits total

### J32 #REL32
Unconditional jump with 32-bit relative displacement.

#### Bits
Format:  `| OPCODE(8) | REL32(32) |`  
Opcode:  `00101000b`

#### Summary
40 bits total

### CALL #REL32
Call with 32-bit relative displacement: pushes return IP to the stack and jumps to IP + REL32.

#### Operation
```
ret = IP_of_next
STK = STK - 4
MEM32[STK] = ret
IP = IP + sign_extend(REL32)
```

#### Bits
Format:  `| OPCODE(8) | REL32(32) |`  
Opcode:  `00110000b`

#### Summary
40 bits total

---

### CALLR rS
Call Register: Pushes return IP to the stack and jumps to the address in rS.

#### Operation
```
ret = IP_of_next
STK = STK - 4
MEM32[STK] = ret
IP = rS
```

#### Bits
Format:  `| OPCODE(8) | rS(4) | UNUSED(4) |`  
Opcode:  `00101010b`

#### Summary
16 bits total

---

### JR rS
Jump Register: Jumps to the address in rS without modifying the stack.

#### Operation
```
IP = rS
```

#### Bits
Format:  `| OPCODE(8) | rS(4) | UNUSED(4) |`  
Opcode:  `00101011b`

#### Summary
16 bits total

---

### JA #ABS32
Jump Absolute: Jumps to the absolute address specified by the 32-bit immediate.

#### Operation
```
IP = ABS32
```

#### Bits
Format:  `| OPCODE(8) | ABS32(32) |`  
Opcode:  `00101100b`

#### Summary
40 bits total

---

### CALLA #ABS32
Call Absolute: Pushes return IP to the stack and jumps to the absolute address specified by the 32-bit immediate.

#### Operation
```
ret = IP_of_next
STK = STK - 4
MEM32[STK] = ret
IP = ABS32
```

#### Bits
Format:  `| OPCODE(8) | ABS32(32) |`  
Opcode:  `00101101b`

#### Summary
40 bits total

---

## Calls and Returns

### RET
Return from function: Pops return address from stack into IP and resumes execution at that address.

#### Syntax
```
RET
```

#### Operation
```
ip_ret = MEM32[STK]
STK = STK + 4
IP = ip_ret
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `00100110b`

#### Summary
8 bits total

#### Notes
- Does not modify STAT. Allowed in user and kernel modes.
- For interrupt/exception returns, use IRET.

### ENTERI16 #FRAME16
Function prologue (static frame size): Pushes FR, establishes FR as the frame base, and allocates a static-sized local stack frame.

#### Syntax
```
ENTERI16 #frame_bytes
```

#### Operation
```
// Push old FR
STK = STK - 4
MEM32[STK] = FR
// Establish new frame base
FR = STK
// Allocate locals (frame_bytes is a 16-bit unsigned immediate)
STK = STK - zero_extend(FRAME16)
```

#### Bits
Format:  `| OPCODE(8) | FRAME16(16) |`  
Opcode:  `00101111b`

#### Summary
24 bits total

#### Notes
- FRAME16 may be 0 for frameless or register-only leaf setups that still want a canonical prologue.
- Alignment: Compilers should round up frame_bytes to preserve 4-byte STK alignment at call boundaries.

### LEAVE
Function epilogue: Discards the local stack frame and restores the caller’s FR.

#### Syntax
```
LEAVE
```

#### Operation
```
STK = FR
FR = MEM32[STK]
STK = STK + 4
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `00101110b`

#### Summary
8 bits total

#### Notes
- Typically followed by RET.

---

## Special Register Stack Operations

Dedicated helpers to spill/restore selected special registers without using temporaries. These follow the same privilege constraints as SYSRD/SYSWR where applicable.

### PSTAT
Push STAT onto the stack.

#### Syntax
```
PSTAT
```

#### Operation
```
STK = STK - 4
MEM32[STK] = zero_extend(STAT & 0xFF) // stores defined STAT bits; upper bits zeroed
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `00101001b`

#### Summary
8 bits total; allowed in user and kernel modes.

### POPSTAT
Pop 32-bit word from stack and write STAT (kernel-only).

#### Syntax
```
POPSTAT
```

#### Operation
```
tmp = MEM32[STK]
STK = STK + 4
// Only architecturally defined STAT bits are written; reserved bits ignored
STAT = (tmp & 0xFF)
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `00110001b`

#### Summary
8 bits total; privileged (kernel mode only). In user mode, raises a protection fault.

### PCTL CTL[n]
Push CTL[n] onto the stack.

#### Syntax
```
PCTL CTL[n]
```

#### Operation
```
STK = STK - 4
MEM32[STK] = CTL[n]
```

#### Bits
Format:  `| OPCODE(8) | CTLIDX(3) | UNUSED(5) |`  
Opcode:  `00110010b`

#### Summary
16 bits total. Privilege: Same as SYSRD — in user mode, only CTL2 and CTL5 are permitted; others fault.

### POPCTL CTL[n]
Pop 32-bit word from stack into CTL[n].

#### Syntax
```
POPCTL CTL[n]
```

#### Operation
```
tmp = MEM32[STK]
STK = STK + 4
CTL[n] = tmp
```

#### Bits
Format:  `| OPCODE(8) | CTLIDX(3) | UNUSED(5) |`  
Opcode:  `00110011b`

#### Summary
16 bits total. Privilege: Same as SYSWR — in user mode, only CTL5 is permitted; others fault.

### PFR / POPFR
Push/pop the frame register (FR) without establishing a full frame.

#### Syntax
```
PFR
POPFR
```

#### Operation
```
// PFR
STK = STK - 4
MEM32[STK] = FR
// POPFR
FR = MEM32[STK]
STK = STK + 4
```

#### Bits
Format:  `| OPCODE(8) |`  
PFR Opcode:   `00110100b`  
POPFR Opcode: `00110101b`

#### Summary
Each 8 bits total; allowed in user and kernel modes.

---

## Direct Special-Register Moves (SP/BP)

These instructions move between general-purpose registers and the special registers STK (stack pointer, "SP") and FR (frame/base pointer, "BP"). They enable directly setting/reading SP and BP from code without going through stack helpers.

Semantics
- Do not modify STAT.
- Allowed in user and kernel modes.
- Software is responsible for preserving the ABI’s 4-byte stack alignment at call boundaries when writing STK.

### RDSTK rD
Read STK into rD.

Bits
Format: `| OPCODE(8) | rD(4) | UNUSED(4) |`  Opcode: `00111000b` (0x38)  Size: 16 bits  Priv: U/K

### WRSTK rS
Write STK from rS.

Bits
Format: `| OPCODE(8) | rS(4) | UNUSED(4) |`  Opcode: `00111001b` (0x39)  Size: 16 bits  Priv: U/K

### RDFR rD
Read FR into rD.

Bits
Format: `| OPCODE(8) | rD(4) | UNUSED(4) |`  Opcode: `00111010b` (0x3A)  Size: 16 bits  Priv: U/K

### WRFR rS
Write FR from rS.

Bits
Format: `| OPCODE(8) | rS(4) | UNUSED(4) |`  Opcode: `00111011b` (0x3B)  Size: 16 bits  Priv: U/K

Notes
- Typical usage to set both BP and SP at function entry:
    - WRFR Bx        ; FR = Bx (establish base)
    - WRSTK By       ; STK = By (install stack)
- For code that needs to sample pointers:
    - RDSTK Bx       ; Bx = STK
    - RDFR By        ; By = FR

---

## Integer Compare

### CMP rA, rB
Compare Registers: Sets STAT as if computing rA - rB (result not stored).

#### Syntax
```
CMP rA, rB
```

#### Operation
```
tmp = rA - rB
STAT.Z = (tmp == 0);
STAT.S = (tmp < 0);
STAT.C = (borrow occurred) ? 1 : 0; // for unsigned
STAT.O = (signed overflow occurred) ? 1 : 0;
STAT.P = (parity of tmp) ? 1 : 0;
```

#### Bits
Format:  `| OPCODE(8) | rA(4) | rB(4) |`  
Opcode:  `00001100b`

#### Summary
16 bits total

### CMPI32 rA, #IMM32
Compare Immediate 32-bit: Sets STAT from rA - IMM32.

#### Syntax
```
CMPI32 rA, #IMM32
```

#### Bits
Format:  `| OPCODE(8) | rA(4) | UNUSED(4) | IMM32(32) |`  
Opcode:  `00001101b`

#### Summary
48 bits total

---

## Bit Manipulation and Count

### ROR rD, rS
Rotate Right: rD = rD >>> (rS & 31) | rD << (32 - (rS & 31)).

#### Syntax
```
ROR rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `01000000b`

#### Summary
16 bits total

#### Notes
- Rotation count uses only the low 5 bits of rS. A count of 0 leaves rD unchanged.

### RORI8 rD, #IMM8
Rotate Right by Immediate: rD = rotr(rD, IMM8 & 31).

#### Syntax
```
RORI8 rD, #IMM8
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM8(8) |`  
Opcode:  `01000001b`

#### Summary
24 bits total

#### Notes
- Only the low 5 bits of IMM8 are used for the rotation count. A count of 0 leaves rD unchanged.

### POPC rD, rS
Population Count: rD = number_of_set_bits(rS).

#### Syntax
```
POPC rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `01000010b`

#### Summary
16 bits total

#### Notes
- Result is in the range 0–32 inclusive.

### CLZ rD, rS
Count Leading Zeros: rD = leading_zero_count(rS).

#### Syntax
```
CLZ rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `01000011b`

#### Summary
16 bits total

#### Notes
- For rS == 0, CLZ returns 32.

### CTZ rD, rS
Count Trailing Zeros: rD = trailing_zero_count(rS).

#### Syntax
```
CTZ rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `01000100b`

#### Summary
16 bits total

#### Notes
- For rS == 0, CTZ returns 32.

### BTST rA, #BITIDX
Bit Test: Sets STAT.Z = 0 if bit BITIDX in rA is 1, else STAT.Z = 1. Other STAT bits unchanged.

#### Syntax
```
BTST rA, #bit
```

#### Operation
```
STAT.Z = ((rA >> (BITIDX & 31)) & 1) ? 0 : 1;
```

#### Bits
Format:  `| OPCODE(8) | rA(4) | UNUSED(4) | BITIDX(8) |`  
Opcode:  `01000101b`

#### Summary
24 bits total

#### Notes
- BITIDX uses only the low 5 bits.

### BEXT rD, rS, rM
Bit Extract: Extract bits from rS where mask rM has 1s, packed densely into LSBs of rD.

#### Syntax
```
BEXT rD, rS, rM
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) | rM(4) | UNUSED(4) |`  
Opcode:  `01110000b`

#### Summary
24 bits total

### BDEP rD, rS, rM
Bit Deposit: Deposit low bits of rS into rD positions where rM has 1s; other bits of rD preserved.

#### Syntax
```
BDEP rD, rS, rM
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) | rM(4) | UNUSED(4) |`  
Opcode:  `01110001b`

#### Summary
24 bits total

---

## Core Integer ALU (Additions)

The following instructions complete the base integer arithmetic and logical set.

### SUB rD, rS
Subtract: rD = rD - rS.

#### Syntax
```
SUB rD, rS
```

#### Operation
```
tmp = rD - rS
rD = tmp
// Does not modify STAT. For a flag-setting form, see SUB.F.
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `00001110b`

#### Summary
16 bits total

### OR rD, rS
Bitwise OR.

#### Syntax
```
OR rD, rS
```

#### Operation
```
rD = rD | rS
// Does not modify STAT. For a flag-setting form, see OR.F.
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `00001111b`

#### Summary
16 bits total

### XOR rD, rS
Bitwise XOR.

#### Syntax
```
XOR rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `00010000b`

#### Summary
16 bits total

### NOT rD, rS
Bitwise NOT into rD: rD = ~rS.

#### Syntax
```
NOT rD, rS
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) |`  
Opcode:  `00010001b`

#### Summary
16 bits total

### Immediate forms for OR/XOR/SUB

The following immediate forms mirror the ANDI encodings and complete the immediate ALU set for codegen symmetry. Flag-setting forms are defined separately; the base immediate forms do not update STAT.

#### ORI32 rD, #IMM32 / ORI16 rD, #IMM16 / ORI8 rD, #IMM8
Bitwise OR with immediate. Bits formats mirror ANDI forms.

Bits
- ORI32: `| OPCODE(8) | rD(4) | UNUSED(4) | IMM32(32) |`  Opcode: `00010111b` (0x17)  Size: 48
- ORI16: `| OPCODE(8) | rD(4) | UNUSED(4) | IMM16(16) |`  Opcode: `00011000b` (0x18)  Size: 32
- ORI8:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM8(8)  |`  Opcode: `00011001b` (0x19)  Size: 24

#### XORI32 rD, #IMM32 / XORI16 rD, #IMM16 / XORI8 rD, #IMM8
Bitwise XOR with immediate. Bits formats mirror ANDI forms.

Bits
- XORI32: `| OPCODE(8) | rD(4) | UNUSED(4) | IMM32(32) |`  Opcode: `00011010b` (0x1A)  Size: 48
- XORI16: `| OPCODE(8) | rD(4) | UNUSED(4) | IMM16(16) |`  Opcode: `00011011b` (0x1B)  Size: 32
- XORI8:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM8(8)  |`  Opcode: `00011100b` (0x1C)  Size: 24

#### SUBI32 rD, #IMM32 / SUBI16 rD, #IMM16 / SUBI8 rD, #IMM8
Integer subtract with immediate: rD = rD - IMM. Base forms do not modify STAT. See SUBI*.F in Flag-Setting Instructions for flag behavior.

Bits
- SUBI32: `| OPCODE(8) | rD(4) | UNUSED(4) | IMM32(32) |`  Opcode: `00011101b` (0x1D)  Size: 48
- SUBI16: `| OPCODE(8) | rD(4) | UNUSED(4) | IMM16(16) |`  Opcode: `00011110b` (0x1E)  Size: 32
- SUBI8:  `| OPCODE(8) | rD(4) | UNUSED(4) | IMM8(8)  |`  Opcode: `00011111b` (0x1F)  Size: 24

### SHL rD, rS  /  SHLI8 rD, #IMM8
Logical shift left; immediate form uses IMM8 & 31.

#### Bits
Formats:  
`| OPCODE(8) | rD(4) | rS(4) |` (SHL, opcode `01000110b`)  
`| OPCODE(8) | rD(4) | UNUSED(4) | IMM8(8) |` (SHLI8, opcode `01000111b`)

### LSR rD, rS  /  LSRI8 rD, #IMM8
Logical shift right; immediate form uses IMM8 & 31.

#### Bits
Formats:  
`| OPCODE(8) | rD(4) | rS(4) |` (LSR, opcode `01001000b`)  
`| OPCODE(8) | rD(4) | UNUSED(4) | IMM8(8) |` (LSRI8, opcode `01001001b`)

### ASR rD, rS  /  ASRI8 rD, #IMM8
Arithmetic shift right (sign-propagating); immediate form uses IMM8 & 31.

#### Bits
Formats:  
`| OPCODE(8) | rD(4) | rS(4) |` (ASR, opcode `01001010b`)  
`| OPCODE(8) | rD(4) | UNUSED(4) | IMM8(8) |` (ASRI8, opcode `01001011b`)

### MUL rD, rA, rB
Integer multiply (low 32 bits): rD = (rA * rB) & 0xFFFFFFFF.

#### Bits
Format:  `| OPCODE(8) | rD(4) | rA(4) | rB(4) | UNUSED(4) |`  
Opcode:  `00010010b`

### DIV rD, rA, rB  / UDIV rD, rA, rB
Signed/Unsigned integer divide; rD = quotient. Division by zero sets CTL7.CAUSE=0x05 and raises a fault.

#### Bits
Format:  `| OPCODE(8) | rD(4) | rA(4) | rB(4) | UNUSED(4) |`  
Opcodes:  `00010011b` (DIV), `00010100b` (UDIV)

---

## Flag-Setting Instructions

This section defines the flag-setting variants as separate instructions with unique opcodes. They mirror the operations of their base counterparts but update STAT as described.

Flag semantics (recap)
- For integer add/adc/addi: set Z,S,C (carry-out), O (signed overflow), P (even parity of low 8 bits of result).
- For integer logical AND/OR/XOR (and immediates): set Z,S and clear C,O; P reflects even parity of the result’s low byte.
- For integer subtraction/subi: set Z,S,C (borrow=1), O, P.
- For FP arithmetic: Z=1 for ±0.0 results, S reflects signbit, O indicates overflow/invalid, C,P set to 0.

### Integer register-register

- ADC.F rD, rS   — opcode 0x50 — rD = rD + rS + (C?1:0); updates Z,S,C,O,P
- ADD.F rD, rS   — opcode 0x54 — rD = rD + rS; updates Z,S,C,O,P
- AND.F rD, rS   — opcode 0x58 — rD = rD & rS; updates Z,S; C=0,O=0; P=parity
- OR.F  rD, rS   — opcode 0x5D — rD = rD | rS; updates Z,S; C=0,O=0; P=parity
- XOR.F rD, rS   — opcode 0x5E — rD = rD ^ rS; updates Z,S; C=0,O=0; P=parity
- SUB.F rD, rS   — opcode 0x5C — rD = rD - rS; updates Z,S,C(borrow),O,P

### Integer immediates

- ADCI32.F rD, #imm32 — opcode 0x51 — rD += imm32 + C; updates Z,S,C,O,P
- ADCI16.F rD, #imm16 — opcode 0x52 — rD += imm16 + C; updates Z,S,C,O,P
- ADCI8.F  rD, #imm8  — opcode 0x53 — rD += imm8 + C;  updates Z,S,C,O,P
- ADDI32.F rD, #imm32 — opcode 0x55 — rD += imm32;     updates Z,S,C,O,P
- ADDI16.F rD, #imm16 — opcode 0x56 — rD += imm16;     updates Z,S,C,O,P
- ADDI8.F  rD, #imm8  — opcode 0x57 — rD += imm8;      updates Z,S,C,O,P
- ANDI32.F rD, #imm32 — opcode 0x59 — rD &= imm32;     updates Z,S; C=0,O=0; P=parity
- ANDI16.F rD, #imm16 — opcode 0x5A — rD &= imm16;     updates Z,S; C=0,O=0; P=parity
- ANDI8.F  rD, #imm8  — opcode 0x5B — rD &= imm8;      updates Z,S; C=0,O=0; P=parity
- SUBI32.F rD, #imm32 — opcode 0x5F — rD -= imm32;     updates Z,S,C(borrow),O,P
- SUBI16.F rD, #imm16 — opcode 0x60 — rD -= imm16;     updates Z,S,C(borrow),O,P
- SUBI8.F  rD, #imm8  — opcode 0x61 — rD -= imm8;      updates Z,S,C(borrow),O,P

### Floating-point

- FADD.F  rD, rS — opcode 0x90 — rD = bits(f32(rD)+f32(rS)); updates Z,S,O; C=0,P=0
- FSUB.F  rD, rS — opcode 0x91 — rD = bits(f32(rD)-f32(rS)); updates Z,S,O; C=0,P=0
- FMUL.F  rD, rS — opcode 0x92 — rD = bits(f32(rD)*f32(rS)); updates Z,S,O; C=0,P=0
- FDIV.F  rD, rS — opcode 0x93 — rD = bits(f32(rD)/f32(rS)); updates Z,S,O; C=0,P=0
- FINV.F  rD, rS — opcode 0x94 — rD = bits(1.0/f32(rS));     updates Z,S,O; C=0,P=0
- FSQRT.F rD, rS — opcode 0x95 — rD = bits(sqrt(f32(rS)));   updates Z,S,O; C=0,P=0

Encodings
- All encodings mirror the base forms’ operand formats (register-register vs immediate), with only the opcode byte changed as listed above.

Notes
- CMP/CMPI32 and FCMP/FCMPI already update STAT by definition; there are no separate “.F” forms for these.
- Logical and arithmetic immediate forms follow the same flag behavior as their register-register counterparts when using the .F mnemonics listed here.


## Keyed Memory Access (uses CTL6 key mask)

Keyed memory instructions enforce that the current key (CTL6 mask) authorizes access to the target region. On violation, a precise protection fault is raised.

Key check semantics (normative):
- The implementation associates an 8-bit key ID (0–255) with each 4 KiB page.
- Access is permitted iff ((1u << (page_key_id & 7)) & (CTL6 & 0xFF)) != 0. Implementations may define page_key_id>7 to alias into 0–7.
- If the access is not permitted, the instruction raises a precise protection fault before any memory side effects.

### LD.K rD, [rA + #OFF8]
Keyed Load: Loads 32-bit word from memory with key check.

#### Syntax
```
LD.K rD, [rA + #off8]
```

#### Operation
```
addr = rA + sign_extend(OFF8)
check_key(addr) // faults if unauthorized
rD = MEM32[addr]
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rA(4) | OFF8(8) |`  
Opcode:  `10100000b`

#### Summary
24 bits total

### ST.K [rA + #OFF8], rS
Keyed Store: Stores 32-bit word to memory with key check.

#### Syntax
```
ST.K [rA + #off8], rS
```

#### Operation
```
addr = rA + sign_extend(OFF8)
check_key(addr) // faults if unauthorized
MEM32[addr] = rS
```

#### Bits
Format:  `| OPCODE(8) | rS(4) | rA(4) | OFF8(8) |`  
Opcode:  `10100001b`

#### Summary
24 bits total

### LDRGN rD, [rA], #LEN8
Region-Check Load (non-faulting probe): If the region [rA, rA+LEN8) is readable, loads rD = MEM32[rA]; otherwise leaves rD unchanged and sets STAT.O = 1.

#### Syntax
```
LDRGN rD, [rA], #len8
```

#### Operation
```
if (region_readable(rA, LEN8)) {
    rD = MEM32[rA];
    STAT.O = 0;
} else {
    // rD unchanged
    STAT.O = 1;
}
```

#### Notes
- region_readable applies current privilege and key-mask rules and ensures the entire [rA, rA+LEN8) lies within readable pages. No faults are generated by LDRGN itself.
- On success, LDRGN has the same memory effects as a normal 32-bit load at address rA.

#### Bits
Format:  `| OPCODE(8) | rD(4) | rA(4) | LEN8(8) |`  
Opcode:  `10100100b`

#### Summary
24 bits total

---

## Plain Memory Access

Standard (unkeyed) load/store operations. Alignment rules follow the Memory Subsystem policy.

### LD rD, [rA + #OFF8]
Load 32-bit word from memory.

#### Bits
Format:  `| OPCODE(8) | rD(4) | rA(4) | OFF8(8) |`  
Opcode:  `10100010b`

#### Summary
24 bits total

### ST [rA + #OFF8], rS
Store 32-bit word to memory.

#### Bits
Format:  `| OPCODE(8) | rS(4) | rA(4) | OFF8(8) |`  
Opcode:  `10100011b`

#### Summary
24 bits total

### LDBU rD, [rA + #OFF8]  /  LDBS rD, [rA + #OFF8]
Load 8-bit byte, zero/sign-extended into rD.

#### Bits
Format:  `| OPCODE(8) | rD(4) | rA(4) | OFF8(8) |`  
Opcodes: `10100101b` (LDBU), `10100110b` (LDBS)

### LDHU rD, [rA + #OFF8]  /  LDHS rD, [rA + #OFF8]
Load 16-bit halfword, zero/sign-extended into rD. Requires addr%2==0.

#### Bits
Format:  `| OPCODE(8) | rD(4) | rA(4) | OFF8(8) |`  
Opcodes: `10100111b` (LDHU), `10101000b` (LDHS)

### STB [rA + #OFF8], rS  /  STH [rA + #OFF8], rS
Store 8-bit/16-bit value from rS to memory (low bits used). Halfword store requires addr%2==0.

#### Bits
Format:  `| OPCODE(8) | rS(4) | rA(4) | OFF8(8) |`  
Opcodes: `10101001b` (STB), `10101010b` (STH)

### LDRIP rD, #REL32
IP-relative 32-bit load: rD = MEM32[IP + REL32]. Useful for PIC data references.

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | REL32(32) |`  
Opcode:  `10101011b`

#### Summary
48 bits total

## Memory Model and Atomics

This section defines the architectural memory model, granular fences, and load-linked/store-conditional primitives for building atomic operations. It complements the existing full FENCE and plain loads/stores.

### Memory model (normative)

- Atomicity
    - Naturally aligned 32-bit loads and stores are atomic with respect to other naturally aligned 32-bit loads/stores to the same address. Byte and halfword accesses are atomic at their natural sizes. Unaligned word/halfword accesses fault (see Alignment policy).
- Ordering
    - B/ is a weakly ordered architecture: the CPU may reorder independent loads and stores around each other and around arithmetic instructions, subject to data/addr dependencies and fences.
    - FENCE provides a full sequentially-consistent barrier for both memory and I/O (SC fence).
    - For finer control, the following granular fences are provided:
        - FENCE.R: all prior loads complete before any subsequent loads or stores.
        - FENCE.W: all prior stores become visible before any subsequent loads or stores.
        - FENCE.I: synchronize the instruction stream with prior data writes (self-modifying code). Ensures subsequent instruction fetches observe prior code modifications.
- Data-race-free (DRF) guarantee
    - Programs that are data-race-free using LL/SC (or higher-level atomics built from them) and appropriate fences observe sequentially consistent behavior.

### Granular fences

#### FENCE.R
Read barrier: Orders prior loads before subsequent memory operations.

Bits: `| OPCODE(8) |`  Opcode: `11101010b` (0xEA)  Size: 8 bits  Priv: U/K

#### FENCE.W
Write barrier: Orders prior stores before subsequent memory operations.

Bits: `| OPCODE(8) |`  Opcode: `11101011b` (0xEB)  Size: 8 bits  Priv: U/K

#### FENCE.I
Instruction-stream sync: Ensures subsequent instruction fetch observes prior data writes to code.

Bits: `| OPCODE(8) |`  Opcode: `11101100b` (0xEC)  Size: 8 bits  Priv: U/K

Notes
- FENCE is equivalent to FENCE.R followed by FENCE.W and provides SC ordering. Compilers may emit FENCE exclusively when SC semantics are required.

### Load-Linked / Store-Conditional

LL/SC provide a compact primitive to build higher-level atomic operations (e.g., CAS, atomic add) in software.

#### LL rD, [rA + #OFF8]
Load-linked 32-bit word and establish a reservation on the addressed location.

Syntax
```
LL rD, [rA + #off8]
```

Semantics
```
addr = rA + sign_extend(OFF8)
rD = MEM32[addr]
reservation = addr   // implementation holds a reservation for addr
// Acquire semantics: subsequent memory ops cannot be reordered before this LL
```

Bits
Format: `| OPCODE(8) | rD(4) | rA(4) | OFF8(8) |`  Opcode: `10101100b` (0xAC)

#### SC [rA + #OFF8], rS
Store-conditional 32-bit word: attempts to store rS to the address if the reservation is still valid.

Syntax
```
SC [rA + #off8], rS
```

Semantics
```
addr = rA + sign_extend(OFF8)
if (reservation == addr && no conflicting write observed) {
        MEM32[addr] = rS
        STAT.Z = 1   // success
} else {
        // no store performed
        STAT.Z = 0   // failure
}
clear reservation
// Release semantics on success: prior memory ops become visible before the store
```

Bits
Format: `| OPCODE(8) | rS(4) | rA(4) | OFF8(8) |`  Opcode: `10101101b` (0xAD)

Notes
- LL/SC pairs must target the same effective address and execute on the same core without intervening context switches to maximize success probability. Any exception, interrupt, or conflicting write may clear the reservation.
- For sequentially consistent atomics, bracket LL/SC loops with FENCE (or use FENCE.R and FENCE.W as appropriate).

## Transactional Microblocks

Best-effort atomic regions for small critical sections. On abort, effects are rolled back and STAT.O is set to 1.

Semantics (normative):
- A transaction begins at TXBEGIN and ends at the matching TXEND. Nesting of transactions is not supported; executing TXBEGIN within a transaction causes an immediate abort of the inner attempt (STAT.O=1) and no state change.
- The architectural memory effects of all stores within a transaction become visible atomically at TXEND on a successful commit. On abort, none of the transactional stores become architecturally visible.
- Loads within a transaction may be performed; their returned values are not rolled back.
- Aborts may occur due to capacity limits, conflicts, exceptions/faults, interrupts, or explicit TXABORT. On abort, STAT.O is set to 1; otherwise TXEND sets STAT.O=0.
- Register contents are not rolled back by the CPU on abort. Software should structure transactions to be restartable using STAT.O and optional software bookkeeping.
- Memory ordering: A successful TXEND acts as a full fence for the transaction’s memory operations.

### TXBEGIN
Begin transactional region.

#### Syntax
```
TXBEGIN
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `10110000b`

#### Summary
8 bits total

### TXEND
End transactional region. Attempts to commit.

#### Syntax
```
TXEND
```

#### Bits
Format:  `| OPCODE(8) |`  
Opcode:  `10110001b`

#### Summary
8 bits total

### TXABORT #CODE8
Abort transactional region with a software-specified code.

#### Syntax
```
TXABORT #code8
```

#### Bits
Format:  `| OPCODE(8) | CODE8(8) |`  
Opcode:  `10110010b`

#### Summary
16 bits total

---

## Fiber Primitives

Lightweight cooperative threading primitives.

### SPAWN rD, rFn, rArg
Spawn a fiber starting at address in rFn with argument rArg. Returns a fiber handle in rD.

#### Syntax
```
SPAWN rD, rFn, rArg
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rFn(4) | rArg(4) | UNUSED(4) |`  
Opcode:  `11000000b`

#### Summary
24 bits total

#### Execution Semantics
- Creates a new fiber with entry IP = rFn and initial argument in B0 = rArg. The new fiber’s STK is implementation-defined (per runtime/OS ABI).
- On success: rD receives a nonzero handle; STAT.Z = 0. On failure: rD = 0; STAT.Z = 1.
- Does not modify other STAT bits.

### YIELD rQ
Yield to run queue rQ (implementation-defined scheduling domain). If rQ is unused, yields to default queue.

#### Syntax
```
YIELD rQ
```

#### Bits
Format:  `| OPCODE(8) | rQ(4) | UNUSED(4) |`  
Opcode:  `11000001b`

#### Summary
16 bits total

#### Execution Semantics
- Cooperatively yields the CPU. rQ (low 8 bits) selects a run queue/domain; if no such queue exists, yields to the default queue.
- Returns to caller when rescheduled. Does not modify STAT.

---

## Observability

### TLOG rA, #TAG8
Trace Log: If tracing enabled (CTL0 bit implementation-defined), emit (IP, rA, TAG8) to a ring buffer; otherwise no-op.

#### Syntax
```
TLOG rA, #tag8
```

#### Bits
Format:  `| OPCODE(8) | rA(4) | UNUSED(4) | TAG8(8) |`  
Opcode:  `11010000b`

#### Summary
24 bits total

### PERF rD, #SEL8
Performance Counter Read: Read a selected performance metric into rD.

#### Syntax
```
PERF rD, #sel8
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) | SEL8(8) |`  
Opcode:  `11010001b`

#### Summary
24 bits total

### PEEKPTE rD, [rA]
Peek Page Table Entry: Read a summary of the PTE for the address in rA into rD without changing translation state.

#### Syntax
```
PEEKPTE rD, [rA]
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rA(4) |`  
Opcode:  `11010010b`

#### Summary
16 bits total

### CICPY rD, rS, rC
Circuit Copy: Copies `rC` bytes from source address in `rS` to destination address in `rD`.

#### Syntax
```
CICPY rD, rS, rC
```

#### Operation
```
dst = rD
src = rS
count = rC
for i in [0, count):
    MEM8[dst + i] = MEM8[src + i]
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | rS(4) | rC(4) | UNUSED(4) |`
Opcode:  `11010110b`

#### Summary
24 bits total

#### Execution Semantics
- Alignment is not required; unaligned copy is architecturally valid.
- Intended as a hardware-assisted bulk copy primitive; implementations may use optimized internal paths.
- Overlap behavior is implementation-defined. Software should not rely on memmove-style overlap guarantees unless the platform ABI specifies them.
- If any byte access faults, the instruction raises the corresponding precise fault (protection/page/alignment where applicable to platform policy).

---

### BGE #rel8
Branch if Greater or Equal (signed): if (STAT.S == STAT.O) IP += sign_extend(REL8).

#### Syntax
```
BGE #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `11101101b`

#### Summary
16 bits total

### BLT #rel8
Branch if Less Than (signed): if (STAT.S != STAT.O) IP += sign_extend(REL8).

#### Syntax
```
BLT #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `11101110b`

#### Summary
16 bits total

### BLE #rel8
Branch if Less or Equal (signed): if (STAT.Z == 1 || STAT.S != STAT.O) IP += sign_extend(REL8).

#### Syntax
```
BLE #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `11110000b`

#### Summary
16 bits total

### BGT #rel8
Branch if Greater Than (signed): if (STAT.Z == 0 && STAT.S == STAT.O) IP += sign_extend(REL8).

#### Syntax
```
BGT #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `11101111b`

#### Summary
16 bits total

### BGEU #rel8
Branch if Greater or Equal (unsigned): if (STAT.C == 0) IP += sign_extend(REL8).

#### Syntax
```
BGEU #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `11110001b`

#### Summary
16 bits total

### BLTU #rel8
Branch if Less Than (unsigned): if (STAT.C == 1) IP += sign_extend(REL8).

#### Syntax
```
BLTU #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `11110010b`

#### Summary
16 bits total

### BGTU #rel8
Branch if Greater Than (unsigned): if (STAT.C == 0 && STAT.Z == 0) IP += sign_extend(REL8).

#### Syntax
```
BGTU #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `11110011b`

#### Summary
16 bits total

### BLEU #rel8
Branch if Less or Equal (unsigned): if (STAT.C == 1 || STAT.Z == 1) IP += sign_extend(REL8).

#### Syntax
```
BLEU #rel8
```

#### Bits
Format:  `| OPCODE(8) | REL8(8) |`  
Opcode:  `11110100b`

#### Summary
16 bits total

### PUSHR rS
Push register onto stack: STK -= 4; MEM32[STK] = rS.

#### Syntax
```
PUSHR rS
```

#### Bits
Format:  `| OPCODE(8) | rS(4) | UNUSED(4) |`  
Opcode:  `11110101b`

#### Summary
16 bits total

### POPR rD
Pop register from stack: rD = MEM32[STK]; STK += 4.

#### Syntax
```
POPR rD
```

#### Bits
Format:  `| OPCODE(8) | rD(4) | UNUSED(4) |`  
Opcode:  `11110110b`

#### Summary
16 bits total

### PUSHI32 #IMM32
Push immediate 32-bit value onto stack: STK -= 4; MEM32[STK] = IMM32.

#### Syntax
```
PUSHI32 #imm32
```

#### Bits
Format:  `| OPCODE(8) | IMM32(32) |`  
Opcode:  `11110111b`

#### Summary
40 bits total

### RRPIN #loop32, INSTRUCTION
Rapid repeat and instruction: Repeats the following instruction #loop32 times with minimal overhead.
<br>
Valid Instructions: All non-control-flow instructions except LL/SC, TXBEGIN/TXEND, and FENCE variants.
List:
- Arithmetic: ADC, ADCI*, ADD, ADDI*, AND, ANDI*, CMP, CMPI*, SUB, SUBI*, OR, ORI*, XOR, XORI*
- Shifts: SHL, SHLI8, LSR, LSRI8, ASR, ASRI8
- Multiply/Divide: MUL, DIV, UDIV
- Move/Load/Store: MOV, MOVI32, LD, ST, LDBU, LDBS, LDHU, LDHS, STB, STH, LDRIP

#### Syntax
```
RRPIN #loop32, INSTRUCTION
```

#### Bits
Format:  `| OPCODE(8) | loop32(32) | INSTRUCTION(x) |`  
Opcode:  `11111000b`

#### Summary
Variable size total

### RRPIN.R #loop32, rD, INSTRUCTION
Rapid repeat with incrementation: Repeats INSTRUCTION #loop32 times, incrementing rD each iteration.

#### Syntax
```
RRPIN.R #loop32, rD, INSTRUCTION
```

#### Bits
Format:  `| OPCODE(8) | loop32(32) | rD(4) | UNUSED(4) | INSTRUCTION(x) |`  
Opcode:  `11111001b`

#### Summary
Variable size total

#### Note
The register is incremented after each iteration of INSTRUCTION. The INSTRUCTION can use rD as a destination or source operand, including for memory accesses.

### RRPIN.V rT, INSTRUCTION
Rapid repeat with value in rT: Repeats INSTRUCTION #rT times, where rT holds the loop count and will be decremented to zero.

#### Syntax
```
RRPIN.V rT, INSTRUCTION
```

#### Bits
Format:  `| OPCODE(8) | rT(4) | UNUSED(4) | INSTRUCTION(x) |`  
Opcode:  `11111010b`

#### Summary
Variable size total

### RRPIN.VR rT, rD, INSTRUCTION
Rapid repeat with value in rT and incrementation: Repeats INSTRUCTION #rT times, incrementing rD each iteration and decrementing rT to zero.

#### Syntax
```
RRPIN.VR rT, rD, INSTRUCTION
```

#### Bits
Format:  `| OPCODE(8) | rT(4) | rD(4) | UNUSED(4) | INSTRUCTION(x) |`
Opcode:  `11111011b`

#### Summary
Variable size total

### SICF #IMM8
Software interrupt CPU flow: triggers a software interrupt with immediate 8-bit code.

#### Syntax
```
SICF #imm8
```

#### Bits
Format:  `| OPCODE(8) | IMM8(8) |`
Opcode:  `11111100b`

#### Summary
16 bits total

#### Execution Semantics
- Allowed in user and kernel modes.
- Sets `CTL7.CAUSE = imm8` and pushes the return IP onto the stack.
- Transfers control to the handler stored at `[CTL1 + 4*imm8]`; the handler returns with `IRET`.
- Does not modify `STAT` flags other than the optional IE masking performed by software.

### OUTPRT.S #BYTESIZE2, #PORT16, rS
Output to I/O port from register: writes the low BYTESIZE2 bytes of rS to the specified I/O port (little-endian byte order).

#### Syntax
```
OUTPRT.S #bytesize2, #port16, rS
```

#### Bits
Format:  `| OPCODE(8) | PORT16(16) | BYTESIZE2(2) | rA(4) | UNUSED(2) |`  
Opcode:  `11111101b`

#### Summary
32 bits total

### INPRT.S #BYTESIZE2, rD, #PORT16
Input from I/O port to register: reads BYTESIZE2 bytes from the specified I/O port into rD (little-endian, zero-extended).

#### Syntax
```
INPRT.S #bytesize2, rD, #port16
```

#### Bits
Format:  `| OPCODE(8) | PORT16(16) | BYTESIZE2(2) | rD(4) | UNUSED(2) |`  
Opcode:  `11111110b`

#### Summary
32 bits total

---

## Instruction Table (Canonical)

A consolidated, canonical instruction table with flag effects and privilege notes. Offsets are IP-relative; sizes include opcode and operands.

| Opcode | Mnemonic | Operands | Size (bits) | Flags | Priv | Summary |
|-------:|----------|----------|------------:|-------|------|---------|
| 0x00 | ADC | rD, rS | 16 | — | U/K | Add with carry (uses STAT.C); no flag update |
| 0x01 | ADCI32 | rD, #imm32 | 48 | — | U/K | Add imm32 with carry; no flag update |
| 0x02 | ADCI16 | rD, #imm16 | 32 | — | U/K | Add imm16 with carry; no flag update |
| 0x03 | ADCI8 | rD, #imm8 | 24 | — | U/K | Add imm8 with carry; no flag update |
| 0x04 | ADD | rD, rS | 16 | — | U/K | Integer add; no flag update |
| 0x05 | ADDI32 | rD, #imm32 | 48 | — | U/K | Integer add immediate 32-bit; no flag update |
| 0x06 | ADDI16 | rD, #imm16 | 32 | — | U/K | Integer add immediate 16-bit; no flag update |
| 0x07 | ADDI8 | rD, #imm8 | 24 | — | U/K | Integer add immediate 8-bit; no flag update |
| 0x08 | AND | rD, rS | 16 | — | U/K | Bitwise AND; no flag update |
| 0x09 | ANDI32 | rD, #imm32 | 48 | — | U/K | Bitwise AND with imm32; no flag update |
| 0x0A | ANDI16 | rD, #imm16 | 32 | — | U/K | Bitwise AND with imm16; no flag update |
| 0x0B | ANDI8 | rD, #imm8 | 24 | — | U/K | Bitwise AND with imm8; no flag update |
| 0x0C | CMP | rA, rB | 16 | Z,S,C,O,P | U/K | Compare rA - rB; updates STAT |
| 0x0D | CMPI32 | rA, #imm32 | 48 | Z,S,C,O,P | U/K | Compare rA - imm32; updates STAT |
| 0x0E | SUB | rD, rS | 16 | — | U/K | Integer subtract; no flag update |
| 0x0F | OR | rD, rS | 16 | — | U/K | Bitwise OR; no flag update |
| 0x10 | XOR | rD, rS | 16 | — | U/K | Bitwise XOR; no flag update |
| 0x11 | NOT | rD, rS | 16 | — | U/K | Bitwise NOT into rD |
| 0x12 | MUL | rD, rA, rB | 24 | — | U/K | Integer multiply; low 32 bits to rD |
| 0x13 | DIV | rD, rA, rB | 24 | — | U/K | Signed divide; fault on div-by-zero |
| 0x14 | UDIV | rD, rA, rB | 24 | — | U/K | Unsigned divide; fault on div-by-zero |
| 0x15 | MOV | rD, rS | 16 | — | U/K | Bit-preserving move (alias of FMOV) |
| 0x16 | MOVI32 | rD, #imm32 | 48 | — | U/K | Move 32-bit immediate into rD |
| 0x17 | ORI32 | rD, #imm32 | 48 | — | U/K | Bitwise OR with imm32; no flag update |
| 0x18 | ORI16 | rD, #imm16 | 32 | — | U/K | Bitwise OR with imm16; no flag update |
| 0x19 | ORI8  | rD, #imm8  | 24 | — | U/K | Bitwise OR with imm8; no flag update |
| 0x1A | XORI32 | rD, #imm32 | 48 | — | U/K | Bitwise XOR with imm32; no flag update |
| 0x1B | XORI16 | rD, #imm16 | 32 | — | U/K | Bitwise XOR with imm16; no flag update |
| 0x1C | XORI8  | rD, #imm8  | 24 | — | U/K | Bitwise XOR with imm8; no flag update |
| 0x1D | SUBI32 | rD, #imm32 | 48 | — | U/K | Integer subtract with imm32; no flag update |
| 0x1E | SUBI16 | rD, #imm16 | 32 | — | U/K | Integer subtract with imm16; no flag update |
| 0x1F | SUBI8  | rD, #imm8  | 24 | — | U/K | Integer subtract with imm8; no flag update |
| 0x17 | ORI32 | rD, #imm32 | 48 | opt .F | U/K | Bitwise OR with imm32 |
| 0x18 | ORI16 | rD, #imm16 | 32 | opt .F | U/K | Bitwise OR with imm16 |
| 0x19 | ORI8  | rD, #imm8  | 24 | opt .F | U/K | Bitwise OR with imm8 |
| 0x1A | XORI32 | rD, #imm32 | 48 | opt .F | U/K | Bitwise XOR with imm32 |
| 0x1B | XORI16 | rD, #imm16 | 32 | opt .F | U/K | Bitwise XOR with imm16 |
| 0x1C | XORI8  | rD, #imm8  | 24 | opt .F | U/K | Bitwise XOR with imm8 |
| 0x1D | SUBI32 | rD, #imm32 | 48 | opt .F | U/K | Integer subtract with imm32 |
| 0x1E | SUBI16 | rD, #imm16 | 32 | opt .F | U/K | Integer subtract with imm16 |
| 0x1F | SUBI8  | rD, #imm8  | 24 | opt .F | U/K | Integer subtract with imm8 |
| 0x20 | BZ | #rel8 | 16 | — | U/K | Branch if Z==1; IP += sign-extended rel8 |
| 0x21 | BNZ | #rel8 | 16 | — | U/K | Branch if Z==0 |
| 0x22 | BS | #rel8 | 16 | — | U/K | Branch if S==1 |
| 0x23 | BNS | #rel8 | 16 | — | U/K | Branch if S==0 |
| 0x24 | BO | #rel8 | 16 | — | U/K | Branch if O==1 |
| 0x25 | BNO | #rel8 | 16 | — | U/K | Branch if O==0 |
| 0x26 | RET | — | 8 | — | U/K | Pop IP from [STK]; STK += 4 |
| 0x27 | J | #rel8 | 16 | — | U/K | Unconditional short jump |
| 0x28 | J32 | #rel32 | 40 | — | U/K | Unconditional long jump |
| 0x29 | CALL | #rel32 | 40 | — | U/K | Push RA; IP += rel32 |
| 0x2A | CALLR | rA | 16 | — | U/K | Push RA; IP = rA |
| 0x2B | JR | rA | 16 | — | U/K | IP = rA |
| 0x2C | JA | #abs32 | 40 | — | U/K | Jump to absolute address |
| 0x2D | CALLA | #abs32 | 40 | — | U/K | Push RA; Jump to absolute address |
| 0x2E | LEAVE | — | 8 | — | U/K | Epilogue: STK=FR; FR=[STK]; STK+=4 |
| 0x2F | ENTERI16 | #frame16 | 24 | — | U/K | Prologue: push FR; FR=STK; STK-=frame16 |
| 0x30 | PSTAT | — | 8 | — | U/K | Push STAT (lower 8 bits) onto stack |
| 0x31 | POPSTAT | — | 8 | — | K | Pop into STAT (kernel only) |
| 0x32 | PCTL | CTL[n] | 16 | — | U: CTL2,5; K: all | Push CTL[n] onto stack |
| 0x33 | POPCTL | CTL[n] | 16 | — | U: CTL5; K: all | Pop into CTL[n] |
| 0x34 | PFR | — | 8 | — | U/K | Push FR onto stack |
| 0x35 | POPFR | — | 8 | — | U/K | Pop FR from stack |
| 0x40 | ROR | rD, rS | 16 | — | U/K | Rotate right by (rS & 31) |
| 0x41 | RORI8 | rD, #imm8 | 24 | — | U/K | Rotate right by (imm8 & 31) |
| 0x42 | POPC | rD, rS | 16 | — | U/K | Population count of rS (0–32) |
| 0x43 | CLZ | rD, rS | 16 | — | U/K | Count leading zeros (0→32) |
| 0x44 | CTZ | rD, rS | 16 | — | U/K | Count trailing zeros (0→32) |
| 0x45 | BTST | rA, #bit | 24 | Z only | U/K | Test bit; Z=0 if bit set, else Z=1 |
| 0x36 | NOP | — | 8 | — | U/K | No operation |
| 0x37 | BRK | — | 8 | — | U/K | Software breakpoint; sets CAUSE=0x06 and traps |
| 0x50 | ADC.F | rD, rS | 16 | Z,S,C,O,P | U/K | Add with carry; updates STAT |
| 0x51 | ADCI32.F | rD, #imm32 | 48 | Z,S,C,O,P | U/K | Add imm32 with carry; updates STAT |
| 0x52 | ADCI16.F | rD, #imm16 | 32 | Z,S,C,O,P | U/K | Add imm16 with carry; updates STAT |
| 0x53 | ADCI8.F | rD, #imm8 | 24 | Z,S,C,O,P | U/K | Add imm8 with carry; updates STAT |
| 0x54 | ADD.F | rD, rS | 16 | Z,S,C,O,P | U/K | Integer add; updates STAT |
| 0x55 | ADDI32.F | rD, #imm32 | 48 | Z,S,C,O,P | U/K | Integer add imm32; updates STAT |
| 0x56 | ADDI16.F | rD, #imm16 | 32 | Z,S,C,O,P | U/K | Integer add imm16; updates STAT |
| 0x57 | ADDI8.F | rD, #imm8 | 24 | Z,S,C,O,P | U/K | Integer add imm8; updates STAT |
| 0x58 | AND.F | rD, rS | 16 | Z,S (C,O=0,P) | U/K | Bitwise AND; updates Z,S; C,O=0; P=parity |
| 0x59 | ANDI32.F | rD, #imm32 | 48 | Z,S (C,O=0,P) | U/K | Bitwise AND imm32; updates flags |
| 0x5A | ANDI16.F | rD, #imm16 | 32 | Z,S (C,O=0,P) | U/K | Bitwise AND imm16; updates flags |
| 0x5B | ANDI8.F | rD, #imm8 | 24 | Z,S (C,O=0,P) | U/K | Bitwise AND imm8; updates flags |
| 0x5C | SUB.F | rD, rS | 16 | Z,S,C,O,P | U/K | Integer subtract; updates STAT |
| 0x5D | OR.F | rD, rS | 16 | Z,S (C,O=0,P) | U/K | Bitwise OR; updates Z,S; C,O=0; P=parity |
| 0x5E | XOR.F | rD, rS | 16 | Z,S (C,O=0,P) | U/K | Bitwise XOR; updates Z,S; C,O=0; P=parity |
| 0x5F | SUBI32.F | rD, #imm32 | 48 | Z,S,C,O,P | U/K | Integer subtract imm32; updates STAT |
| 0x60 | SUBI16.F | rD, #imm16 | 32 | Z,S,C,O,P | U/K | Integer subtract imm16; updates STAT |
| 0x61 | SUBI8.F | rD, #imm8 | 24 | Z,S,C,O,P | U/K | Integer subtract imm8; updates STAT |
| 0x38 | RDSTK | rD | 16 | — | U/K | Read STK (stack pointer) into rD |
| 0x39 | WRSTK | rS | 16 | — | U/K | Write STK (stack pointer) from rS |
| 0x3A | RDFR | rD | 16 | — | U/K | Read FR (frame/base pointer) into rD |
| 0x3B | WRFR | rS | 16 | — | U/K | Write FR (frame/base pointer) from rS |
| 0x46 | SHL | rD, rS | 16 | — | U/K | Logical shift left by (rS & 31) |
| 0x47 | SHLI8 | rD, #imm8 | 24 | — | U/K | Logical shift left by (imm8 & 31) |
| 0x48 | LSR | rD, rS | 16 | — | U/K | Logical shift right by (rS & 31) |
| 0x49 | LSRI8 | rD, #imm8 | 24 | — | U/K | Logical shift right by (imm8 & 31) |
| 0x4A | ASR | rD, rS | 16 | — | U/K | Arithmetic shift right by (rS & 31) |
| 0x4B | ASRI8 | rD, #imm8 | 24 | — | U/K | Arithmetic shift right by (imm8 & 31) |
| 0x70 | BEXT | rD, rS, rM | 24 | — | U/K | Bit extract by mask rM into rD |
| 0x71 | BDEP | rD, rS, rM | 24 | — | U/K | Bit deposit of rS into rD masked by rM |
| 0x80 | FADD | rD, rS | 16 | — | U/K | FP add (f32); no flag update |
| 0x81 | FSUB | rD, rS | 16 | — | U/K | FP subtract (f32); no flag update |
| 0x82 | FMUL | rD, rS | 16 | — | U/K | FP multiply (f32); no flag update |
| 0x83 | FDIV | rD, rS | 16 | — | U/K | FP divide (f32); no flag update |
| 0x84 | FFMA | rD, rA, rB | 24 | — | U/K | Fused multiply-add (one rounding) |
| 0x85 | FINV | rD, rS | 16 | — | U/K | FP reciprocal 1.0/f32(rS); no flag update |
| 0x86 | FSQRT | rD, rS | 16 | — | U/K | FP square root; no flag update |
| 0x87 | FCMP | rA, rB | 16 | Z,S,O | U/K | FP compare; NaN→O=1,Z=0 |
| 0x88 | FCMPI | rA, #imm32 | 48 | Z,S,O | U/K | FP compare with immediate |
| 0x89 | ITOF | rD, rS | 16 | — | U/K | Convert int32→f32 (bits to rD) |
| 0x8A | FTOI | rD, rS | 16 | — | U/K | Convert f32→int32 per CTL5 |
| 0x8B | FMOV | rD, rS | 16 | — | U/K | Bit-preserving move |
| 0x8C | FMOVI32 | rD, #imm32 | 48 | — | U/K | Move raw 32-bit immediate (IEEE-754 bits) |
| 0x90 | FADD.F | rD, rS | 16 | Z,S,O | U/K | FP add; updates Z,S,O; C=0,P=0 |
| 0x91 | FSUB.F | rD, rS | 16 | Z,S,O | U/K | FP subtract; updates Z,S,O; C=0,P=0 |
| 0x92 | FMUL.F | rD, rS | 16 | Z,S,O | U/K | FP multiply; updates Z,S,O; C=0,P=0 |
| 0x93 | FDIV.F | rD, rS | 16 | Z,S,O | U/K | FP divide; updates Z,S,O; C=0,P=0 |
| 0x94 | FINV.F | rD, rS | 16 | Z,S,O | U/K | FP reciprocal; updates Z,S,O; C=0,P=0 |
| 0x95 | FSQRT.F | rD, rS | 16 | Z,S,O | U/K | FP sqrt; updates Z,S,O; C=0,P=0 |
| 0xA0 | LD.K | rD, [rA + #off8] | 24 | — | U/K | Keyed load (CTL6 authorization) |
| 0xA1 | ST.K | [rA + #off8], rS | 24 | — | U/K | Keyed store (CTL6 authorization) |
| 0xA2 | LD | rD, [rA + #off8] | 24 | — | U/K | Load 32-bit word |
| 0xA3 | ST | [rA + #off8], rS | 24 | — | U/K | Store 32-bit word |
| 0xA4 | LDRGN | rD, [rA], #len8 | 24 | O on fail | U/K | Non-faulting region-check load |
| 0xA5 | LDBU | rD, [rA + #off8] | 24 | — | U/K | Load byte zero-extended |
| 0xA6 | LDBS | rD, [rA + #off8] | 24 | — | U/K | Load byte sign-extended |
| 0xA7 | LDHU | rD, [rA + #off8] | 24 | — | U/K | Load halfword zero-extended |
| 0xA8 | LDHS | rD, [rA + #off8] | 24 | — | U/K | Load halfword sign-extended |
| 0xA9 | STB | [rA + #off8], rS | 24 | — | U/K | Store byte (low 8 bits) |
| 0xAA | STH | [rA + #off8], rS | 24 | — | U/K | Store halfword (low 16 bits) |
| 0xAB | LDRIP | rD, #rel32 | 48 | — | U/K | Load 32-bit from [IP + rel32] |
| 0xAC | LL | rD, [rA + #off8] | 24 | — | U/K | Load-linked word; acquire semantics |
| 0xAD | SC | [rA + #off8], rS | 24 | Z (success) | U/K | Store-conditional word; release on success |
| 0xB0 | TXBEGIN | — | 8 | — | U/K | Begin transactional region |
| 0xB1 | TXEND | — | 8 | O=0 on commit | U/K | End/commit transactional region |
| 0xB2 | TXABORT | #code8 | 16 | O=1 | U/K | Abort transactional region with code |
| 0xC0 | SPAWN | rD, rFn, rArg | 24 | Z (success/fail) | U/K | Create fiber; rD=handle (0 on failure) |
| 0xC1 | YIELD | rQ | 16 | — | U/K | Yield to run queue rQ (low 8 bits) |
| 0xD0 | TLOG | rA, #tag8 | 24 | — | U/K | Tracepoint emit (IP, rA, tag) if enabled |
| 0xD1 | PERF | rD, #sel8 | 24 | — | U/K | Read performance counter selection into rD |
| 0xD2 | PEEKPTE | rD, [rA] | 16 | — | U/K | Read PTE summary for address rA |
| 0xD3 | RDBAD | rD | 16 | — | U/K | Read last fault address into rD |
| 0xD6 | CICPY | rD, rS, rC | 24 | — | U/K | Hardware-assisted byte copy from rS to rD for rC bytes |
| 0xE0 | SYSRD | rD, CTL[n] | 16 | — | U: CTL2,5; K: all | Read control register n |
| 0xE1 | SYSWR | CTL[n], rS | 16 | — | U: CTL5; K: all | Write control register n |
| 0xE2 | SYSCALL | — | 8 | — | U/K | System call using CTL4 low 8 bits; traps to kernel |
| 0xE3 | WFI | — | 8 | — | K | Wait For Interrupt (kernel only) |
| 0xE4 | IRET | — | 8 | — | K | Return from interrupt (kernel only) |
| 0xE5 | ENI | — | 8 | — | K | Enable interrupts (kernel only) |
| 0xE6 | DIS | — | 8 | — | K | Disable interrupts (kernel only) |
| 0xE7 | FENCE | — | 8 | — | U/K | Full memory and I/O barrier |
| 0xEA | FENCE.R | — | 8 | — | U/K | Read barrier (order prior loads) |
| 0xEB | FENCE.W | — | 8 | — | U/K | Write barrier (order prior stores) |
| 0xEC | FENCE.I | — | 8 | — | U/K | Instruction-stream sync (icache) |
| 0xE8 | RDTSC | rD | 16 | — | U/K | Read cycle counter (CTL2) into rD |
| 0xE9 | CPUID | #sel8 | 16 | — | U/K | CPU info query via CTL3; clobbers B0–B3 |
| 0xED | BGE | #rel8 | 16 | — | U/K | Branch if S==O |
| 0xEE | BLT | #rel8 | 16 | — | U/K | Branch if S!=O |
| 0xEF | BGT | #rel8 | 16 | — | U/K | Branch if Z==0 and S==O |
| 0xF0 | BLE | #rel8 | 16 | — | U/K | Branch if Z==1 or S!=O |
| 0xF1 | BGEU | #rel8 | 16 | — | U/K | Branch if C==0 |
| 0xF2 | BLTU | #rel8 | 16 | — | U/K | Branch if C==1 |
| 0xF3 | BGTU | #rel8 | 16 | — | U/K | Branch if C==0 and Z==0 | |
| 0xF4 | BLEU | #rel8 | 16 | — | U/K | Branch if C==1 or Z==1 |
| 0xF5 | PUSHR | rS | 16 | — | U/K | Push rS onto stack |
| 0xF6 | POPR | rD | 16 | — | U/K | Pop from stack into rD |
| 0xF7 | PUSHI32 | #imm32 | 48 | — | U/K | Push imm32 onto stack |
| 0xF8 | RRPIN | #loop32, INSTRUCTION | 32 + variable  | — | U/K | Repeat an instruction #loop32 times |
| 0xF9 | RRPIN.R | #loop32, rD, INSTRUCTION | 40 + variable | — | U/K | Repeat an instruction #loop32 times whilst incrementing rD |
| 0xFA | RRPIN.VR | rT, rS, INSTRUCTION | 8 + variable | — | U/K | Repeat an instruction rT times whilst incrementing rS |
| 0xFB | RRPIN.V | rS, INSTRUCTION | 8 + variable | — | U/K | Repeat an instruction rS times |
| 0xFC | SICF | #imm8 | 16 | - | U/K | Software interrupt CPU flow |
| 0xFD | OUTPRT.S | #bytesize2, #port16, rS | 32 | — | U/K | Output to I/O port (size 1,2,4 bytes) |
| 0xFE | INPRT.S | #bytesize2, rD, #port16 | 32 | — | U/K | Input from I/O port (size 1,2,4 bytes) |


---

## Application Binary Interface (ABI)

This section defines the canonical B/ user-level ABI for functions, libraries, and compilers. It standardizes how arguments and return values are passed, which registers must be preserved, and how the stack frame is organized. Kernel/interrupt ABIs are noted where relevant.

Scope and assumptions:
- 32-bit little-endian data model. Pointers are 32-bit.
- General-purpose register file is B0–B15; there is no separate FP register file. FP32 values are carried in Bn bitwise.
- Stack is full descending and must be 4-byte aligned at each call boundary.
- STAT is volatile across calls (caller-saved). CTL registers are not part of the user-level ABI contract.

### Register classification

- Caller-saved (volatile): B0–B7, STAT
    - B0–B5: Primary argument/return registers
    - B6–B7: Temporary values
- Callee-saved (non-volatile): B8–B15, FR, STK
    - A callee must preserve B8–B15 if it uses them (save/restore in its frame)
    - FR (frame register) and STK (stack top) must be restored to entry values on return
- Special registers:
    - IP is modified by control flow; call/return mechanisms manage it
    - CTL[n] are outside the user ABI; user code should not assume stability across calls

### Argument passing

- Up to six 32-bit arguments are passed in B0–B5, left-to-right: arg0→B0, arg1→B1, ...
- Additional arguments are passed on the stack, right-to-left, 4-byte aligned.
- FP32 arguments are passed as their raw 32-bit IEEE-754 bit-patterns in Bn or on the stack equivalently.
- 64-bit integer arguments are passed as two 32-bit words (low in the lower-numbered register/slot):
    - In registers when available (e.g., arg uses Bk=low, Bk+1=high)
    - On the stack as two 4-byte words, 8-byte aligned
- Aggregates (struct/array):
    - If size ≤ 4 bytes: passed in a single register or one 4-byte stack slot
    - If 5–8 bytes: passed in two registers (or two 4-byte stack slots)
    - If > 8 bytes: passed by reference (caller provides pointer in the next argument slot)

### Return values

- 32-bit scalar (int32, pointer, f32): returned in B0 (f32 returns use raw IEEE-754 bits in B0)
- 64-bit scalar: returned in B0 (low) and B1 (high)
- Small aggregates:
    - size ≤ 4 bytes: B0
    - 5–8 bytes: B0–B1, low word in B0
- Larger aggregates (> 8 bytes): returned via a caller-allocated return buffer; caller passes its address as a hidden first argument (sret) in B0, shifting user arguments right by one. In this case, the function returns no additional value in B0.

### Stack and frame layout

- Alignment: STK must be 4-byte aligned at each call instruction boundary. Callers adjust STK before a call so that the callee observes a 4-byte aligned incoming STK.
- Growth: Stack grows downward. Callee allocates local storage by decrementing STK.
- Return address placement: The caller places the 32-bit return address (IP of the first instruction after the call transfer) on the stack before control transfers to the callee. On callee entry, STK points to the return address. RET will pop this value into IP and increment STK by 4.

- Prologue (preferred):
    - ENTERI16 #frame_bytes (FRAME16 rounded to maintain 4-byte alignment)
    - Save any callee-saved registers used (B8–B15) below FR
- Prologue (manual equivalent):
    1. STK = STK - 4; MEM32[STK] = FR
    2. FR = STK
    3. Save any callee-saved registers used (B8–B15) below FR
    4. STK = STK - frame_bytes (rounded for 4-byte alignment)
- Epilogue (typical):
    1. Deallocate locals (if any were allocated manually)
    2. Restore callee-saved registers
    3. LEAVE
    4. RET
- No red zone: The ABI does not define a red zone; interrupts and debuggers may use the stack. Do not place temporaries below STK.

Frame layout (conceptual, higher addresses at top):
- [FR+4]: return address (pushed by caller)
- [FR+0]: saved FR (32 bits)
- [FR-4], [FR-8], ...: saved callee-saved registers as needed
- [FR-N...]: local variables and spill slots (aligned to 16)
- Outgoing stack arguments (beyond B0–B5), placed by caller above the return address prior to transfer

### Call and return mechanics

- With RET defined, a call/return pair is:
    - Caller: (a) place outgoing stack args; (b) push return IP on stack; (c) transfer control to callee entry.
    - Callee: establish frame with ENTERI16 (or manual sequence); on exit, LEAVE; RET to resume caller.
- CALL is defined and pushes the return IP before transferring control. Toolchains may also synthesize a call by pushing the return IP and using J/J32; both forms produce the identical stack layout.

### Variadic functions (C varargs)

- Callers pass fixed arguments per the normal rules.
- Callees that accept varargs should, on entry, spill register arguments B0–B5 into a contiguous home area in their frame (at known offsets), then access the full argument list by first reading those homes, then stack slots beyond the fixed portion. Stack varargs are 4-byte aligned.

### Tail calls

- A function may perform a tail call if, before the final jump, it:
    - Restores all callee-saved registers to their incoming values
    - Restores FR and STK to the caller’s expected state and maintains 4-byte alignment
    - Places target arguments per ABI into B0–B5 and on the stack as necessary

### STAT and flags across calls

- STAT is caller-saved and volatile. Call sites must not rely on STAT values surviving a function call.
- The .F flag-setting instruction variants are allowed within callees; no preservation of Z/S/C/O/P is required at returns.

### Syscall ABI (user ↔ kernel)

- SYSCALL uses B0–B5 for up to six 32-bit arguments; return value in B0. Matches the function ABI for simplicity.
- CTL4 low 8 bits supply the syscall number; CTL4 is kernel-owned (user mode must not modify it). Kernel returns with B0 set and may clobber caller-saved registers and STAT.
- On SYSCALL, consider all caller-saved registers and STAT volatile. Callee-saved registers are preserved by kernel policy only when explicitly documented by the OS.

### Interrupt/exception handlers (kernel)

- Handlers must follow the hardware context order in this spec. A C-callable handler should establish an ABI-conforming frame (save FR if used; preserve callee-saved B8–B15) before calling into C code.
- IRET restores architectural state; no assumptions should be made about STAT beyond the handler’s explicit settings.

### Data layout

- Fundamental types
    - i8/u8: size 1, align 1
    - i16/u16: size 2, align 2
    - i32/u32: size 4, align 4
    - i64/u64: size 8, align 8
    - f32: size 4, align 4
    - pointer: size 4, align 4
- Struct layout: Natural alignment per field; fields are placed in order with padding as required. Overall struct size is rounded up to the max field alignment.
- Arrays: Contiguous elements; alignment equals element alignment.

### Conformance and notes

- Compilers must honor callee-saved preservation for B8–B15, FR, and STK. All others are volatile.
- Leaf functions that do not use callee-saved registers or establish a frame may omit saving FR; they must still preserve STK alignment at calls (if any).
- FP32 values do not require special handling beyond the general rules; they occupy one 32-bit register or stack slot and return in B0 like other scalars.

---

## Relocations and Assembler Notes

Assemblers/linkers targeting B/ should support the following relocations and conveniences:

- REL8/REL32: PC-relative displacements for branches and jumps (BZ/BNZ/…/J/J32/CALL). The displacement is applied relative to the next instruction (IP after the entire instruction).
- ABS32: Absolute 32-bit value for MOVI32/FM0VI32 immediates and data addresses.
- RIPREL32: IP-relative 32-bit for LDRIP.

Recommended pseudo-ops and conveniences:
- `J label` and `BZ label` forms resolve to suitable REL8/REL32 encodings; assemblers may error if the target is out of range for the chosen form unless an explicit long form is used.
- `LI rD, imm32` expands to `MOVI32 rD, #imm32`.
- `CLR rD` expands to `ANDI32 rD, #0`.

Note: Self-modifying code must use `FENCE.I` after patching code bytes to ensure subsequent fetches observe the new instructions.

---

## C and Assembly Examples (ABI in practice)

Below are small C functions and illustrative B/ assembly bodies showing how the ABI and ISA map typical code. Unless noted, examples are leaf bodies that operate purely in registers, so no prologue/epilogue or stack is shown. Entry registers follow the ABI (B0→arg0, B1→arg1, ...); return values in B0.

Note: FMOV is bit-preserving and usable as a generic register move where needed.

### 1) Simple add

C
```
int add(int a, int b) {
    return a + b;
}
```

B/ assembly (body)
```
// Entry: B0=a, B1=b
ADD B0, B1       // B0 = a + b
RET               // return
```

### 2) Add three integers

C
```
int add3(int a, int b, int c) {
    return a + b + c;
}
```

B/ assembly (body)
```
// Entry: B0=a, B1=b, B2=c
ADD B0, B1       // B0 = a + b
ADD B0, B2       // B0 = (a + b) + c
RET
```

### 3) Max of two integers (branch on flags)

C
```
int max(int a, int b) {
    return (a >= b) ? a : b;
}
```

B/ assembly (body)
```
// Entry: B0=a, B1=b; want B0 = max(a,b)
CMP B0, B1        // STAT.S=1 if a<b
BS  use_b         // if S==1, choose b
// fallthrough: keep a in B0
J  done           // unconditional short jump
use_b:
FMOV B0, B1       // B0 = b (bit-preserving move)
done:
RET
```

### 4) Floating fused multiply-add

C
```
float fmaf32(float a, float b, float c) {
    return a * b + c;
}
```

B/ assembly (body)
```
// Entry: B0=a(bits), B1=b(bits), B2=c(bits)
FMOV B3, B0        // save a in B3
FMOV B0, B2        // rD = c in B0 (destination)
FFMA B0, B3, B1    // B0 = fma(a,b,c) with one rounding
RET
```

### 5) Population count

C
```
unsigned popcnt32(unsigned x) {
    return __builtin_popcount(x);
}
```

B/ assembly (body)
```
// Entry: B0=x
POPC B0, B0        // B0 = popcount(x)
RET
```

### 6) Floating max using FCMP + branch

C
```
float fmaxf32(float a, float b) {
    return (a >= b) ? a : b;
}
```

B/ assembly (body)
```
// Entry: B0=a(bits), B1=b(bits)
FCMP B0, B1        // compare a vs b; S=1 if a<b, Z=1 if a==b
BS  use_b          // if a<b, choose b
// fallthrough: keep a in B0
J  done
use_b:
FMOV B0, B1        // return b
done:
RET
```

### 7) Syscall wrapper (example: write)

Assume syscall number 0x01 is write(fd, buf, len) → returns count or negative errno.

C
```
int sys_write(int fd, const void* buf, unsigned len) {
    // B/ convention mirrors function ABI: args in B0–B5, return in B0
    // syscall number placed in CTL4 low 8 bits
    register int r0 __asm__("B0") = fd;
    register const void* r1 __asm__("B1") = buf;
    register unsigned r2 __asm__("B2") = len;
    (void)r0; (void)r1; (void)r2; // illustrative only
    // Actual inline asm depends on toolchain; see assembly body below
    return -1; // placeholder
}
```

B/ assembly (body)
```
// Entry: B0=fd, B1=buf, B2=len
// Load syscall number 1 into a temp (B3) and write CTL4 low 8 bits
ANDI32 B3, #0x00000000  // B3 = 0
ADDI32 B3, #0x00000001  // B3 = 1 (write)
SYSWR CTL[4], B3        // CTL4 low 8 bits = 1
SYSCALL                  // trap to kernel; return value in B0
RET
```

Notes
- Unconditional jumps use J #rel8 (or J32 for longer displacements). Assemblers may also support a label form (J label) that resolves to the appropriate displacement.
- Where a plain integer MOV is preferred, FMOV is used as a bit-preserving move per the spec.

### 8) Function with locals using ENTERI16/LEAVE

C
```
int sum2_local(int a, int b) {
    int t = a + b;   // one 4-byte local
    return t + 1;
}
```

B/ assembly (body)
```
// Entry: B0=a, B1=b; need 4 bytes of locals → frame_bytes=16 for alignment
ENTERI16 #16         // push FR; FR=STK; STK-=16
// Save callee-saved if used (none here)
ADD B2, B0           // B2 = a      (use a temp)
ADD B2, B1           // B2 = a+b
// Optionally spill t at [FR-4] if needed; we keep it in B2
ADDI8 B2, #1         // t+1
FMOV B0, B2          // move result to return register
LEAVE                // STK=FR; FR=[STK]; STK+=4; STK now points to RA
RET                  // pop RA into IP
```

### 9) Directly set SP (STK) and BP (FR)

B/ assembly (body)
```
// Suppose B4 holds a new frame base, and B5 holds a new stack top
WRFR B4            // FR = B4   (set base/frame pointer directly)
WRSTK B5           // STK = B5  (set stack pointer directly)
// Do work...
RDSTK B0           // read back STK into B0
RDFR  B1           // read back FR into B1
RET
```


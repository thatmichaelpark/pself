# Pself
Pself is a single-board computer designed to recreate the experience of working with 1970s-era
computers such as the Cosmac Elf. A Parallax P8X32A "Propeller" microcontroller simulates an 8-bit
microprocessor with 256 bytes of RAM.

## Hardware
### Inputs
- 8 data input toggle switches
- 4 control toggle switches: Run/Stop, Mode, Write-protect, Enter (3-position momentary switch)

### Outputs
- 8 LEDs
- 4 7-segment LED displays

### CPU
- 8-bit data, 8-bit address, made-up instruction set
- Accumulator, program counter, stack pointer, condition codes
- 256 bytes of RAM
- 32k bytes of EEPROM for storage of 128 256-byte blocks

## Operation
 Run/Stop | Mode           | Write-protect | Display |Enter               | Notes
----------|----------------|---------------|---------|--------------------|------
Down (Stop)| Down (Normal)  | Down (R/W)    | AADD [1]| Press up to deposit and advance, down to load address | Modify memory
Down (Stop)| Down (Normal)  | Up (Read-only)| AADD [1]| Press up to advance, down to load address | Examine memory
Down (Stop)| Up (Load/Save) | Down (Load)   | L       | Press up to load 256-byte block | Data input switches specify block (0-127)
Down (Stop)| Up (Load/Save) | Up (Save)   | S       | Press up to save 256-byte block |Data input switches specify block (0-127)
Up (Run)  | Down (Normal)  | (Can be read by program | Under program control | (Can be read by program) | Program is running
Up (Run)  | Up (Step mode) | -             | AADD [2]     | Up to single step  | Program is halted; can single-step

Notes:

[1] Two hex digits for the current address, two hex digits for the contents of that address

[2] Alternates between AADD as in [1] and the output of the halted program

## Pself Programmer's Model

8-bit accumulator (A)
8-bit program counter (PC)
8-bit stack pointer (SP)
Four condition code bits: Z, N, C, V

Eight input ports. Port 0 is 8 data-entry switches. Port 1 is 5 control switches. Ports 2-7 are unassigned.
Port 1 bit 7: Run (active high)
Port 1 bit 6: Mode (active high)
Port 1 bit 2: Protect (active high)
Port 1 bit 1: Enter Address (active low)
Port 1 bit 0: Enter Data (active low)

Eight output ports. Port 0 is 8 LEDs. Ports 1-4 are 7-segment displays, LSD-MSD
Ports 1-4 bit 7: A segment
Ports 1-4 bit 6: B segment
Ports 1-4 bit 5: C segment
Ports 1-4 bit 4: D segment
Ports 1-4 bit 3: E segment
Ports 1-4 bit 2: F segment
Ports 1-4 bit 1: G segment
Ports 1-4 bit 0: Decimal point

### Opcode Format

Pself instructions consist of a 1-byte opcode possibly followed by another byte.
Opcodes are divided into four groups:

Group    | Bit pattern               | Description   
---------|----------------------|---------------------------------------
0-operand   |0000_aaaa          | aaaa (0000..1111) specifies operation                   |
src-operand |0001_bbbb..1100_bbbb  | high nibble specifies operation, low nibble (bbbb) indicates addressing mode [1]
in/out      |1101_deee | d = 0 indicates output; d = 1 indicates input; eee (000..110) indicates port #, eee = 111 means indirect
dst-operand |111f_fffg | f (0000..1111) specifes operation

Notes:

[1] bbbb (0000..1100) is quick immediate addressing (bbbb is the data); bbbb = 1101 is immediate addressing (the following byte is the data); bbbb = 1110 is absolute addressing (the next byte is the address of the data); bbbb = 1111 is indirect addressing (the next byte is the address of the address of the data). Examples:
``` LDA #10 ; quick immediate
 LDA #100 ; immediate
 LDA 100 ; absolute
 LDA [100] ; indirect

## 0-operand Instructions

Mnemonic | Opcode | Description
NOP     | %0000_0000 | No-op
LSL     | %0000_0001 | Logical shift accumulator left
LSR     | %0000_0010 | Logical shift accumulator right
ASR     | %0000_0011 | Arithmetic shift accumulator right
ROL     | %0000_0100 | Rotate accumulator left
ROR     | %0000_0101 | Rotate accumulator right
NEG     | %0000_0110 | Negate accumulator
COM     | %0000_0111 | Logical complement accumulator
POP     | %0000_1000 | Pop top of stack into accumulator
PUSH    | %0000_1001 | Push accumulator onto stack
CLC     | %0000_1010 | Clear carry flag
SEC     | %0000_1011 | Set carry flag
RET     | %0000_1100 | Return (from subroutine)

## Src-operand Instructions

Mnemonic | Opcode | Description
LDA     | %0001_0000 | Load accumulator with data
ADD     | %0010_0000 | Add data to accumulator
ADC     | %0011_0000 | Add data and carry to accumulator
SUB     | %0100_0000 | Subtract data from accumulator
SBC     | %0101_0000 | Subtract data and borrow (!carry) from accumulator
AND     | %0110_0000 | Bitwise AND data and accumulator
OR      | %0111_0000 | Bitwise OR data and accumulator
XOR     | %1000_0000 | Bitwise XOR data and accumulator
CMP     | %1001_0000 | Set condition codes according to (accumulator - data)
TST     | %1010_0000 | Set Z and N according to accumulator

dst-operand
STA INC DEC JMP JSR JZ JNZ JN JNN JC JNC JV JNV
}}

_IN      = %1101_1_000
_OUT     = %1101_0_000

_STA     = %111_0000_0
_CLR     = %111_0001_0
_INC     = %111_0010_0
_DEC     = %111_0011_0
_JMP     = %111_0110_0
_JSR     = %111_0111_0
_JNZ     = %111_1000_0
_JZ      = %111_1001_0
_JNN     = %111_1010_0
_JN      = %111_1011_0
_JNC     = %111_1100_0
_JC      = %111_1101_0
_JNV     = %111_1110_0
_JV      = %111_1111_0


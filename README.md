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

### Opcode format

Pself instructions consist of a 1-byte opcode possibly followed by another byte.
Opcodes are divided into four groups:

Group    | Bit pattern               | z                 | x
---------|----------------------|-------------------|---------------------
0-operand   |0000_aaaa          | aaaa (0000..1111) specifies operation                   |
src-operand |0001bbbb..1100bbbb  | high nibble specifies operation, low nibble (bbbb) indicates addressing mode
in/out      |1101deee | d = 0 indicates output; d = 1 indicates input; eee (000..110) indicates port #, eee = 111 means indirect
dst-operand |111ffffg | f (0000..1111) specifes operation

0: 0-operand    0000aaaa aaaa = 0000..1111
1: src-operand  bbbbcccc bbbb = 0001..1100; cccc = 0000..1100(quick immediate), 1101(immediate), 1110(absolute), 1111(indirect)
2: IN/OUT       1101deee d = 0(output), 1(input); eee = 000..110("immediate"), 111("indirect")
3: dst-operand  111ffffg ffff = 0000..1111; g = 0(absolute), 1(indirect)

0-operand
LSL LSR ASR ROL ROR NEG COM POP PUSH RET CLC SEC

src-operand
ADD ADC SUB SBC AND OR XOR LDA CMP TST

dst-operand
STA INC DEC JMP JSR JZ JNZ JN JNN JC JNC JV JNV
}}
_NOP     = %0000_0000
_LSL     = %0000_0001
_LSR     = %0000_0010
_ASR     = %0000_0011
_ROL     = %0000_0100
_ROR     = %0000_0101
_NEG     = %0000_0110
_COM     = %0000_0111
_POP     = %0000_1000
_PUSH    = %0000_1001
_CLC     = %0000_1010
_SEC     = %0000_1011
_RET     = %0000_1100

_LDA     = %0001_0000
_ADD     = %0010_0000
_ADC     = %0011_0000
_SUB     = %0100_0000
_SBC     = %0101_0000
_AND     = %0110_0000
_OR      = %0111_0000
_XOR     = %1000_0000
_CMP     = %1001_0000
_TST     = %1010_0000

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


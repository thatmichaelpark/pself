# Pself
Pself is a single-board computer designed to recreate the experience of working with 1970s-era
computers such as the Cosmac Elf. A Parallax P8X32A "Propeller" microcontroller simulates an 8-bit
microprocessor with 256 bytes of RAM.

## Hardware
### Inputs
- 8 data input toggle switches
- 4 control toggle switches: Run/Stop, Mode, Read/Write, Enter (3-position momentary switch)

### Outputs
- 8 LEDs
- 4 7-segment LED displays

### CPU (Simulated)
- 8-bit data, 8-bit address, made-up instruction set
- Accumulator, program counter, stack pointer, condition codes
- 256 bytes of RAM
- 32k bytes of EEPROM for storage of 128 256-byte blocks

## Operation
 Run/Stop | Mode           | Read/Write | Display |Enter               | Notes
----------|----------------|---------------|---------|--------------------|------
Down (Stop)| Down (Normal)  | Down (R/W)    | AADD<sup>1</sup>| Press up to deposit and advance, press down to load address | Modify memory. Use data input switches to specify data or address.
Down (Stop)| Down (Normal)  | Up (Read-only)| AADD<sup>1</sup>| Press up to advance, press down to load address | Examine memory. Use data input switches to specify address.
Down (Stop)| Up (Load/Save) | Down (Load)   | L       | Press up to load 256-byte block | Data input switches specify block (0-127)
Down (Stop)| Up (Load/Save) | Up (Save)   | S       | Press up to save 256-byte block |Data input switches specify block (0-127)
Up (Run)  | Down (Normal)  | (Can be read by program | Under program control | (Can be read by program) | Program is running
Up (Run)  | Up (Step mode) | -             | AADD<sup>2</sup>     | Press up to single step  | Program is halted; can single-step

Notes:

<sup>1</sup> Two hex digits for the current address, two hex digits for the contents of that address

<sup>2</sup> Alternates between AADD (as in Note 1) and the output of the halted program

## Pself Programmer's Model

- 8-bit accumulator (A)
- 8-bit program counter (PC)
- 8-bit stack pointer (SP)
- Four condition code bits: Z, N, C, V

- Eight input ports. Port 0 is 8 data-entry switches. Port 1 is 5 control switches. Ports 2-7 are unassigned.
 - Port 1 bit 7: Run (active high)
 - Port 1 bit 6: Mode (active high)
 - Port 1 bit 2: Protect (active high)
 - Port 1 bit 1: Enter Address (active low)
 - Port 1 bit 0: Enter Data (active low)

- Eight output ports. Port 0 is 8 LEDs. Ports 1-4 are 7-segment displays, LSD-MSD
 - Ports 1-4 bit 7: A segment
 - Ports 1-4 bit 6: B segment
 - Ports 1-4 bit 5: C segment
 - Ports 1-4 bit 4: D segment
 - Ports 1-4 bit 3: E segment
 - Ports 1-4 bit 2: F segment
 - Ports 1-4 bit 1: G segment
 - Ports 1-4 bit 0: Decimal point

### Opcode Format

Pself instructions consist of a 1-byte opcode possibly followed by another byte.
Opcodes are divided into four groups:

Group    | Bit pattern               | Description   
---------|----------------------|---------------------------------------
0-operand   |0000_aaaa          | aaaa (0000..1111) specifies operation                   |
src-operand |0001_bbbb..1100_bbbb  | high nibble specifies operation, low nibble (bbbb) indicates addressing mode<sup>1</sup>
in/out      |1101_deee | d = 0 indicates output; d = 1 indicates input; eee (000..110) indicates port #, eee = 111 means indirect
dst-operand |111f_fffg | ffff (0000..1111) specifes operation; g = 0 indicates absolute addressing, g = 1 indicates indirect addressing (address byte follows)

Notes:

<sup>1</sup> bbbb = 0000..1100 indicates quick immediate addressing (i.e., bbbb is the data); bbbb = 1101 indicates immediate addressing (i.e., the next byte is the data); bbbb = 1110 indicates absolute addressing (i.e., the next byte is the address of the data); bbbb = 1111 indicates indirect addressing (i.e., the next byte is the address of the address of the data). Examples:

```
 LDA #10   ; quick immediate
 LDA #100  ; immediate
 LDA 100   ; absolute
 LDA [100] ; indirect
```

### 0-operand Instructions

Mnemonic | Opcode | Description
---------|--------|------------
NOP     | 0000_0000 | No-op
LSL     | 0000_0001 | Logical shift accumulator left
LSR     | 0000_0010 | Logical shift accumulator right
ASR     | 0000_0011 | Arithmetic shift accumulator right
ROL     | 0000_0100 | Rotate accumulator left
ROR     | 0000_0101 | Rotate accumulator right
NEG     | 0000_0110 | Negate accumulator
COM     | 0000_0111 | Logical complement accumulator
POP     | 0000_1000 | Pop top of stack into accumulator
PUSH    | 0000_1001 | Push accumulator onto stack
CLC     | 0000_1010 | Clear carry flag
SEC     | 0000_1011 | Set carry flag
RET     | 0000_1100 | Return (from subroutine)

### Src-operand Instructions

Mnemonic | Opcode | Description
---------|--------|------------
LDA     | 0001_0000 | Load accumulator with data
ADD     | 0010_0000 | Add data to accumulator
ADC     | 0011_0000 | Add data and carry to accumulator
SUB     | 0100_0000 | Subtract data from accumulator
SBC     | 0101_0000 | Subtract data and borrow (!carry) from accumulator
AND     | 0110_0000 | Bitwise AND data and accumulator
OR      | 0111_0000 | Bitwise OR data and accumulator
XOR     | 1000_0000 | Bitwise XOR data and accumulator
CMP     | 1001_0000 | Set condition codes according to (accumulator - data)
TST     | 1010_0000 | Set Z and N according to accumulator

### I/O Instructions

Mnemonic | Opcode | Description
---------|--------|------------
IN      | 1101_1_000 | Read data from specified port into accumulator
OUT     | 1101_0_000 | Write accumulator to specified port

### Dst-operand Instructions

Mnemonic | Opcode | Description
---------|--------|------------
STA     | 111_0000_0 | Store accumulator at specified address
CLR     | 111_0001_0 | Clear byte at specified address
INC     | 111_0010_0 | Increment byte at specified address
DEC     | 111_0011_0 | Decrement byte at specified address
JMP     | 111_0110_0 | Jump to specified address
JSR     | 111_0111_0 | Jump to subroutine at specified address
JNZ     | 111_1000_0 | Jump to specified address if Z = 0
JZ      | 111_1001_0 | Jump to specified address if Z = 1
JNN     | 111_1010_0 | Jump to specified address if N = 0
JN      | 111_1011_0 | Jump to specified address if N = 1
JNC     | 111_1100_0 | Jump to specified address if C = 0
JC      | 111_1101_0 | Jump to specified address if C = 1
JNV     | 111_1110_0 | Jump to specified address if V = 0
JV      | 111_1111_0 | Jump to specified address if V = 1

## Example Programs

### Binary-to-decimal Converter

```
          ; binary-to-decimal converter for the pself computer

          start
00: 1e 81     lda digitBits+0 ; '0'
02: e0 79     sta buff+0  ; lsd
04: 10        lda #0
05: e0 7a     sta buff+1
07: e0 7b     sta buff+2
09: e0 7c     sta buff+3  ; msd

0b: 1d 79     lda #buff
0d: e0 78     sta ptr

0f: d9        in  #1  ; read control switches
10: 61        and #1  ; ENTER?
11: f2 1a     jz  setdecimalmode
13: d9        in  #1
14: 62        and #2  ; ADDR?
15: f0 1d     jnz readinput
          sethexmode
17: 11        lda #1
18: ec 1b     jmp setmode
          setdecimalmode
1a: 10        lda #0
          setmode
1b: e0 80     sta mode

          readinput
1d: d8        in  #0  ; read input switches
1e: d0        out #0  ; echo on LEDs
1f: e0 7e     sta temp
21: 1e 80     lda mode
23: f2 43     jz  convertdecimal

          converthex
25: 1e 7e     lda temp
27: 6d 0f     and #%1111
29: 2d 81     add #digitBits
2b: e0 78     sta ptr
2d: 1f 78     lda [ptr]
2f: e0 79     sta buff+0
31: 1e 7e     lda temp
33: 02        lsr
34: 02        lsr
35: 02        lsr
36: 02        lsr
37: 6d 0f     and #%1111
39: 2d 81     add #digitBits
3b: e0 78     sta ptr
3d: 1f 78     lda [ptr]
3f: e0 7a     sta buff+1
41: ec 5f     jmp output

          convertdecimal
43: 1e 7e     lda temp
          convert
45: e2 7d     clr quot
47: 9a    subloop cmp #10
48: f8 4f     jnc lessthan10
4a: 4a        sub #10
4b: e4 7d     inc quot
4d: ec 47     jmp subloop
          lessthan10
              ; at this point, acc holds remainder after division by 10
4f: 2d 81     add #digitBits
51: e0 7e     sta temp        ; temp points to bit pattern for bcd digit
53: 1f 7e     lda [temp]
55: e1 78     sta [ptr]
57: e4 78     inc ptr
59: 1e 7d     lda quot
5b: f0 45     jnz convert

5d: e4 79     inc buff        ; set decimal point on LSD
          output
5f: 1d 79     lda #buff
61: e0 78     sta ptr
63: 11        lda #1
64: e0 7f     sta port
66: 13        lda #3  ; we only need three digits max
67: e0 77     sta i
          outloop
69: 1f 78     lda [ptr]
6b: e4 78     inc ptr
6d: d7 7f     out port
6f: e4 7f     inc port
71: e6 77     dec i
73: f0 69     jnz outloop

75: ec 00     jmp start

77: 00    i               byte    0
78: 00    ptr             byte    0
79: 00 00 00 00
......... buff            byte    0, 0, 0, 0
7d: 00    quot            byte    0
7e: 00    temp            byte    0
7f: 00    port            byte    0
80: 00    mode            byte    0   ; 0 => decimal output, 1 => hex
          digitBits
81: fc                    byte    %11111100 ; 0
82: 60                    byte    %01100000 ; 1
83: da                    byte    %11011010 ; 2
84: f2                    byte    %11110010 ; 3
85: 66                    byte    %01100110 ; 4
86: b6                    byte    %10110110 ; 5
87: be                    byte    %10111110 ; 6
88: e0                    byte    %11100000 ; 7
89: fe                    byte    %11111110 ; 8
8a: e6                    byte    %11100110 ; 9
8b: ee                    byte    %11101110 ; A
8c: 3e                    byte    %00111110 ; b
8d: 9c                    byte    %10011100 ; C
8e: 7a                    byte    %01111010 ; d
8f: 9e                    byte    %10011110 ; E
90: 8e                    byte    %10001110 ; F
          ; that's all, folks!
```

### Cylon Display

```
          ; cylon display

          start
00: 11            lda     #1
01: e0 37         sta     mask
03: d0            out     #0
04: ee 2c         jsr     Delay
          loop
06: 17            lda     #7
07: e0 35         sta     i
09: ee 1a loop1   jsr     ShiftToTheLeft
0b: e6 35         dec     i
0d: f0 09         jnz     loop1

0f: 17            lda     #7
10: e0 35         sta     i
12: ee 23 loop2   jsr     ShiftToTheRight
14: e6 35         dec     i
16: f0 12         jnz     loop2

18: ec 06         jmp     loop

          ShiftToTheLeft
1a: 1e 37         lda     mask
1c: 01            lsl
1d: e0 37         sta     mask
1f: d0            out     #0
20: ee 2c         jsr     Delay
22: 0c            ret

          ShiftToTheRight
23: 1e 37         lda     mask
25: 02            lsr
26: e0 37         sta     mask
28: d0            out     #0
29: ee 2c         jsr     Delay
2b: 0c            ret

          Delay
2c: 1d 42         lda     #66
2e: e0 36         sta     j
30: e6 36 wait    dec     j
32: f0 30         jnz     wait
34: 0c            ret

35: 00    i       byte    0
36: 00    j       byte    0
37: 00    mask    byte    0
```

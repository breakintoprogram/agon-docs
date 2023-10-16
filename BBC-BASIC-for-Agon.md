# What is BBC BASIC for Agon?

The original version of BBC Basic was written by Sophie Wilson at Acorn in 1981 for the BBC Micro range of computers, and was designed to support the UK Computer Literacy Project. R.T.Russell was involved in the specification of BBC Basic, and wrote his own Z80 version that was subsequently ported to a number of Z80 based machines. [I highly recommend reading his account of this on his website for more details](http://www.bbcbasic.co.uk/bbcbasic/history.html).

As an aside, R.T.Russell still supports BBC Basic, and has ported it for a number of modern platforms, including Android, Windows, and SDL, which are [available from his website here](https://www.bbcbasic.co.uk/index.html).

BBC BASIC for Agon is a port of his BBC BASIC for Z80, which is now open source, with a number of modifications to make it run on Agon.

# Implementation on the Agon Light

BBC BASIC for Z80 runs in Z80 mode, that is within a 64K segment. The interpreter takes around 16K of RAM, leaving around 48K available for user programs and data.

If you are not familiar with the BASIC programming language, or need a refresher on BBC BASIC, please refer to the [official BBC BASIC for Agon documentation here](https://oldpatientsea.github.io/agon-bbc-basic-manual/0.1/index.html).

To run, load into memory and run as follows:

```
LOAD bbcbasic.bin
RUN
```

It is possible to automatically CHAIN (load and run) a BBC BASIC program by passing the filename as a parameter:
```
LOAD bbcbasic.bin
RUN . /path/to/file.bas
```

Note that passing a . as the first parameter of RUN is informing MOS to use the default value there (&40000)

BBC BASIC needs a full 64K segment, so cannot be run from the MOS folder as a star command.

# Summary of Agon Light Specific Changes

## Star Commands

The following * commands are supported

### BYE

Syntax: `*BYE`

Exit BASIC and return to MOS.

### EDIT

Syntax: `*EDIT linenum`

Pull a line into the editor for editing.

### FX

Syntax: `*FX osbyte, params`

Execute an OSBYTE command.

The only OSBYTE commands supported at the moment are:

- 19: Wait for vertical blank retrace

And from MOS 1.03 or above

- 11: Set keyboard repeat delay in ms (250, 500, 750 or 1000)
- 12: Set keyboard repeat rate ins ms (between 33 and 500ms)
- 118: Set keyboard LED (Bit 0: Scroll Lock, Bit 1: Caps Lock, Bit 2: Num Lock) - does not currently change status, just the LED

### VERSION

Syntax: `*VERSION`

Display the current version of BBC BASIC

### MOS commands

In addition, any of the MOS commands can be called by prefixing them with a *

See the MOS documentation for more details

## BASIC

The following statements are not currently implemented:

- ENVELOPE
- ADVAL

The following statements differ from the BBC Basic standard:

### REM

REM does not tokenise any statements within comments. This is to bring it inline with string literals for internationalisation.

### LOAD
### SAVE

The following file extensions are supported:

- `.BBC`: LOAD and SAVE in BBC BASIC for Z80 tokenised format
- `.BAS`: LOAD and SAVE in plain text format (also `.TXT` and `.ASC`)

If a file extension is omitted, ".BBC" is assumed.

### MODE

The modes differ from those on the BBC series of microcomputers. The full list can be found [here](https://github.com/breakintoprogram/agon-docs/wiki/VDP#screen-modes) in the VDP documentation.

### COLOUR

Syntax: `COLOUR c`

Change the the current text output colour

- If c is between 0 and 63, the foreground text colour will be set
- If c is between 128 and 191, the background text colour will be set

The following two commands are only applicable to paletted modes with less than 64 colours.

Syntax: `COLOUR l,p`

Set the logical colour l to the physical colour p

Syntax: `COLOUR l,r,g,b`

### GCOL

Syntax: `GCOL mode,c`

Set the graphics colour to c

This has been extended for VDP 1.04:

* The graphics background colour is set as per COLOUR.
* Mode is applied to graphics operations
  * 0: Use the current foreground colour
  * 4: Invert the pixel

### POINT

Syntax: `POINT(x,y)`

This returns the physical colour index of the colour at pixel position (x, y)

### PLOT

Syntax: `PLOT mode,x,y`

Plot supports the following operations:

- 4: Move
- 5: Line
- 80: Filled Triangle
- 144: Circle with radius specified either by x or y
- 148: Circle passing through point x,y

### GET$

Syntax: `GET$(x,y)`

Returns the ASCII character at position x,y

### GET

Syntax: `GET(x,y)` (from BBC BASIC 1.05)

As GET$, but returns the ASCII code of the character at position x, y

Syntax: `GET(p)`

Read and return the value of Z80 port p

### SOUND

Syntax: `SOUND channel,volume,pitch,duration`

Play a sound through the Agon Light buzzer and audio output jack

- `Channel`: 0 to 2
- `Volume`: 0 (off) to -15 (full volume)
- `Pitch`: 0 to 255
- `Duration`: -1 to 254 (duration in 20ths of a second, -1 = play forever)

### TIME$

Access the ESP32 RTC data

Example:

```
  10 REM CLOCK
  20 : 
  30 CLS
  40 PRINT TAB(2,2); TIME$
  50 GOTO 40
```

NB: This is a virtual string variable; at the moment only getting the time works. Setting is not yet implemented.

### VDU

The VDU commands on the Agon Light will be familiar to those who have coded on Acorn machines. Please read the [[VDP]] documentation for details on what VDU commands are supported.

## Inline Assembler

BBC BASIC for Z80, like its 6502 counterpart, includes an inline assembler. For instructions on usage, please refer to the [original documentation](https://www.bbcbasic.co.uk/bbcbasic/mancpm/bbc3.html#introduction).

In addition to the standard set of Z80 instructions, the following eZ80 instructions have been added

- `MLT`

The assembler will only compile 8-bit Z80 code and there are currently no plans for extending the instruction set much further in this version.

## Integration with MOS

For the most part, the MOS is transparent to BASIC; most of the operations via the MOS and VDP are accessed via normal BBC BASIC statements, with the following exceptions:

### Accessing the MOS system variables

MOS has a small system variables area which is in an area of RAM outside of its 64K segment. To access these, you will need to do an OSBYTE call

Example: Print the least significant byte of the internal clock counter
```
10 L%=&00 : REM The system variable to fetch
20 A%=&A0 : REM The OSBYTE number
30 PRINT USR(&FFF4)
```

For a full list of system variables, please refer to [mos_api.inc](https://github.com/breakintoprogram/agon-mos/blob/main/src/mos_api.inc).

### Running star commands with variables

The star command parser does not use the same evaluator as BBC BASIC, so whilst commands can be run in BASIC, variable names are treated as literals.

Example: This will not work
```
10 INPUT "Filename";f$
20 INPUT "Load Address";addr%
30 *LOAD f$ addr%
```

To do this correctly, you must call the star command indirectly using the OSCLI command

Example: This will work
```
30 OSCLI("LOAD " + f$ + " " + STR$(addr%)) 
```
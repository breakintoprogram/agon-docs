# What is the MOS

The MOS is a command line machine operating system, similar to CP/M or DOS, that provides a human interface to the Agon file system.

It also provides an [MOS API](MOS-API.md) for file I/O and other common operations for BBC Basic and other third-party applications.

## System Requirements

* A 32GB or less micro-SD card formatted FAT32

## The MOS folder

From version 1.02 of MOS, a `mos` folder needs to be created in the root of the SD card. This is for any applications marked as MOS extensions that run off the SD card

## The autoexec.txt file

If the MOS detects an autoexec.txt file in the root folder of the SD card during cold-boot, it will read the file in, and execute the MOS commands in the file sequentially from top to bottom.

For example, to set keyboard to US, load BBC BASIC from the root folder, change to the test folder, then run BASIC

```
SET KEYBOARD 1
LOAD bbcbasic
CD test
RUN
```

## The MOS editor

MOS implements a line editor that can also be used by third-party apps like BBC BASIC.

It implements the following functionality:

- Cursor keys to navigate within the block of text in the line (UP and DOWN for multi-line edits)
- Backspace to delete the character to the left of the current line
- Text is inserted at the current cursor position
- Escape is used to quit the editor without entering the text
- Enter is used to submit the line to the calling application (BBC BASIC, MOS command line, etc)

And from MOS 1.03:

- Cursoring left and right off the edge of the screen will wrap the cursor to the adjacent line
- Home and End keys will cursor to the start and end of the text being edited

And from MOS 1.03:

- Backspace now wraps correctly when backspacing off the left-hand side of the screen
- Pressing the UP arrow key when the buffer is empty will retrieve the last entered command
- Pressing CTRL+N will switch to paged mode, CTRL+O will switch it off. Screen will pause after scrolling a page. Press SHIFT to continue, or ESC to break out. Paged mode should work with any application including BBC BASIC.

## Soft Boot

Press CTRL+ALT+DEL to soft-boot the Z80 (CTRL+SHIFT+ESC for MOS 1.02 or earlier)

NB:

- This assumes that MOS is still talking to the ESP32

## MOS Commands

1. Commands can be abbreviated with a dot, so `DELETE myfile` and `DEL. myfile` are equivalent.
2. Commands are case-insensitive and parameters are space delimited.
3. In the syntax description, optional parameters are written as `<param>`
4. The dot (.) character can be used as a substitution for an optional numeric parameter
5. Default LOAD and RUN address is set to 0x040000
6. Numbers are in decimal and can be prefixed by '&' for hexadecimal.
7. Addresses are 24-bit, unless otherwise specified
	- `&000000 - &01FFFF`: MOS (Flash ROM)
	- `&040000 - &0BDFFF`: User RAM
	- `&0B0000 - &0B7FFF`: Storage for loading MOS star command executables off SD card 
	- `&0BC000 - 0BFFFFF`: Global heap and stack
8. The RUN command checks a header embedded from byte 64 of the executable and can run either Z80 or ADL mode executables 
9. MOS will also search the `mos` folder on the SD card for any executables, and will run those like built-in MOS commands

### CAT

Syntax:`*CAT <path>` (Aliases include `DIR` and `.`)

Directory listing of the current directory. 

NB: The path parameter will only work in MOS 1.03 or greater.

### CD

Syntax:`*CD path`

Change current directory

### COPY

Syntax: `*COPY filename1 filename2`

Create a copy of a file.

NB: Requires MOS 1.03 or greater

### CREDITS

Syntax: `*CREDITS`

Output credits and version numbers for third-party libraries used in the Agon firmware

NB: Requires MOS 1.03 or greater

### DELETE

Syntax: `*DELETE filename` (Aliases include `ERASE`)

Delete a file or folder (must be empty). 

### JMP

Syntax:`*JMP addr`: Jump to the specified address in memory

### LOAD

Syntax: `*LOAD filename <addr>`

Load a file from the SD card to the specified address. If no parameters are passed, then addr will default to &40000.

### MKDIR

Syntax: `*MKDIR filename`

Create a new folder on the SD card

### RENAME

Syntax: `*RENAME filename1 filename2` (Aliases include `MOVE`)

Rename a file in the same folder

`*RENAME autoexec.txt autoexec.bak`

Rename a file and move to a different folder (the destination folder must exist)

`*RENAME test.bas archive/test.bas`

NB: MOVE alias in MOS 1.03 or greater

### RUN

Syntax: `*RUN <addr>`

Call an executable binary loaded in memory. If no parameters are passed, then addr will default to &40000

### SAVE

Syntax: `*SAVE filename addr size`

Save a block of memory to the SD card

### SET

Syntax: `*SET option value`

Set a system option

#### Keyboard Layout

`*SET KEYBOARD n`: Set the keyboard layout

- 0: UK
- 1: US
- 2: German
- 3: Italian
- 4: Spanish
- 5: French
- 6: Belgian
- 7: Norwegian
- 8: Japanese

NB: Keyboard layouts 2 to 8 are only available in MOS 1.03

### TIME

Syntax:
- `*TIME`
- `*TIME yyyy mm dd hh mm ss`

Set and read the ESP32 real-time clock

NB: Requires MOS 1.03 or greater

### VDU

Syntax: `*VDU <char1> <char2> ... <charN>`

Write a stream of characters to the VDP

NB: Requires MOS 1.03 or greater
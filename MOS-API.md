# What is the MOS API

The MOS API can be used by external applications to access MOS functionality

## Usage from Z80 assembler

There are four RST instructions for accessing MOS functionality from Z80.

- `RST 00h`: Reset the eZ80
- `RST 08h`: Execute a MOS command
- `RST 10h`: Output a single character to the VDP
- `RST 18h`: Output a stream of characters to the VDP (MOS 1.03 or above)

In addition, you will probably want to include the file `mos_api.inc` in your project. This can be found in the folder [src](https://github.com/breakintoprogram/agon-mos/tree/main/src) of project [agon-mos](https://github.com/breakintoprogram/agon-mos).

NB:

- Include the file mos_api.inc in your project
- The `RST.LIS` ensures the MOS RST instructions are called regardless of the eZ80s current addressing mode

### RST 08h: Execute a MOS command

Parameters:

- A: MOS command number to execute

NB:

- There is a macro in mos_api.inc with EQUs for all the MOS commands
- Other MOS-command dependant parameters may be required

Macro:

```
;
; Macro for calling the API
; Parameters:
; - function: One of the function numbers listed above
;
MOSCALL:	MACRO	function
		LD	A, function
		RST.LIS	08h
		ENDMACRO 	
```

Example:

```
; OSRDCH: Read a character in from the ESP32 keyboard handler
;
OSRDCH:		MOSCALL	mos_getkey
		OR	A 		
		JR	Z, OSRDCH		; Loop until key is pressed
		RET
```

### RST 10h: Output a single character to the VDP

Parameters:

- A: Character to output

Example:

```
; OSWRCH: Write a character out to the ESP32 VDU handler via the MOS
; A: Character to write
;
OSWRCH:		RST.LIS	10h			; This calls a RST in the eZ80 address space
		RET
```

### RST 18h: Output a stream of characters to the VPD (MOS 1.03 or above)

Parameters:

- HL: Address of the data stream (16-bit for Z80 mode, 24-bit for ADL mode)
- BC: Length of stream (or 0 if the stream is limited)
- A: Stream delimiter (if BC=0)

Example:

```
; Write a stream of characters to the VDP
; HLU: Address of buffer containing data - if in 16-bit segment, U will be replaced by MB
;  BC: Number of characters to write out, or 0 if the data is delimited
;   A: End of data delimiter, i.e. 0 for C strings
;
		LD	HL, text		; Address of text
		LD	BC, 0			; Set to 0, so length ignored...
		LD	A, 0			; Use character in A as delimiter
		RST.LIS	18h			; This calls a RST in the eZ80 address space
		RET
;
text:		DB	"Hello World", 0
```

## MOS commands

MOS commands can be executed from a classic 64K Z80 segment or whilst the eZ80 is running in 24-bit ADL mode. For classic mode, 16 bit registers are passed as pointers to the MOS commands; these are automatically promoted to 24 bit by adding the MB register to bits 16-23 of the register. When running in ADL mode, a 24-bit register will be passed, but MB must be set to 0.

See mos_api.asm for implementation

The following MOS commands are supported

### 0x00: mos_getkey

Read a keypress from the VDP

Parameters: None

Returns:
- A: The keycode of the character pressed

### 0x01: mos_load

Load a file from SD card

Parameters: 
- HL(U): Address of filename (zero terminated)
- DE(U): Address at which to load
- BC(U): Maximum allowed size (bytes)

Returns:
- A: File error, or 0 if OK
- F: Carry reset if no room for file, otherwise set

### 0x02: mos_save

Save a file to SD card

Parameters: 

- HL(U): Address of filename (zero terminated)
- DE(U): Address to save from
- BC(U): Number of bytes to save

Returns:

- A: File error, or 0 if OK
- F: Carry set

### 0x03: mos_cd

Change current directory on the SD card

Parameters: 

- HL(U): Address of path (zero terminated)

Returns:

- A: File error, or 0 if OK

### 0x04: mos_dir

List SD card folder contents

Parameters: 

- HL(U): Address of path (zero terminated)

Returns:

- A: File error, or 0 if OK

### 0x05: mos_del

Delete a file or folder from the SD card

Parameters: 

- HL(U): Address of path (zero terminated)

Returns:

- A: File error, or 0 if OK

### 0x06: mos_ren

Rename a file on the SD card

Parameters: 

- HL(U): Address of filename1 (zero terminated)
- D(E)U: Address of filename2 (zero terminated)

Returns:

- A: File error, or 0 if OK

### 0x07: mos_mkdir

Make a folder on the SD card

Parameters: 

- HL(U): Address of path (zero terminated)

Returns:

- A: File error, or 0 if OK

### 0x08: mos_sysvars

Fetch a pointer to the [system variables](#system-variables)

Parameters: None

Returns:

- IXU: Pointer to the MOS system variable area (this is always 24 bit)

### 0x09: mos_editline

Invoke the line editor

Parameters: 

- HL(U): Address of the buffer
- BC(U): Buffer length
- E: 0 to not clear buffer, 1 to clear

Returns:
- A: Key that was used to exit the input loop (CR=13, ESC=27)

### 0x0A: mos_fopen

Get a file handle

Parameters: 

- HL(U): Address of filename (zero terminated)
- C: Mode

Returns:
- A: Filehandle, or 0 if couldn't open

Mode can be one of: fa_read, fa_write, fa_open_existing, fa_create_new, fa_create_always, fa_open_always or fa_open_append

NB: If you open the file using mos_fopen, you must close it using mos_fclose, not ffs_api_fclose

### 0x0B: mos_fclose

Close a file handle

Parameters: 

- C: Filehandle, or 0 to close all open files

Returns:

- A: Number of files still open

### 0x0C: mos_fgetc

Get a character from an open file

Parameters: 

- C: Filehandle

Returns:
- A: Character read
- F: C set if last character in file, otherwise NC (MOS 1.04 or greater)

### 0x0D: mos_fputc

Write a character to an open file

Parameters: 

- C: Filehandle
- B: Character to write

Returns: None

### 0x0E: mos_feof

Check for end of file

Parameters: 

- C: Filehandle

Returns:
- A: 1 if at end of file, otherwise 0

### 0x0F: mos_getError

Copy an error string to a buffer

Parameters: 

- E: The error code
- HL(U): Address of buffer to copy message into
- BC(U): Size of buffer

Returns: None

### 0x10: mos_oscli

Execute a MOS command

Parameters: 

- HLU: Pointer the the MOS command string
- DEU: Pointer to additional command structure
- BCU: Number of additional commands

Returns:

- A: MOS error code

### 0x11: mos_copy

Copy a file on the SD card

Parameters: 

- HL(U): Address of filename1 (zero terminated)
- D(E)U: Address of filename2 (zero terminated)

Returns:

- A: File error, or 0 if OK

NB: Requires MOS 1.03 or greater

### 0x12: mos_getrtc

Get a time string from the RTC (Requires MOS 1.03 or above)

Parameters:

- HLU: Pointer to a buffer to copy the string to (at least 32 bytes)

Returns:

- A: Length of time string

### 0x13: mos_setrtc

Set the RTC (Requires MOS 1.03 or above)

Parameters:

- HLU: Pointer to a 6-byte buffer with the time data in

```
+0: Year (offset from 1980, so 1989 is 9)
+1: Month (1 to 12)
+2: Day of Month (1 to 31)
+3: Hour (0 to 23)
+4: Minute (0 to 59)
+5: Second (0 to 59)
```

Returns: None

### 0x14: mos_setintvector

Set an interrupt vector (Requires MOS 1.03 or above)

Parameters:

- E: Interrupt vector number to set
- HLU: Address of new interrupt vector (24-bit pointer)

Returns:

- HLU: Address of the previous interrupt vector (24-bit pointer)

### 0x15: mos_uopen

Open UART1 (Requires MOS 1.03 or above)

Parameters:

- IXU: Pointer to a UART struct

```
+0: Baud rate (24-bit, little endian)
+3: Data bits (5, 6, 7 or 8)
+4: Stop bits (1 or 2)
+5: Parity bits (0: None, 1: Odd, 3: Even)
+6: Flow control (0: None, 1: Hardware)
+7: Enabled interrupts
    - Bit 0: Set to enable received data interrupt
    - Bit 1: Set to enable transmit data interrupt
    - Bit 2: Set to enable line status change interrupt
    - Bit 3: Set to enable modem status change interrupt
    - Bit 4: Set to enable transmit complete interrupt
```

To handle the received interrupts, you will need to assign a handler to UART1's interrupt vector (0x1A).

Returns:

- A: Error code (always 0)

### 0x16: mos_uclose

Close UART1 (Requires MOS 1.03 or above)

### 0x17: mos_ugetc

Read a character from UART1 (Requires MOS 1.03 or above)

Returns:

- A: The character read
- F: C if successful, NC if the UART is closed

### 0x18: mos_uputc

Write a character to UART1 (Requires MOS 1.03 or above)

Parameters:

- C: The character to write

Returns:

- F: C if successful, NC if the UART is closed

### 0x19: mos_getfil

Get a pointer to a FIL structure in MOS (Requires MOS 1.03 or above)

Parameters:

- C: Filehandle

Returns:

- HLU: 24-bit pointer to a FIL structure (in MOS RAM)

### 0x1A: mos_fread

Read a block of data from a file (Requires MOS 1.03 or above)

Parameters:

- C: Filehandle
- HLU: Pointer to a buffer to read the data into
- DEU: Number of bytes to read

Returns:

- DEU: Number of bytes read

### 0x1B: mos_fwrite

Write a block of data to a file (Requires MOS 1.03 or above)

Parameters:

- C: Filehandle
- HLU: Pointer to a buffer that contains the data to write 
- DEU: Number of bytes to write out

Returns:

- DEU: Number of bytes written

### 0x1C: mos_flseek

Move the read/write pointer in a file (Requires MOS 1.03 or above)

Parameters:

- C: Filehandle
- HLU: Least significant 3 bytes of the offset from the start of the file
- E: Most significant byte of the offset (set to 0 for files < 16MB)

Returns:

- A: FRESULT

***

## FatFS commands

In addition to the MOS commands, it is possible to access FatFS at a lower level

### 0x80: ffs_fopen

Open a file (Requires MOS 1.03 or above)

Parameters:

- HL(U): Pointer to an empty FIL structure
- DE(U): Pointer to a C (zero-terminated) filename string
- C: File open mode

Preserves: HL(U), DE(U), C

Returns:

- A: FRESULT

Example:

```
			LD	HL, fil				; FIL buffer
			LD	DE, filename			; Filename (0 terminated)
			LD	C, fa_read			; Mode
			MOSCALL	ffs_fopen			; Open the file
			LD	DE, buffer			; Where to store the read file
			LD	BC, 256				; Number of bytes to read
			MOSCALL	ffs_fread			; Read the data in
			PUSH	BC				; Preserve number of bytes read
			MOSCALL	ffs_fclose			; Close the file
			POP	BC				; BC: Number of bytes read
			RET 

filename:		DB	"example.txt", 0		; The file to read

fil:			DS	FIL_SIZE			; FIL buffer (defined in mos_api.inc)
buffer:			DS	256				; Buffer for storing read data
```

### 0x81: ffs_fclose

Close a file (Requires MOS 1.03 or above)

Parameters:

- HL(U): Pointer to a FIL structure

Preserves: HL(U)

Returns:

- A: FRESULT

See ffs_fopen for an example

### 0x82: ffs_fread

Read from a file (Requires MOS 1.03 or above)

Parameters:

- HL(U): Pointer to a FIL structure
- DE(U): Pointer to a buffer to store the data in
- BC(U): Number of bytes to read (typically the size of the buffer)

Preserves: HL(U), DE(U)

Returns:

- BC(U): Number of bytes read
- A: FRESULT

See ffs_fopen for an example

### 0x83: ffs_fwrite

Write to a file (Requires MOS 1.03 or above)

Parameters:

- HL(U): Pointer to a FIL structure
- DE(U): Pointer to a buffer to read the data from
- BC(U): Number of bytes to write (typically the size of the buffer)

Preserves: HL(U), DE(U)

Returns:

- BC(U): Number of bytes written
- A: FRESULT

Example:

```
			LD	HL, fil				; FIL buffer
			LD	DE, filename			; Filename (0 terminated)
			LD	C, fa_write | fa_create_always	; Mode
			MOSCALL	ffs_fopen			; Open the file
			LD	DE, buffer			; Location of data to write
			LD	BC, 256				; Number of bytes to write
			MOSCALL	ffs_write			; Write the data
			MOSCALL	ffs_fclose			; Close the file
			RET 

filename:		DB	"example.txt", 0		; The file to read

fil:			DS	FIL_SIZE			; FIL buffer (defined in mos_api.inc)
buffer:			DS	256				; Buffer containing data to write out
```

### 0x8E: ffs_feof

Detect end of file (Requires MOS 1.03 or above)

Parameters:

- HL(U): Pointer to a FIL structure

Preserves: HL(U)

Returns:

- A: 1 if at the end of the file, otherwise 0

### 0x96: ffs_stat

Get file information (Requires MOS 1.03 or above)

Parameters:

- HL(U): Pointer to a FILINFO structure
- DE(U): Pointer to a C (zero-terminated) filename string

Preserves: HL(U), DE(U)

Returns:

- A: FRESULT

Example:

```
			LD	HL, filinfo			; FILINFO buffer
			LD	DE, filename			; Filename (0 terminated)
			MOSCALL	ffs_stat
			RET

filename:		DB	"example.txt", 0		; The file to read

filinfo:		DS	FILINFO_SIZE			; FILINFO buffer (defined in mos_api.inc)
```

***

## System Variables

The MOS API command [mos_sysvars](#0x08-mos_sysvars) returns a pointer to the base of the MOS system variables area in IXU - a 24-bit pointer.

The following system variables are available in [mos_api.inc](#usage-from-z80-assembler):

```
; System variable indexes for api_sysvars
; Index into _sysvars in globals.asm
;
sysvar_time:		EQU	00h	; 4: Clock timer in centiseconds (incremented by 2 every VBLANK)
sysvar_vpd_pflags:	EQU	04h	; 1: Flags to indicate completion of VDP commands
sysvar_keyascii:	EQU	05h	; 1: ASCII keycode, or 0 if no key is pressed
sysvar_keymods:		EQU	06h	; 1: Keycode modifiers
sysvar_cursorX:		EQU	07h	; 1: Cursor X position
sysvar_cursorY:		EQU	08h	; 1: Cursor Y position
sysvar_scrchar:		EQU	09h	; 1: Character read from screen
sysvar_scrpixel:	EQU	0Ah	; 3: Pixel data read from screen (R,B,G)
sysvar_audioChannel:	EQU	0Dh	; 1: Audio channel 
sysvar_audioSuccess:	EQU	0Eh	; 1: Audio channel note queued (0 = no, 1 = yes)
sysvar_scrWidth:	EQU	0Fh	; 2: Screen width in pixels
sysvar_scrHeight:	EQU	11h	; 2: Screen height in pixels
sysvar_scrCols:		EQU	13h	; 1: Screen columns in characters
sysvar_scrRows:		EQU	14h	; 1: Screen rows in characters
sysvar_scrColours:	EQU	15h	; 1: Number of colours displayed
sysvar_scrpixelIndex:	EQU	16h	; 1: Index of pixel data read from screen
sysvar_vkeycode:	EQU	17h	; 1: Virtual key code from FabGL
sysvar_vkeydown		EQU	18h	; 1: Virtual key state from FabGL (0=up, 1=down)
sysvar_vkeycount:	EQU	19h	; 1: Incremented every time a key packet is received
sysvar_rtc:		EQU	1Ah	; 8: Real time clock data
sysvar_keydelay:	EQU	22h	; 2: Keyboard repeat delay
sysvar_keyrate:		EQU	24h	; 2: Keyboard repeat rate
sysvar_keyled:		EQU	26h	; 1: Keyboard LED status
sysvar_scrMode:		EQU	27h	; 1: Screen mode (from MOS 1.04)
```
Example: Reading a virtual keycode in ADL mode (24-bit):
```
		MOSCALL	mos_getkey
		LD	A, (IX + sysvar_vkeycode)	; Load A with the virtual keycode from FabGL
```
Example: Reading a virtual keycode in Z80 mode (16-bit):
```
		MOSCALL	mos_getkey
		LD.LIL	A, (IX + sysvar_vkeycode)	; Load A with the virtual keycode from FabGL
```
# General

## Are these documents available offline or in a different format?

No, and there are no immediate plans on doing this to reduce the amount of time spent doing documentation.

## Is the Agon open source?

The official hardware and software components that are part of the Quark firmware are open source, including BBC BASIC for Agon. Please check the license of any third party applications.

## What else do I need to buy?

Okay, so you've ordered your Agon Light, and are wondering what else you will need to purchase.

Minimum:
- A PS/2 compatible keyboard
- A micro USB card

NB:
- The orginal Agon Light requires a PS/2 keyboard, or a USB keyboard that supports the PS/2 protocol with a USB to PS/2 adaptor.
- The Agon Light 2 requires a USB keyboard that supports the PS/2 protocol, or a PS/2 keyboard with a PS/2 to USB adaptor.

# Software

## Does AGON support CP/M?

Officially it is not designed to run CP/M out of the box. There are however third-party developers who are porting CP/M to the platform, and the official build will support this.

# Software Development

## Do I need the ZDS Smart Cable?

Probably not, unless you are interested in developing software on the eZ80F92 Flash ROM.

## Is BASIC the only programming language available?

The AGON comes pre-shipped with BASIC as it is a good language to start coding with, but you are not limited to it. The following languages are supported:

- Forth
- Assembly Language (either via inline assembler or external assembler)
- C (via ZDS tools)

## Are there any simple BBC BASIC examples?

Your AGON may have a tests folder with some BBC BASIC examples; if it doesn't, they can be [found here](https://github.com/breakintoprogram/agon-bbc-basic/tree/main/tests).

## Can you code feature {x}?

All suggestions are welcome, though the developers will be concentrating on key features. If we think your idea has legs, we'll add it to the pile.

# Hardware

## What is the difference between the Agon Light and the Olimex Agon Light 2?

The main differences are:

- The keyboard connector is USB, yet still requires a keyboard that supports the PS/2 protocol
- LIPO battery charging circuit
- UEXT connector
- USB C power connector
- Plastic boxed 34-pin GPIO connector 

And there are some minor revisions to discrete components on the board. Other than that, it is functionally identical to the original Agon Light design.

### Can you recommend a PS/2 keyboard?

Sure, there's a crowd-sourced spreadsheet [here](https://docs.google.com/spreadsheets/d/1-6_sz6l-vJW5rFg3M0Y6bwC0hmFS7U6PPNjIZ9plrM8/edit?fbclid=IwAR0SBuM-oCywGE7Km6PupIWpiKnQNXIHQ2hD7iSDo5T7b_LTHXX0JNEe3Fw#gid=0) with a list of known working keyboards.

If you have one, and it's not on the list, then please feel free to add an entry.

### I'm having issues with some video modes

Make sure you have PSRAM enabled in the [Arduino IDE settings](https://github.com/breakintoprogram/agon-vdp#arduino-ide-settings) when you compile and transfer the VDP code.

## Can you add support for hardware {x}?

All suggestions are welcome. If we think your idea has legs, we'll add it to the pile.
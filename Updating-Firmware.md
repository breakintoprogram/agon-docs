## How to upgrade from Quark 1.02 to 1.03 using the Agon-Flash utility

Instructions kindly collated by Tim Delmare

### Disclaimer

The responsibility for any issues during and/or after the firmware upgrade lies with the user

### Host PCs and OS

This has been tested on:

- Windows 10
- Windows 11
- Ubuntu 20.04 LTS
- Ubuntu 22.04 LTS

There are reported compatibility issues with the Arduino IDE / ESPTools running on MacOS, especially with the M series CPUs. I have collated some of the workarounds in the 'Configuring the Arduino IDE' steps.

### Prerequisites

#### You will need

- A USB cable (the one you are using to power the Agon is probably fine, as long as it supports data as well as power, which most do)
- A microSD card (the one you are using in the Agon)
- A microSD card adapter for your PC

#### Download the following components to your PC

- [MOS 1.03 binary](https://github.com/breakintoprogram/agon-mos/releases/download/v1.03/MOS.bin)
- [VDP 1.03 source code](https://github.com/breakintoprogram/agon-vdp/archive/refs/tags/v1.03.zip)
- [BBC BASIC 1.04 binary](https://github.com/breakintoprogram/agon-bbc-basic/releases/download/v1.04/bbcbasic.bin)
- [Agon MOS Flash Upgrader](https://github.com/envenomator/agon-flash/releases/download/v1.2/flash.bin)

#### Configure the Arduino IDE

If you have already configured the Arduino IDE for Agon then you can skip this step, otherwise follow the instructions here: [Arduino IDE Settings](https://github.com/breakintoprogram/agon-vdp#arduino-ide-settings)

### Update Steps

If you do not have the Zilog cable then the order in which you do things is especially important. Update the MOS first. If you update the VDP first, you will not be able to update MOS using the Agon MOS Flash Upgrader due to changes in the MOS.

#### 1. Update the MOS

1. Mount the microSD card in your PC and create a directory call `mos` in the root directory if one does not already exist. Copy flash.bin to this directory
2. Copy MOS.bin to the root directory
3. Unmount the microSD card and insert it in your Agon
4. Power the Agon on
5. Type `*BYE` to quit BBC BASIC
6. Type `CD /` to return to root directory (you can also use backslash)
7. Type `FLASH MOS.bin 0x81E397C9`
8. Power off

Once this has succeeded the Agon will not work properly until you upgrade VDP.

#### 2. Update BBC BASIC

1. Power the Agon off
2. Remove the SD card and return to your PC
3. Copy bbcbasic.bin to the root directory, overwriting any existing file
4. Copy the following folders to the root directory to update the examples (optional, but recommended)
   - examples
   - resources
   - tests
5. Eject the SD card and return it to the Agon

#### 3. Update the VDP

1. Create a folder called video and unzip the VDP source code inside this new folder
2. Open video.ino in the Arduino IDE
3. Connect the Agon to the computer using your USB cable
4. Identify the port and set it in the Arduino IDE
5. Now select Upload from the Sketch menu

Note that compilation may take several minutes. Once this is complete, and the Arduino IDE has uploaded the new code to the ESP32 on the Agon, the Agon should automatically reboot with the latest and greatest.
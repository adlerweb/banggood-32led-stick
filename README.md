This repository contains hardware documentation and software for Banggoods ["DIY 32 Bit LED Shake Stick Kit"](http://goo.gl/JAC3Vc). This repository is a supplement for [BitBastelei #205](https://www.adlerweb.info/blog/2016/07/24/bitbastelei-205-diy-32-bit-led-shake-stick-kit-banggood)(German).

# Hardware

The design itself is based on a [STC11F02E microcontroller](http://www.stcmcu.com/datasheet/stc/STC-AD-PDF/STC11F-10Fxx-english.pdf) (8051-compatible, 2KB Flash, 2KB EEPROM). All LEDs are connectred in a 3-way-matrix (lines, rows and polarity). An additional direction-switch and a push button are also available. The chip can be programmed using a serial connection which is available on a separate pin header.

Design notes:
 * While the battery case is build for 4xAA you should only use 3 and short the last one to avoid overvoltage. [See Instructions](http://forum.banggood.com/forum-topic-149391.html).
 * LEDs are not current limited and must be pulsed quickly to avoid damage
 * The IC is inteded for <=20mA and most likely overused

The provided design files can be opened using [KiCad](http://kicad-pcb.org/). The schematic schould be correct, the board layout is only a coarse drawing and should not be used as a reference without further checks 


# Software

The software is written in C and intended to be compiled using [sdcc](http://sdcc.sourceforge.net/). A project file for [MCU 8051 IDE](http://www.moravia-microsystems.com/mcu-8051-ide/) is included. Both tools are available for linux under a open source license, Windows-Versions should also be available.

In it's current state some basic functionallity is given, but the software is far from complere, see below for a list of known flaws.

## How the software works

At the top of the file a array of longs is defined. Every element represents one column of 32 LEDs, the left/highest bit will be drawn at the base of the unit. The first element will be drawn first. You can make the array wider or smaller using GRAPHSIZE above.

To draw a column the code will iterate every LED, switching the corresponding ports for LEDs to be lit either LOW or HIGH accordingly. LEDs and pins not used well be set to High-Z to avoid involuntary driving of other LEDs on the lines/columns of the LED-array. 

The main program will first inizialize all ports as inputs (High-Z) with an exception for both buttons which will be driven using the "quasi bidirectional" mode, allowing us to use the MCU as pull-up.

The main loop will start drawing once the push-button is pressed. This should be replaced with the direction switch for proper drawing. It will begin using the first array element and draw it several times. If the timeout is up it will continue with the second column, etc. Drawing will automatically stop when reaching the last element or releasing the button.

The timeout is calculated dynamically, the code will start with one and increase the number of repetition if the draw was finished before releasing the button (of later changing the direction again). Thes isn't really feasible when using the push-button but should somewhat work one the direction switch is used.

## How the software should work

Have a nice whish-list of things, that should be improved…

* Fix the direction-switch. I wasn't able to get the direction switch working right away and the push-button was enough for demonstration. Shouldn't be too much work
* The graph should be stored in EEPROM instead of flash memory as it is now - this should noticably increase available space and should allow for multiple graphs
* A more or less intelligent way to compress the graphs may also free up space, but I'm not sure if loosing the human-redability and a lot of processing power is worth the effort
* The 2-line-speed-algorithm may need a bit of care to be reliable
* Currently every LED is driven separatly. While this is easy to implement and keeps the current down it is also fscking slow. Grouping lines, columns or whatever should save execution time and allow for higher horizontal resolution and nicer text
*  It should be possible to implement a basic font and add a "programming mode" (power up with button pressed?) to allow for easier customization without the need to recompile the code every time


## Flashing

The IC can be programmed using a serial bootloader. You need a USB so TTL serial adapter to connect your PC to the board. Connect GND to GND, VCC to VCC/5V and cross RX/TX. On the software side you can use [stcgal](https://github.com/grigorig/stcgal) on linux or [the official tool](http://www.stcmcu.com/STCISP/stc-isp-15xx-v6.63.exe) for windows (which is afaik not in english…). You have to start the flashing process on your PC first and then powercycle the IC to enter the serial bootloader.

For Linux you can use this command:

```stcgal -P stc12 -p /dev/ttyUSB0 main.hex```

Yes, the LED-board is using a stc11-IC, but the protocol is identical. Please be aware you need full access to your serial programmer, depending on your setup you have to start the process as root or use sudo.

# Thanks

* [Banggood](http://banggood.com) for sponsoring the kit (and getting me to learn 8051-C)
* [Grigori Goronzy and all stcgal contributors](https://github.com/grigorig/stcgal) for saving me from wine with chinese fonts
* The team behind [sdcc](http://sdcc.sourceforge.net/) so I'm not required to write my own STC11 header files
* The guys behind [MCU 8051 IDE](http://www.moravia-microsystems.com/mcu-8051-ide/) so I hadn't to read the sdcc man page
* The whole [KiCad](http://kicad-pcb.org/) Folks because it still rocks
* You for providing patches, right? ;)

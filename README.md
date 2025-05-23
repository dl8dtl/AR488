# AR488 Arduino GPIB Interface


The AR488 GPIB controller is an Arduino-based controller for interfacing with IEEE488 GPIB devices via USB. This work was inspired by and has been based on the work originally released by Emanuele Girlando and has been released with his permission.

This sketch represents a rewrite of that work and implements the full set of Prologix ++ commands in both controller and device mode, with the exception of ++help. Secondary GPIB addressing is not yet supported. A number of additional features are provided, for example, a macro feature is provided to allow automation of frequently used command sequences was well as controller and instrument initialisation at startup. As of version 0.48.x, interfacing with SN75160 and SN75161 GPIB transceiver integrated circuits is supported.

To build an interface, at least one Arduino board will be required to act as the interface hardware. Arduino boards provide a low cost alternative to other commercial interfaces. Currently the following boards are supported:

<table>
<tr><td><i>MCU</i></td><td><i>Board</i></td><td><i>Serial Ports</i></td><td><i>Comments and layouts</i></td></tr>
<tr><td>328p</td><td>Uno R3</td><td>Single UART shared with USB</td><td>Layout as per original project by Emanuelle Girlando. Only two pins (6 & 13) remain spare as the remainder are all used to interface with the GPIB bus.</td></tr>

<tr><td>328p</td><td>Nano</td><td>USB/Single UART shared with USB</td><td>Identical to Uno.</td></tr>

<tr><td>328p</td><td>Logic Green LG8FX</td><td>USB/Single UART shared with USB</td><td>Identical to Nano, but might have a different UART IC</td></tr>


<tr><td>32u4</td><td>Micro</td><td>USB/CDC+1 UART</td><td>Compact layout by Artag, designed for his back-of-IEEE488-plug adapter board. All GPIO pins are used so none remain spare. Some Micro boards may not have a reset button, in which case the reset pin need to be briefly shorted to ground by some other means.</td></tr>

<tr><td>32u4</td><td>Leonardo R3</td><td>USB/CDC+1 UART</td><td>Same layout as the UNO, leaving the same two pins (6 & 13) spare.</td></tr>

<tr><td>644</td><td>MightyCore ATMega644</td><td>USB/Single UART</td><td>Similar in form factor to the Nano but an MCU with more memory and additional GPIO pins over the UNO ir Nano</td></tr>

<tr><td>1284</td><td>MicroCore ATMega1284</td><td>USB/Single UART</td><td>Same form factor and layout as the MightyCore 644</td></tr>

<tr><td>2560</td><td>Mega 2560</td><td>4 x UART, Serial0 shared with USB</td><td>
Costs slightly more but has a design and layout that includes considerably more control pins, additional serial ports and a more powerful ATmega2560 MCU. It therefore provides greater flexibility and potential for further expansion. Currently, 3 layouts are provided for the AtMega2650 using either the lower numbered pins on the sides of the board (<D>efault), the first row of pins of the two row header at the end of the board (E1), or the second row of the same header (E2). This provides some flexibility and allows various displays and other devices to be connected if desired. Please be aware that when using the <D>efault layout, pins 16 and 17 that correspond to TXD2 and RXD2 (Serial2) cannot be used for serial communication as they are used to drive GPIB signals, however serial ports 0, 1 and 3 remain available for use. Layouts E1 and E2 do not have the same restriction.
</td></tr>

<tr><td>RP2040</td><td>Raspberry Pico</td><td>Two serial ports. DFU mode.</td><td>A compact board with more GPIO pins and features than UNO or Nano and runs at a much higher clock frequency. However, it runs at 3.3V and may require level shifters and buffer ICs. Two layouts L1 and L2 available.</td></tr>
</table>

Including the SN7516x chipset into the interface design will naturally add to the cost, but has the advantage of providing the full 48mA drive current capacity regardless of the capability of the Arduino board being used, as well as providing proper tri-state output with Hi-Z when the board is powered down. The latter isolates the Arduino micro-controller from the GPIB bus when the interface is powered down, preventing GPIB bus communication problems due to 'parasitic power' from signals present on the GPIB bus, thereby allowing the interface to be safely powered down while not in use. Construction will involve adding a daughter-board between the Arduino GPIO pins and the GPIB bus. This could be constructed using prototyping board or shield, or custom designed using KiCad or other PCB layout design software.

To use the sketch, create a new directory, and then unpack the .zip file into this location. Open the main sketch, AR488.ino, in the Arduino IDE. This should also load all of the linked .h and .cpp files. Review Config.h and make any configuration adjustment required (see the 'Configuration' section of the AR488 manual for details), including the selcetion of the board layout selection appropriate to the Arduino board that you are using. Set the target board in Board Manager within the Arduino IDE (Tools => Board:), and then compile and upload the sketch. There should be no need to make any changes to any other files. Once uploaded, the firmware should respond to the ++ver command with its version information.

Please note that the Arduino UNO, Nano and Mega 2560 perform a reset when a connection is made to their serial port. This is by design so as to allow the board to be programmed. The Micro and other 32u4 boards (e.g. Leonardo) as well as the Raspberry Pico do not reset in this manner and the serial port responds immediatey. These differing behaviours may affect interaction with third party applications and scripts. The Arduino IDE takes care of the programming process for all of the supported boards via USB and should work normally. Boards can also be programmed using a USBASP programmer or similar. 

When using the Arduino IDE on Linux (Linux Mint and possibly other Ubuntu derivatives), it should be bourne in mind that the modemmanager and brltty services are designed to interact with serial ports and are known to interfere with the programming process. This maty result in boards being rendered no longer accessible via USB. If this ocurrs, the board can be restored to normal operation by re-programming it using a suitable programmer (e.g. USBASP) and having the bootloader flashed back onto it. It is therefore advisable to disable these services.

Unless some form of shield or custom design with integral IEEE488 connector is used, connecting to an instrument will require a 16 core cable and a suitable IEEE488 connector. This can be salvaged from an old GPIB cable or purchased from various electronics parts suppliers. Searching for a 'centronics 24-way connector' sometimes yields better results than searching for 'IEEE 488 connector' or 'GPIB connector'. Details of interface construction and the mapping of Arduino pins to GPIB control signals and data bus are explained in the "Building an AR488 GPIB Interface" section of the <a href="https://github.com/Twilight-Logic/AR488/blob/master/AR488-manual.pdf">AR488 Manual</a>.
 
Commands generally adhere closely to the Prologix syntax, however there are some minor differences, additions and enhancements. For example, due to issues with longevity of the Arduino EEPROM memory, the ++savecfg command has been implemented differently to save EEPROM wear. Some commands have been enhanced with additional options and a number of new custom commands have been added to provide new features that are not found in the standard Prologix implementation. Details of all commands and features can be found in the Command Reference section of the <a href="https://github.com/Twilight-Logic/AR488/blob/master/AR488-manual.pdf">AR488 Manual</a>.

Once uploaded, the firmware should respond to the ++ver command with its version information.

<b><i>Wireless Communication:</i></b>

The 32u4 and mega 2560 boards have additional serial ports which can be used to connect the ESP8266 WiFi add-on or the HC05 bluetooth module. The firmware sketch supports auto-configuration of the Bluetooth HC05 module, the details of which can be found in the AR488 manual <a href="https://github.com/Twilight-Logic/AR488/blob/master/AR488-Manual.pdf">AR488 Manual</a>. It is also possible to use a HC06 module, but since this module is capable of operating in slave mode only, automatic configuration is not possible. It will therefore need to be configured manually.

Using these wireless modules in conjunction with the Uno or Nano is not advised as the only available serial UART is also used for USB communication. Serial protocols were not designed to accomodate multiple devices on a single UART. Communication problems may arise when both USB and a serial device on RX0/TX0 are connected and communicating with the MCU at the same time. It is possible instead to use SoftwareSerial (TX = pin 6, RX = pin 13) although at a speed of no more than 57600 baud.

The ESP32 is supported via the excellent work done by David Douard. Have a look at his <a href="https://github.com/douardda/AR488-ESP32">ESP-488 project</a>.

<b><i>Obtaining support:</i></b>

In the event that a problem is found, this can be logged via the Issues feature on the AR488 GitHub page. Please provide at minimum:

- the firmware version number
- the type of board being used
- the make and model of instrument you are trying to control
- a description of the issue including;
- what steps are required to reproduce the issue

Comments and feedback can be provided here:<BR>
https://www.eevblog.com/forum/projects/ar488-arduino-based-gpib-adapter/

<b><i>Acknowledgements:</i></b>
<table>
<tr><td>Emanuelle Girlando</td><td>Original project for the Arduino Uno</td></tr>
<tr><td>Luke Mester</td><td>Testing of original Uno/Nano verions against Prologix</td></tr>
<tr><td>Artag</td><td>Porting to the Arduino Micro (32u4) board</td></tr>
<tr><td>Tom DG8SAQ</td><td>Plotting and printing</td></tr>
<tr><td>douardda</td><td>Support with project maintenance, addition of ++help command, ESP32 port</td></tr>
<tr><td>Monty McGraw</td><td>Testing of MicroCore ATMega644 and ATMega1024 boards</td></tr>
<tr><td>justjason</td><td>Testing of Logic Green LG8FX board</td></tr>
<tr><td>tom_iphi</td><td>Testing of FindAllListeners function and secondary addressing</td></tr>
</table>

Also, a big thank you to all the contributors to the AR488 EEVblog thread for their suggestions and support.

The original work by Emanuele Girlando is found here:<BR>
http://egirland.blogspot.com/2014/03/arduino-uno-as-usb-to-gpib-controller.html


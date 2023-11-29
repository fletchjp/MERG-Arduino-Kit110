<img align="right" src="arduino_cbus_logo.png"  width="150" height="75">

# CANshield

An Arduino programm for the testing of the Arduino CAN Shield (MERG Kit Locker #110).
The kit can be built either with an Arduino NANO mounted onto it , or as a shield to fit onto an Arduino UNO or MEGA.
Note that for a MEGA some pin allocations are different.

This sketch provides the code which will enable users of the Arduino CAN Shield to test functionality of the shield.
Sketch is the Arduino name for the program code which is run on an Arduino.

Key Features:
- MERG CBUS interface.
- This can be configured to a CBUS network using the FLiM Configuration Utility (FCU) which only runs on Windows computers.
- This can be done EITHER using the push button on the shield OR the Serial Monitor as described below.

## Overview

The program is written in C++ but you do not need to understand this to use the program.

## Loading the code and libraries

In order to load the code it is necessary to load the Arduino IDE (Interactive Development Environment) onto a computer.
There are currently two versions, the older 1.8.19 and the newer 2.2.1. Either will do for this.
The IDE is available for different versions of Windows and also for other systems.
The IDE will manage both the code and also several libraries which can be loaded using a Library Manager.
All of the libraries needed for CANshield are available through the Library Manager which also prompts with updates for installed libraries.
The user installs libraries into a fixed location which has all of the libraries which have been loaded, available for any sketch.

Whenever a sketch is to be used the Arduino IDE calls a C++ compiler which compiles the code and a copy of each of the libraries needed. This can be done as a check that it allo works and then loaded onto the Arduino using the connection.

On a Windows computer the USB connection to the computer will be via a COM port and it is necessary to tell the Arduino IDE which COM port to use.

## Using CANshield

The MCP2515 interface requires five Arduino pins to be allocated. Three of these are fixed
in the architecture of the Arduino processor. One pin must be connected to an interrupt
capable Arduino pin. On the shield, this is set to pin 2 but a jumper can be removed and a
wire link used to connect to another interrupt pin if desired.  This may be appropriate
when using a Mega with additional interrupt inputs.

The Chip Select pin on the shield is connected to pin 10.  Once again, by removing the relevant
jumper, another pin can be wire linked for this function.

If you change the interrupt or chip select pins, make sure that you update the relevant pin 
allocations in the sketch.

If the MERG Kit 110 CAN Shield is used, the following pins are connected by default:

Pin | Description
--- | ---
Digital pin 2 | Interupt CAN
Digital pin 10| (SS)    CS    CAN
Digital pin 11| (MOSI)  SI    CAN
Digital pin 12| (MISO)  SO    CAN
Digital pin 13| (SCK)   Sck   CAN

Using the CAN Shield, the following pins are used for CBUS Initialisation:

Pin | Description
--- | ---
Digital pin 4 | CBUS Green LED
Digital pin 7 | CBUS Yellow LED
Digital pin 8 | CBUS Switch

**It is the users responsibility that the total current that the Arduino is asked to supply 
stays within the capacity of the on board regulator.  Failure to do this will result in 
terminal damage to your Arduino.**

Pins defined as inputs are active low.  That is to say that they are pulled up by an 
internal resistor. The input switch should connect the pin to 0 Volts.

Pins defined as outputs are active high.  They will source current to (say) an LED. It is 
important that a suitable current limiting resistor is fitted between the pin and the LED 
anode.  The LED cathode should be connected to ground.

### Library Dependencies

The following third party libraries are required:

Library | Purpose
---------------|-----------------
Streaming  |*C++ stream style output, v5, (http://arduiniana.org/libraries/streaming/)*
Bounce2    |*Debounce of switch inputs*
ACAN2515   |*library to support the MCP2515/25625 CAN controller IC*
CBUS2515   |*CAN controller and CBUS class*
CBUSconfig |*module configuration*
CBUS      |*CBUS Class*
CBUSParams   |*Manage CBUS parameters*
CBUSSwitch   |*Manage CBUS switch*
CBUSLED      |*Manage CBUS indicator yellow/green LEDs*

### Application Configuration

The example is configured for use with the MERG Kit 110 CAN Shield. This has a crystal
frequency of 16MHz.

#### Consumed Events

The 4 LEDs that are controlled by consumed events are connected to pins 3, 5, 6 & 9. These
pins are ACTIVE LOW.  In other words, when ON, the pin will go low. The LED Cathode (in a 
circuit diagram this is the point of the arrow with the bar across it) should be connected 
to the pin and the ANode connected to +5V via a current limiting resistor.  Allowing for
the forward voltage drop of the LED (roughly 2V depending upon colour), a 1K8 resistor will
give an LED current of about 1.7mA which should suffice for most general purpose LEDs.

#### Produced Events

The 4 switches that produce events are connecte to pins A0, A1, A2 & A3. These inputs are
active low.  An Internal pull-up resistor in the Arduino will hold these inputs at +5V. The
switch should connect a pin to 0V and when closed it will initiate an event.

### CBUS Op Codes

The following Op codes are supported:

OP_CODE | HEX | Function
----------|---------|---------
 OPC_ACON | 0x90 | On event
 OPC_ACOF | 0x91 | Off event
 OPC_ASON | 0x98 | Short event on
 OPC_ASOF | 0x99 | Short event off

### Event Variables

Event Variables control the action to take when an event is received.
The number of Event Variables (EV) is equal to the number of LEDs.

Event Variable 1 (EV1) controls the first LED pin in the ```LED``` array. 
EV2 controls the second LED pin, etc.

The LEDs are controlled by the LEDControl class.  This allows for any LED to be
switched on, switched off or flashed at a rate determined in the call:
LEDControl::flash(unsigned int period)

The following EV values are defined to control the LEDs:

 EV Value | Function
--------|-----------
 0 | LED off
 1 | LED on
 2 | LED flash at 500mS
 3 | LED flash at 250mS
 
### Node Variables

Node Variables control the action to take when a switch is pressed or depressed.

The number of Node Variables (NV) is equal to the number of switches.
NV1 corresponds to the first pin defined in the array ```SWITCH```, 
NV2 corresponds to the second pin in that array, etc.

The following NV values define input switch function:

NV Value | Function
--------|--------
 0 | On/Off switch
 1 | On only push button
 2 | Off only push button
 3 | On/Off toggle push button

## The Serial Monitor

If your Arduino is connected to the Arduino IDE on your computer via the USB port, you can
access information by sending a character from the IDE.  The function of these characters
is as follows:

### 'r'
This character will cause the module to renegotiate its CBUS status by requesting a node number.
The FCU will respond as it would for any other unrecognised module.

### 'z'
This character needs to be sent twice within 2 seconds so that its action is confirmed.
This will reset the module and clear the EEPROM.  It should thus be used with care.

Other information is available using the serial monitor using other commands:

### 'n'
This character will return the node configuration.

### 'e'
This character will return the learned event table in the EEPROM.

### 'v'
This character will return the node variables.

### 'c'
This character will return the CAN bus status.

### 'h'
This character will return the event hash table.

### 'y'
This character will reset the CAN bus and CBUS message processing.

### '\*'
This character will reboot the module.

### 'm'
This character will return the amount of free memory. 
 
 
 
 
 

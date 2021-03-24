# The DBuno
Using the AVR128DB32 on an Arduino UNO Form Factor Board

I think Microchip’s new AVR128DBxx MCU is a large evolutionary step for the basic AVR® product line.  I’ve been using this device for several months and, although the ATmega328p has been a great part for the original UNO, the AVR-DB is a big step forward.  So much so, that I want to design new projects for the AVR-DB and not the 328p.

When I started with Arduino I expected that I’d be using the ProMini most frequently, because of it’s small form factor.  I assumed that I’d be using it as a module on some larger board that consisted of I/O glue.  Instead, I found that I was mostly using UNOs with a perf-board shield (daughter board) mounted on top.  I used M2.5x6 pan head screws into M2.5x12 standoffs to mount the two board sandwich into whatever housing I was using.  That worked well for me and I’d like to continue with that going forward.  Yes, there are some glitches in the UNO form factor – tight clearance for some of the screw holes, 50mil offset (instead of 100mil) between the connectors for D7 and D8.  But it is what it is, and there are too many proto-boards and manufactured PCB shields that make use of the UNO form factor, and that can’t be ignored.

So, the question becomes, if a typical UNO clone was modified by removing the ATmega328p (32 pin package) and replaced with an AVR128DB32 (32 pin package) to become a DBuno, what should be the assignments from MCU ports to the signal connectors at the edge of the DBuno board?

I’ve given a fair amount of thought to what the mappings should be from the MCU pin ports to the DBuno's edge pins (A0-A5, D0-D13).  In doing so, I considered the following assumptions and goals:

1.  It’s more difficult to change a printed circuit board shield (daughter board) than it is to change firmware, so the firmware should accommodate the existing shield pin assignments.

2.  Given that this “DBuno” board would likely be used with existing shields, then the basic special functions of the Arduino UNO (UART, SPI, I2C) need to be provided on the same edge pins for the DBuno.  This might be done with primary or alternate pins, but those port functions must be provided on the same pins as used on the original UNO.  Specifically,  (a) a USART needs to be provided at D0 and D1, (b) an SPI port needs to be provided at D10, D11, D12 and D13, and (c) an I2C port needs to be provided at SCL and SDA, and finally, if possible, (d) provide an I2C port on A4 and A5.

3.  Many of my projects require clock signals for timers with 100 ppm accuracy, which can only be achieved with a crystal, so I’m going to set reserve PA0 and PA1 for a 24 MHz crystal and load capacitors, and not bring those signals to the edge connector.  The crystal does not need to be populated for those who do not want the additional expense of a crystal, but the board layout will provide for the crystal.  Although the PF0 and PF1 leads are not connected to the DBuno edge connectors, these signals should be available on a two pin header for use in those cases where the crystal is not populated.

4.  One of the big attractions for me to the AVR-DB line (as well as the tinyAVR® 1-series) is the hardware DAC (Digital to Analog Converter).  So I will ensure that the DAC is available.

5.  I also want to ensure that all available PCx pins are aviable for use since they are powered from VDDIO2 and allow for different port voltage than that used for system voltage.

In considering the pin assignments, one factor to keep in mind is that, with the exception of PF6 (RESET), all port pins can function as GPIOs.  So just because the assignment list doesn’t say GPIO, any and all port pins (excluding PF6) can be used for GPIO.

## DB32 - Configuration for the 32 Pin Part

So, as currently configured, this assignment table assings MCU port pins to DBuno edge pins as follows:

USART-0 (ALT) (PA5, PA4) appears on DBuno edge pins D0 and D1 respectively, placing a USART for the DBuno on the same pins as the original UNO.

SPI-1 (PC3, PC0, PC1, PC2) appears on DBuno edge pins D10, D11, D12 and D13 respectively, placing an SPI port for the DBuno on the same pins as the original UNO.

I2C-0 (PA2, PA3) appears on the DBuno edge pins SDA and SC respectively, placing an I2C port for the DBuno on the same pins as the original UNO.


Additional MCU pin port assignments are as follows:

The Vref port (PD7) is provided on DBuno edge pin A0.

PD1, PD2 and PD3 are assigned DBuno edge pins A1, A2 and A3 respectively, to provide access to OpAmp-0 or CCL-LUT-2.

I2C-1 (PF2, PF3) appears on DBuno edge pins A4 and A5 respectively, placing an I2C port for the DBuno on the same pins as the original UNO (note that on the original UNO the SDA and SCL pins were added as a later REV of the board, but they were cross-connected to A4 and A5 on the original UNO - so one I2C port was presented on two different sets of edge pins.  On the DBuno pins SDA/SCL edge pins are independent of pins A4/A5 edge pins and use two independent I2C ports).

Note that given the assignments above DBuno edge pins A0, A1, A2, A3, A4 and A5 all are capable of serving as ADC inputs, as they were on the original UNO.

A DAC (PD6) is provided on DBuno edge pin D7.

The XDIR-0(ALT) (PA7) pin for USART-0(ALT) at D0/D1 appears on DBuno edge pin D3, to facilitate RS-485 communications with this UART.  XCK-0(ALT) for the same USART appears on DBuno edge pin D2.

USART-2  RX/TX (PF1/PF0) appear on D4/D5 respectively.  This is also used for the 32 KHz cyrstal and loading capacitors, which should be provided for on the board laout but not populate.

If the SPI-1 port at D10-D13 is not used by firmware, the firmware can assign those pins to USART-1 (TX, RX, DIR, CK).  Similarly, if the USART-0(ALT) pins are not used as a USART they can be used for SPI-0.

At this point all DBuno edge pins have been assigned except D6, and the only AVR-DB32 port pins that have not been assigned are PD4 and PD5.  In reviewing the I/O Multiplexing table in section 3.1 of the AVR128DB28 data sheet, it’s not clear that one of these pins is more “useful” than the other, therefore I will arbitrarily assign PD4 to DBuno edge pin PD4 and PD5 is unused.

With those assignments complete, we have a system that is much more flexible than one based on the ATmega328p.  Not all functions are available in all situations.  Even though many of the pins on the AVR-DB serve several functions, they can only serve one function at a time.

With these pin assignments on the DB32 part, there are three primary port configurations available.  They are:


2 USARTS, 1 SPI, 2 I2C
3 USARTS, no SPI, 2 I2C
1 USART, 2 SPI, 2 I2C

In simplified table form these assignments for the DB32 are:

~~~text
AVR-DB32    DBuno Edge
Port Pins   Pin Lables  Available Functions - Notes
---------  ----------   -------------------------------
  PA0        n/a       24 MHz Xtal - pin 1 of 2 pin header
  PA1        n/a       24 MHz Xtal - pin 2 of 2 pin header
  PA2        SDA       0-SDA
  PA3        SDL       0-SDL
  PA4        D1        0-USART-RX(ALT)    0-MISO
  PA5        D0        0-USART-TX(ALT)    0-MOSI
  PA6        D2        0-USART-XCK(ALT)   0-SCK
  PA7        D3        0-USART-XDIR(ALT)  0-SS*
  
  PC0        D11       1-USART-TX    1-MOSI  (VDDIO2)
  PC1        D12       1-USART-RX    1-MISI  (VDDIO2)
  PC2        D13       1-USART-XCK   1-SCK   (VDDIO2)
  PC3        D10       1-USART-XDIR  1-SS*   (VDDIO2)
  
  PD0        not present on DB32
  PD1        A1        AIN1   OP0,INP   LUT2,IN1
  PD2        A2        AIN2   OP0,OUT   LUT2,IN2
  PD3        A3        AIN3   OP0,INN   LUT2,OUT
  PD4        D6        AIN4   
  PD5        unused
  PD6        D7        DAC    AIN6      LUT2,OUT(ALT)
  PD7        A0        VREFA  AIN7
  
  PF0        D4        2-USART-TX      32K-XTAL - D4 via open solder bridge
  PF1        D5        2-USART-RX      32K-XTAL - D5 via open solder bridge
  PF2        A4        2-USART-XCK     1-SDA
  PF3        A5        2-USART-XDIR    1-SCL
  PF4        D8        2-USART-TX(ALT)   AIN20
  PF5        D9        2-USART-RX(ALT)   AIN21
  PF6        RESET*
  
~~~

## DB28 - Configuration for the 28 Pin DIP Part

In the case of the 28 pin DIP part (thank you Microchip for including a DIP package in the mix for those who have difficulty soldering surfacemount devices), we loose PF2, PF3 (access to I2C-1) and PF4 and PF5 (access to USART-2), which leaves open DBuno edge pins A4, A5, D8 and D9.

So for a ARV-DB28 pin assignment, the external HF oscillators pins PA0 and P1 will be brought out to D8 and D9 respectively through solder bridge jumpers (closed if crystal is not populated, open if crystal is populated) and PD5 that was abandoned in the 32 pin configuration will be connected to DBuno edge pin A4 through a solder bride (normally closed).  DBuno edge pin A5 will be left open-circuit.  With DBuno edge pins A5 isolcated, and A4 capabile of being isolated, when needed the A4/A5 edge pins may be cross connected by hand wiring to edge pins SDA/SCL via small gauge wire jumpers.

With these pin assignments on the DB28 part, there are three primary port configurations available.  They are:

1 USARTs, 1 SPI, 1 I2C
2 USARTs, no SPI, 1 I2C
no USARTs, 2 SPI, 1 I2C

In simplified table form these assignments for the DB28 DIP are:

~~~text
AVR-DB28    DBuno Edge
Port Pins   Pin Lables  Available Functions - Notes
---------  ----------   -------------------------------
  PA0        D8        24 MHz Xtal - D8 via open solder bridge - 
  PA1        D9        24 MHz Xtal - D9 via open solder bridge 
  PA2        SDA       0-SDA
  PA3        SDL       0-SDL
  PA4        D1        0-USART-RX(ALT)    0-MISO
  PA5        D0        0-USART-TX(ALT)    0-MOSI
  PA6        D2        0-USART-XCK(ALT)   0-SCK
  PA7        D3        0-USART-XDIR(ALT)  0-SS*
  
  PC0        D11       1-USART-TX    1-MOSI  (VDDIO2)
  PC1        D12       1-USART-RX    1-MISI  (VDDIO2)
  PC2        D13       1-USART-XCK   1-SCK   (VDDIO2)
  PC3        D10       1-USART-XDIR  1-SS*   (VDDIO2)
  
  PD0        not present on DB28
  PD1        A1        AIN1   OP0,INP   LUT2,IN1
  PD2        A2        AIN2   OP0,OUT   LUT2,IN2
  PD3        A3        AIN3   OP0,INN   LUT2,OUT
  PD4        D6        AIN4   
  PD5        unused
  PD6        D7        DAC    AIN6      LUT2,OUT(ALT)
  PD7        A0        VREFA  AIN7
  
  PF0        D4        2-USART-TX      32K-XTAL - D4 via open solder bridge
  PF1        D5        2-USART-RX      32K-XTAL - D5 via open solder bridge
  PF2                  n/a
  PF3                  n/a
  PF4                  n/a
  PF5                  n/a
  PF6                  RESET*
  
             A5 is isolated, no connection
~~~

## UPDI vs The Serial Port - Programming and Debugging

I’d say that 50% of my projects use MPLAB X IDE (free) with SNAP ($25) to program and debug AVR systems.  The other 50% of my projects use the Arudino IDE (free), pyupdi (free) and a USB<->Serial module ($2).  Using MPLAB X gives the programmer a lot of freedom, but with that freedom comes a lot of opportunities to “get stuck”.  For people new to MCU coding, I strongly recommend the simplicity and shallow learning curve of the Arduino IDE.  One downside of using Arduino IDE is that debugging tools are limited to printf() statements and toggling an LED.  Life is compromise.

With the advent of of the pyupdi utility and UPDI on AVR MCUs, I’ve lost interest in using Serial Download to program my application code into the AVR.  I still like having a USB serial port on development boards, but I don’t consider it essential.  It’s easy enough to add an outboard USB<->Serial converter to a system, as one would do with the original ProMini, if a USB serial port is needed.  If cost and space are a consideration, I’d have no problem if the USB<->Serial function was removed from the DBuno.  However, if the USB<->Serial is retained on the DBuno, I’d recommend several considerations:

1.  Retain the 1K resistor as found on the original UNO from the TTL serial output of the USB<->Serial to D0 (UART RX input) as this allows WIRE-ORing other USB<->Serial converters (the current (2019) device driver for the CH340 parts does not support XonXoff flow control – the only viable solution is to wire in a different USB<->Serial IC such as the SiLabs CP2102 as the W10 driver for this IC does support both XonXoff and RTS/CTS flow control).

2.  Provide a solder bridge jumper (normally open) between the CTS* of the USB<->Serial chip and PC3:D10 as this would allow the user’s application code, via PC3, to provide RTS/CTS flow control (the RTS signal may be safely ignored as the host is very unlikely to be overrun by data from the DBuno).

3.  Allow the USB<->Serial to be configured by solder bridge jumpers for normal connection to D0, D1 *or* have a solder bridge jumper from D0 to UPDI and a solder bridge jumper that connects D0 to D1 through a 4.7 K ohm resistor, to allow the USB<->Serial to perform programming via utility pyupdi.


## VDDIO2

Finally, there's the question of VDDIO2, the alternate power rail for the PCx ports.  VDDIO2 is a strong feature of the AVR-DB products because it allows the PCx ports to operate at a different voltage than the main MCU voltage.  There is no requirement that VDDIO2 be have a higher or lower voltage and VDD.

One solution is to tie VDDIO2 to VDD and call it done.  Nothing lost relative to the original UNO.  But I'd rather see VDDIO2 separate from VDD, with it's own filter caps (1 uF and 10 uF) connected to a 2 pin pinpost header.  Additionally, provide a normally closed solder bridge bewteen VDD and VDDIO2.

<p align="center">
---ooOoo---
</p>



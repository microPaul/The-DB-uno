# The DB-uno

I now have constructed several DB-UNO prototype boards designed with KiCad using 1206 components.  I'm up to Rev 4, which doesn't say much for my ability to get it right the first time.  A lot of the info below is out of date.  I need to correct that info as well as post the latest schematic, assembly drawing, parts list, gerber zip and the KiCad design files.  I hope to do that by April 15.  Note that some of the pin assignments outlined below will change.

------------------------

Using the AVR128DB32 on an Arduino UNO Form Factor Board

I think Microchip’s new AVR128DBxx MCU is a large evolutionary step for the basic AVR® product line.  I’ve been using this device for several months, with both MPLAB X IDE with SNAP and the Arduino IDE using Spence Konde's DXcore (see https://github.com/SpenceKonde/DxCore ) and pyMCUprog style programming (see https://github.com/SpenceKonde/DxCore#from-a-usb-serial-adapter-pyupdi-style ).  Although the ATmega328p has been a great part for the original UNO, the AVR-DB is a big step forward.  So much so, that I want to design new projects for the AVR-DB and not the 328p.

When I started working with Arduino I expected that I’d be using the ProMini most frequently, because of it’s small form factor.  I assumed that I’d be using it as a module on some larger board that hosted a lot of I/O glue.  Instead, I found that I was mostly using UNOs with a perf-board shield (daughter board) mounted on top.  I used M2.5x6 pan head screws into M2.5x12 standoffs to mount the two (sometimes three) board sandwich into whatever enclosure I was using.  That worked well for me and I’d like to continue with that going forward with the AVR-DB.  Yes, there are some glitches in the UNO form factor – tight clearance for some of the screw holes, 150mil offset (instead of 200mil) between the connectors for D7 and D8.  But it is what it is.  And as it is, there are too many proto-boards and too many manufactured PCB shields that make use of the UNO form factor, and those facts can’t be ignored.

So, the question becomes, if a typical UNO clone was modified by removing the ATmega328p (32 pin package) and replaced with an AVR128DB32 (32 pin package) to become a DB-uno, what should be the assignments from MCU port pins to the signal connectors at the edge of the DB-uno board?  Spence has made assignments from AVR-DB port pins to pin numbers for his DXcore (see: https://github.com/SpenceKonde/DxCore/blob/master/megaavr/extras/DA28.md ) and the lables for the board edge connectos could be changed to match those pin numbers, but I think I would prefer to have pin labeling like that found on the Curiosity Nano, that being PA0, PA1, etc., rather than the D0-D13 and A0-A5 as found on the original UNO.

I’ve given a fair amount of thought to what the mappings should be from the MCU pin ports to the DB-uno's edge pins (A0-A5, D0-D13).  In doing so, I considered the following assumptions and goals which affect that mapping:

1.  It’s more difficult to change a printed circuit board shield (daughter board) than it is to change firmware, so the firmware should accommodate the existing shield pin assignments.

2.  Given that this “DB-uno” board would likely be used with existing shields, then the basic special functions of the Arduino UNO (UART, SPI, I2C) need to be provided on the same edge pins for the DB-uno.  This might be done with primary or alternate pins, but those port functions must be provided on the same pins as used on the original UNO.  Specifically,  (a) a USART needs to be provided at D0 and D1, (b) an SPI port needs to be provided at D10, D11, D12 and D13, and (c) an I2C port needs to be provided at SCL and SDA, and finally, if possible, (d) provide an I2C port on A4 and A5.

3.  Many of my projects require clock signals for timers with 100 ppm accuracy, which can only be achieved with a crystal, so I’m going to set reserve PA0 and PA1 for a 24 MHz crystal and load capacitors, and not bring those signals to the edge connector.  The crystal does not need to be populated for those who do not want the additional expense of a crystal, but the board layout will provide for the crystal.  Although the PF0 and PF1 leads are not connected to the DB-uno edge connectors, these signals should be available on a two pin header for use in those cases where the crystal is not populated.

4.  One of the big attractions for me to the AVR-DB line (as well as the tinyAVR® 1-series) is the hardware DAC (Digital to Analog Converter).  So I will ensure that the DAC is available.

5.  I also want to ensure that all available PCx pins are aviable for use since they are powered from VDDIO2 and allow for different port voltage than that used for system voltage.

6.  I understand that PWM signals are of importance to some people, but I was not able to meet my goals of item 2 above without giving a bit of short shrift toward PWM.  PWM signals are still available on the board edge pins, but not any any organized manner.  They are where they landed.

In considering the pin assignments, one factor to keep in mind is that, with the exception of PF6 (RESET), all port pins can function as GPIOs.  So just because the assignment list doesn’t say GPIO, any and all port pins (excluding PF6) can be used for GPIO.


## DB32 - Configuration for the 32 Pin Part

So, as currently configured, this assignment table assings MCU port pins to DB-uno edge pins as follows:

USART-0 (ALT) (PA5, PA4) appears on DB-uno edge pins D0 and D1 respectively, placing a USART for the DB-uno on the same pins as the original UNO.

SPI-1 (PC3, PC0, PC1, PC2) appears on DB-uno edge pins D10, D11, D12 and D13 respectively, placing an SPI port (SPI-1) for the DB-uno on the same pins as the original UNO.  Note also that PCx pins are special in that they are powered from VDDIO2, and can have a different supply voltage than the main part of the microcontroller, avoiding voltage level shifting when operating a 3.3V peripheral from a 5V microcontroller (or vise versa). If the SPI-1 port at D10-D13 is not used by firmware, the firmware can assign those pins to USART-1 (TX, RX, DIR, CK).  Similarly, if the USART-0(ALT) pins are not used as a USART they can be used for SPI-0.  Again, all PCx ports can make use of internal level shifting.

I2C-0 (PA2, PA3) appears on the DB-uno edge pins SDA and SC respectively, placing an I2C port for the DB-uno on the same pins as the original UNO.


Additional MCU pin port assignments are as follows:

The Vref port (PD7) is provided on DB-uno edge pin A0.

PD1, PD2 and PD3 are assigned DB-uno edge pins A1, A2 and A3 respectively for ADC inputs, as well as access to OpAmp-0 or CCL-LUT-2.

I2C-1 (PF2, PF3) appears on DB-uno edge pins A4 and A5 respectively, placing an I2C port for the DB-uno on the same pins as the original UNO (note that on the original UNO the SDA and SCL pins were added as a later REV of the board, but they were cross-connected to A4 and A5 on the original UNO - so one I2C port was presented on two different sets of edge pins.  On the DB-uno pins SDA/SCL edge pins are independent of pins A4/A5 edge pins and use two independent I2C ports).

Note that given the assignments above DB-uno edge pins A0, A1, A2, A3, A4 and A5 all are capable of serving as ADC inputs, as they were on the original UNO.

A DAC (PD6) is provided on DB-uno edge pin D7.

The XDIR-0(ALT) (PA7) pin for USART-0(ALT) at D0/D1 appears on DB-uno edge pin D3, to facilitate RS-485 communications with this UART.  XCK-0(ALT) for the same USART appears on DB-uno edge pin D2.

USART-2  RX/TX (PF1/PF0) appear on D4/D5 respectively.  This is also used for the 32 KHz cyrstal and loading capacitors, which should be provided for on the board laout but not populated unless needed.



At this point all DB-uno edge pins have been assigned except D6, and the only AVR-DB32 port pins that have not been assigned are PD4 and PD5.  In reviewing the I/O Multiplexing table in section 3.1 of the AVR128DB28 data sheet, it’s not clear that one of these pins is more “useful” than the other, therefore I will arbitrarily assign PD4 to DB-uno edge pin PD4 and PD5 is unused.

With those assignments complete, we have a system that is much more flexible than one based on the ATmega328p.  Not all functions are available in all situations.  Even though many of the pins on the AVR-DB serve several functions, they can only serve one function at a time.

With these pin assignments on the DB32 part, there are three primary port configurations available.  They are:


2 USARTS, 1 SPI, 2 I2C
3 USARTS, no SPI, 2 I2C
1 USART, 2 SPI, 2 I2C

In simplified table form these assignments for the DB32 are:

~~~text
AVR-DB32    DB-uno Edge
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
  
  PC0        D11       1-USART-TX    1-MOSI              (VDDIO2)
  PC1        D12       1-USART-RX    1-MISI              (VDDIO2)
  PC2        D13       1-USART-XCK   1-SCK   0-SDA(ALT)  (VDDIO2)
  PC3        D10       1-USART-XDIR  1-SS*   0-SCL(ALT)  (VDDIO2)
  
  PD0        not present on DB32
  PD1        A1        AIN1   OP0,INP   LUT2,IN1
  PD2        A2        AIN2   OP0,OUT   LUT2,IN2
  PD3        A3        AIN3   OP0,INN   LUT2,OUT
  PD4        D6        AIN4   
  PD5        unused
  PD6        D7        DAC    AIN6      LUT2,OUT(ALT)
  PD7        A0        VREFA  AIN7
  
  PF0        D4        2-USART-TX        32K-XTAL - D4 via open solder bridge
  PF1        D5        2-USART-RX        32K-XTAL - D5 via open solder bridge
  PF2        A4        2-USART-XCK       1-SDA
  PF3        A5        2-USART-XDIR      1-SCL
  PF4        D8        2-USART-TX(ALT)   AIN20
  PF5        D9        2-USART-RX(ALT)   AIN21
  PF6        RESET*
~~~


## DB28 - Configuration for the 28 Pin DIP Part

In the case of the 28 pin DIP part (thank you Microchip for including a DIP package in the mix for those who have difficulty soldering surface mount devices), we loose PF2, PF3 (access to I2C-1) and PF4 and PF5 (access to USART-2), which leaves open DB-uno edge pins A4, A5, D8 and D9.

So for a ARV-DB28 pin assignment, the external HF oscillators pins PA0 and P1 will be brought out to D8 and D9 respectively through solder bridge jumpers (closed if crystal is not populated, open if crystal is populated) and PD5 that was abandoned in the 32 pin configuration will be connected to DB-uno edge pin A4 through a solder bride (normally closed).  DB-uno edge pin A5 will be left open-circuit.  With DB-uno edge pins A5 isolcated, and A4 capabile of being isolated, when needed the A4/A5 edge pins may be cross connected by hand wiring to edge pins SDA/SCL via small gauge wire jumpers.

With these pin assignments on the DB28 part, there are three primary port configurations available.  They are:

1 USARTs, 1 SPI, 1 I2C
2 USARTs, no SPI, 1 I2C
no USARTs, 2 SPI, 1 I2C

In simplified table form these assignments for the DB28 DIP are:

~~~text
AVR-DB28    DB-uno Edge
Port Pins   Pin Lables  Available Functions - Notes
---------  ----------   -------------------------------
  PA0        D8        24 MHz Xtal - D8 via open solder bridge
  PA1        D9        24 MHz Xtal - D9 via open solder bridge
  PA2        SDA       0-SDA
  PA3        SDL       0-SDL
  PA4        D1        0-USART-RX(ALT)    0-MISO
  PA5        D0        0-USART-TX(ALT)    0-MOSI
  PA6        D2        0-USART-XCK(ALT)   0-SCK
  PA7        D3        0-USART-XDIR(ALT)  0-SS*
  
  PC0        D11       1-USART-TX    1-MOSI              (VDDIO2)
  PC1        D12       1-USART-RX    1-MISI              (VDDIO2)
  PC2        D13       1-USART-XCK   1-SCK   0-SDA(ALT)  (VDDIO2)
  PC3        D10       1-USART-XDIR  1-SS*   0-SCL(ALT)  (VDDIO2)
  
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


## Other Considerations

OK, we've gone that far, but what else might be considering for a DB-uno board?  These proposed changes and enhancements are just that, proposed.  Nothing is required, but all are worth of some consideration.


### UPDI vs The Serial Port - Programming and Debugging

I’d say that for about half my AVR-DB projects I use MPLAB X IDE with SNAP to program and debug AVR systems.  For the othe half I use the Arudino IDE using Spence Konde's DXcore (see https://github.com/SpenceKonde/DxCore ) and pyMCUprog style programming (see https://github.com/SpenceKonde/DxCore#from-a-usb-serial-adapter-pyupdi-style ).  Using MPLAB X gives the developer a lot of freedom, control and diagnostic visibility, but with that freedom and control comes a lot of opportunities to “get stuck”.  For people new to MCU coding, I strongly recommend the simplicity and shallow learning curve of the Arduino IDE.  One downside of using Arduino IDE is that debugging tools are primarily limited to printf() statements and toggling an LED.  Life is compromise.

With the advent of of the pyMCUprog programming for AVR MCUs, I’ve lost interest in using Serial Bootloader Download to program my application code into the AVR.  I still like having a USB serial port on development boards, but I don’t consider it essential.  It’s easy enough to add an outboard USB<->Serial converter to a system, as one would do with the original ProMini, if a USB serial port is needed.  If cost and space are a consideration, I’d have no problem if the USB<->Serial function was removed from the DB-uno.  However, if the USB<->Serial is retained on the DB-uno, I’d recommend several considerations:

1.  Retain the 1K resistor as found on the original UNO from the TTL serial output of the USB<->Serial to D0 (UART RX input) as this allows WIRE-ORing paralleling by means of an open collector/drain buffer from a second USB<->Serial converter.  Why might one want to do this?  To provide a means for XonXoff flow control.  The current (2019) device driver for the CH340 parts does not support XonXoff flow control – the only viable solution is to wire in a different USB<->Serial IC such as the SiLabs CP2102, as the W10 driver for the CP2102 supports both XonXoff and RTS/CTS flow control.

2.  Provide a solder bridge jumper (normally open) between the CTS* of the USB<->Serial chip and PC3:D10 as this would allow the user’s application code, via PC3, to provide RTS/CTS flow control (the RTS signal may be safely ignored as the host is very unlikely to be overrun by data from the DB-uno).

3.  Allow the USB<->Serial to be configured by solder bridge jumpers for normal connection to D0, D1 *or* to use as a pyMCUprog programmer by providing a solder bridge jumper that connects D0 to D1 through a 4.7 K ohm resistor, and then D0 to UPDI, to allow the USB<->Serial to perform programming via utility pyMCUprog.


### VDD

To facilitate operation from batteries and use of sleep mode, provide a solder bridge jumper (closed) between the output of any voltage regulator or USB power source and VDD, so that the user can easily isolate VDD from all other power sources.


### VDDIO2

Then there's the question of VDDIO2, the alternate power rail for the PCx ports.  VDDIO2 is a strong feature of the AVR-DB products because it allows the PCx ports to operate at a different voltage than the main MCU voltage.  There is no requirement that VDDIO2 be have a higher or lower voltage than VDD.

One solution is to tie VDDIO2 to VDD and call it done.  Nothing lost relative to the original UNO.  But I'd rather see VDDIO2 separate from VDD, with it's own filter caps (1 uF and 10 uF) connected to a 2 pin pinpost header for power input.  Additionally, provide a normally closed solder bridge between VDD and VDDIO2 for when an external voltage source is not provided for VDDIO2.


### Reallocating the Arduino Uno ISP Six Pin Header Area

*It's been pointed out to me that the ISP SPI 2x3 header on the Arduino UNO is often used by sheilds for SPI access and not D10-D13, since the Mega does not use D10-D13 for SPI.  So what I have stated below needs to be given more thought -- I might not change it, but one needs to realize that removing the 2x3 header it might have a negative impact for some existing shields.*

Because the AVR-DB parts use UPDI for programming and debug, there's no reason to retain the 2x3 pin header for SPI ISP programming.  That board area can be reallocated to a 3 pinpost header for UPDI as well as a 2 pinpost header for VDDOI2.


<p align="center">
---ooOoo---
</p>



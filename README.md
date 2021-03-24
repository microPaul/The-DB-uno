# The DBuno
Using the AVR128DB32 on an Arduino UNO Form Factor Board

I think Microchip’s new AVR128DBxx MCU is a large evolutionary step for the basic AVR® product line.  I’ve been using this device for several months and, although the ATmega328p has been a great part for the original UNO, the AVR-DB is a big step forward.  So much so, that I want to design new projects for the AVR-DB and not the 328p.

When I started with Arduino I expected that I’d be using the ProMini most frequently, because of it’s small form factor.  I assumed that I’d be using it as a module on some larger board that consisted of I/O glue.  Instead, I found that I was mostly using UNOs with a perf-board shield (daughter board) mounted on top.  I used M2.5x6 pan head screws into M2.5x12 standoffs to mount the two board sandwich into whatever housing I was using.  That worked well for me and I’d like to continue with that going forward.  Yes, there are some glitches in the UNO form factor – tight clearance for some of the screw holes, 50mil offset (instead of 100mil) between the connectors for D7 and D8.  But it is what it is, and there are too many proto-boards and manufactured PCB shields that make use of the UNO form factor, and that can’t be ignored.

So, the question becomes, if a typical UNO clone was modified by removing the ATmega328p (32 pin package) and replaced with an AVR128DB32 (32 pin package) to become a DBuno, what should be the assignments from MCU ports to the signal connectors at the edge of the DBuno board?

I’ve given a fair amount of thought to have and have created a spread sheet (link at bottom) showing how I think that should be done (I'm also working on a mapping for the ATtiny3216 but that's only for prototyping purposes and is not yet complete).  In doing so, I considered the following assumptions and goals:

1.	It’s more difficult to change a printed circuit board shield (daughter board) than it is firmware, so the firmware should accommodate the existing shields.

2.	Given that this “DBuno” board would likely be used with existing shields, then the basic special functions of the Arduino UNO (UART, SPI, I2C) need to be provided on the same edge pins for the DBuno.  This might be done with primary or alternate pins, but those port functions must be provided on the same pins as used on the original UNO.  Specifically,  (a) a USART needs to be provided at D0 and D1, (b) a SPI port needs to be provided at D10, D11, D12 and D13, and (c) an I2C port needs to be provided at SCL and SDA, and finally, if possible, (d) provide I2C port on A4 and A5.

3.	Many of my projects require clock signals for timers with 100 ppm accuracy, which can only be achieved with a crystal, so I’m going to set reserve PA0 and PA1 for a 24 MHz crystal and load capacitors, and not bring those signals to the edge connector.  The crystal does not need to be populated for those who do not need 100 ppm and do not want the additional expense of a crystal, but the board layout will provide for the crystal.  Although the PF0 and PF1 leads are not connected to the DBuno edge connectors, these signals should be available on a two pin header for use in those cases where the crystal is not populated.

4.	One of the big attractions for me to the AVR-DB line (as well as the tinyAVR® 1-series) is the hardware DAC (Digital to Analog Converter).  So I will ensure that the DAC is available.

5.	I also want to ensure that all available PCx pins are aviable for use since they are powered from VDDIO2 and allow for different port voltage than that used for system voltage.

In considering the pin assignments, one factor to keep in mind is that, with the exception of PF6 (RESET), all port pins can function as GPIOs.  So just because the assignment list doesn’t say GPIO, any and all port pins (excluding PF6) can be used for GPIO.

## DB32 - Configuration for the 32 Pin Part

So, as currently configured, this assignment table does the following:

USART-0 (ALT) (PA5, PA4) appears on DBuno edge pins D0 and D1 respectively, placing a USART for the DBuno on the same pins as the original UNO.

SPI-1 (PC3, PC0, PC1, PC2) appears on DBuno edge pins D10, D11, D12 and D13 respectively, placing an SPI port for the DBuno on the same pins as the original UNO.

I2C-0 (PA2, PA3) appears on the DBuno edge pins SDA and SC respectively, placing an I2C port for the DBuno on the same pins as the original UNO.


Additional pin port function assignments are as follows:

The Vref port (PD7) is provided on DBuno edge pin A0.

PD1, PD2 and PD3 are assigned DBuno edge pins A1, A2 and A3 respectively, to provide access to OpAmp-0 or CCL-LUT-2.

I2C-1 (PF2, PF3) appears on DBuno edge pins A4 and A5 respectively, placing an I2C port for the DBuno on the same pins as the original UNO (note that on the original UNO the SDA and SCL pins were added as a later REV of the board, but they were cross-connected to A4 and A5 on the original UNO - so one I2C port was presented on two different sets of edge pins.  On the DBuno pins SDA/SCL edge pins are independent of pins A4/A5 edge pins and use two independent I2C ports).

Note that given the assignments above DBuno edge pins A0, A1, A2, A3, A4 and A5 all are capable of serving as ADC inputs, as they were on the original UNO.

A DAC (PD6) is provided on DBuno edge pin D7.

The XDIR-0(ALT) (PA7) pin for USART-0(ALT) at D0/D1 appears on DBuno edge pin D3, to facilitate RS-485 communications with this UART.  XCK-0(ALT) for the same USART appears on DBuno edge pin D2.

USART-2  RX/TX (PF1/PF0) appear on D4/D5 respectively.

If the SPI-1 port at D10-D13 is not used by firmware, the firmware can assign these pins to USART-1 (TX, RX, DIR, CK).  Similarly, if the USART-0(ALT) pins are not used as a USART they can be used for SPI-0.

At this point all DBuno edge pins have been assigned except D6, and the only AVR-DB32 port pins that have not been assigned are PD4 and PD5.  In reviewing the I/O Multiplexing table in section 3.1 of the AVR128DB28 data sheet, it’s not clear that one of these pins is more “useful” than the other, therefore I will arbitrarily assign PD4 to DBuno edge pin PD4.

With those assignments complete, we have a system that is much more flexible than one based on the ATmega328p.  Not all functions are available in all situations.  Even though many of the pins on the AVR-DB serve several functions, they can only serve one function at a time.

With these pin assignments on the DB32 part, there are three primary port configurations available.  They are:

2 USARTS, 1 SPI, 2 I2C
3 USARTS, no SPI, 2 I2C
1 USART, 2 SPI, 2 I2C

## DB28 - Configuration for the 28 Pin DIP Part

In the case of the 28 pin DIP part (thank you Microchip for including a DIP package in the mix for those who have difficulty soldering surfacemount devices), we loose PF2, PF3 (access to I2C-1) and PF4 and PF5 (access to USART-2), which leaves open DBuno edge pins A4, A5, D8 and D9.

So for a ARV-DB28 pin assignment, the external HF oscillators pins PA0 and P1 will be brought out to D8 and D9 respectively through solder bridge jumpers (closed if crystal is not populated, open if crystal is populated) and PD5 that was abandoned in the 32 pin configuration will be connected to DBuno edge pin A4 through a solder bride (normally closed).  DBuno edge pin A5 will be left open circuit.  With DBuno edge pins A5 open, and A4 capabile of being isolated, when needed the A4/A5 edge pins may be cross connected to SDA/SCL via small gauge wire jumpers.

With these pin assignments on the DB28 part, there are three primary port configurations available.  They are:

1 USARTs, 1 SPI, 1 I2C
2 USARTs, no SPI, 1 I2C
no USARTs, 2 SPI, 1 I2C


These assignments can be seen as a whole on a spread sheet at this link (ignore for now the stuf about ATtiny3216)




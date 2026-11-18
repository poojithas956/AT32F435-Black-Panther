---
title: "AT32F435 - Black Panther"
github: "https://github.com/poojithas956/AT32F435-Black-Panther"
description: "A custom AT32F435 BareBone Board with all required pins exposed."
created_at: "2026-11-16"
total_time: "8h 24m"
---

# November 16, 2026: Brainstorming + Structuring Project folder

<p>This is my first AT32 Dev Board, I have seen a lot of STM32 Dev Boards but very few on the newer, better and cheaper Microcontroller AT32F435.

Before working on the schematics, I organized my project files accordingly as per the guidelines.</p>
<br>
![file-explorer](https://fabricate.hackclub-assets.com/63e0c0d0cd36081a50d0bc2b84b9e46d632c68b02087ebef2544a41bf3c1091a/image-png/image.png)
<br>
<p>So I went through a list of required features and choosing appropriate components.
<br>
I wanted my custom dev board to be different from those on market, something powerful yet compact, filled with sensors, adequate amount of GPIO pins.
<br>
Hence, I went around [LCSC](https://LCSC.com) going through the available components, and choose 
ARTERY AT32F435RGT7 [ Microcontroller ] 
ICM-42688-P [ for Gyroscope ]
BMP581 [ Barometric Pressure Sensor ]
TPS631000DRLR [ High-power density step-down/step-up converter with 1.5A output current ]
BQ24072RGTR [ 1.5-A USB-Friendly Li-ion Battery Charger and Power-Path Management IC ]
<br>
Also went about downloading their datasheets, and going through their pins and required decoupling, external crystal...

**Total time spent: 1h**

# November 17, 2026: Schematic: Session 1 ( gyro & barometer )

After going through Datasheets, I loaded the project onto KiCAD 10, and started by placing all the required components onto the drawing sheet. 
<br>
*I used easyeda2kicad to convert all the components footprint from EasyEDA to KiCAD.

<br>

![](https://fabricate.hackclub-assets.com/8c698e968455170c47902e363b9144e56535f399cb1b8f058f1857da4e51cfab/image-png/image.png)

<br><br>

<p>I started by drawing the Sensor schematic<p/>
- Route clean 3.3V power to both VDD and VDDIO with local decoupling capacitors (C6/C7 for bulk filtering and C1 for the digital I/O rail) to suppress high-frequency switching noise.
- Connect the high-speed SPI communication lines (ICM_SCK, ICM_MOSI, ICM_MISO, and ICM_CS) directly to the microcontroller or single-board computer.
- Tie down or route the hardware interrupt pins (ICM_INT1 and ICM_INT2) to handle high-frequency data-ready triggers without constantly polling the device. 

<br>

![](https://fabricate.hackclub-assets.com/c78a6d6fbd1eae75052301ab093f2737a92f4548099f74495a36ecc11119ec26/image-png/image.png)

<br><br>

<p>I continued to BMP581 Barometer from Bosch</p>
- Route clean 3.3V power to both VDD (pin 10) and VDDIO (pin 1) with local decoupling capacitors (C9 and C5, both 100nF) to suppress supply noise.   
- Configure the communication protocol for I²C by tying CSB (pin 6) high to +3V3 and grounding SDO (pin 5) to set the appropriate I²C device address.
- Connect the I²C communication lines (BMP_SCL and BMP_SDA) with 4.7K pull-up resistors (R4 and R5) tied to +3V3 to ensure proper signal integrity on the bus.
- Route the hardware interrupt line (BMP_INT from pin 7) to handle data-ready triggers efficiently.

<br>

![](https://fabricate.hackclub-assets.com/1edf732c8cf609b2626f647592d5f39ecab4118c5d207845b444d919df0a241d/image-png/image.png)

**Total time spent: 1h 6m**

# November 18, 2026: Schematic: Session 2 ( power management )

Once done with with both the critical Sensors, I went ahead with the Power Management
<br>
Actually for the last JOURNAL, I forgot to mention that I faced quite a issue with the SPI and I2C, as BMP581 supports both, but for my use case SPI communication proved to better theoretically.
<br>
Yep, now I had to do some more research before routing the power management, I don't want my MCU become a ticking time-bomb, So went through a couple of STM32 based projects
<br>
CREDITS:
@Kai for Guide on designing a custom Dev Board
@NotaRoombu for Cyberboard v2
<br>
Use their guides and projects are reference I went ahead with the power management section.
<br>

<h2>TPS631000 buck-boost converter</h2>

- Route input power to VIN (pin 4) with local input decoupling using capacitor C15 (22µF) tied to GND.   Connect an external power inductor (U5, 1µH) between switching pins LX1 (pin 3) and LX2 (pin 2) to handle energy transfer.
- Place output filter capacitor C16 (47µF) on the VOUT / VCC rail (pin 1) to stabilize the regulated output.
- Set the output voltage using a feedback resistor divider with R7 (511K) connected between +3V3 and FB (pin 8), and R8 (91K) connected between FB and GND.
- Tie the enable (EN, pin 5) and mode (MODE, pin 6) pins appropriately to enable the regulator and configure its operation mode.

<br> 

![](https://fabricate.hackclub-assets.com/c3d45a213562596a16b56829aacd64c58d6fa867367709cf371796ee3f3542f1/image-png/image.png)

<br>

<h2>BQ24072 battery charger IC to manage power paths and safely charge the battery! </h2>
- Route input power from VBUS to IN (pin 13) with local input decoupling using capacitor C20 (1µF).   Set the fast-charge current using resistor R11 (2.2K) connected to ISET (pin 16) and configure the input current limit using resistor R13 (3.57K) connected to ILIM (pin 12).
- Connect the battery to the BAT pins (pins 2 and 3) with local decoupling capacitor C19 (4.7µF) and temperature monitoring via the TS pin (pin 1) using resistor R10 (10K).
- Provide system output power through the OUT pins (pins 10 and 11) backed by output capacitor C21 (4.7µF) tied to VCC/VBUS.
- Configure status indication LEDs (LED2 for power good PGOOD on pin 7 and LED3 for charge status CHG on pin 9) with current-limiting resistors R14 (1K) and R15 (1K).  
 
<br>

![](https://fabricate.hackclub-assets.com/af5a66b4f87e53efb247bd8272da4cf321ec5167fe6831671c843f97b1f9043b/image-png/image.png)


<br>

I faced issue routing the Battery IC, specially the TS pin, I accidently connected it to +3v3 thinking its VBUS pin, but upon search on the Datasheet I found out its Temperature Sensor, so it regulated the speed of charging based on the temperature, however if you connect the TS directly to GND, the IC won't charge the battery thinking the battery is damaged or jus too hot / faulty temp. For bench testing, a 10K ohm resistor must be connected from the TS pin to GND, not R10, as R10 is already required by the IC to read the temp, so an external 10k ohm resistor connected in series with the R10 and GND, to enable charging. 

<br>

* Also ummmm i don't the journaling here is kinda weird, I need to add `<br>` every time I want a new line, can't even have space between para :/

**Total time spent: 1h 42m**

# November 18, 2026: Schematic: Session 3 ( final )

Gonna Lock-in and wrap up the schematics Today ;)
<br>
I continued drawing the schematic of the Microcontroller, First up I routed the external crystal as that is required for precise control and function, AT32F435 does have an internal crystal but isn't accurate enough for high precision control/functions. 
<br>
Added Decoupling Capacitors, to filter any high noise in the power chain, also to prevent brown outs during heavy processes, also preventing Brown-outs which might damage the components in the long run.
<br>

![](https://fabricate.hackclub-assets.com/88080363e496bcd62e5e3c02da4d0c2c34f1c0198f46d2065e08701a41fbb5e0/image-png/image.png)

<br>

After completing the Microcontroller, I finished other minor schematics like Pin outs, External USB pin out, status LEDs, BOOT0, BOOT1 & RESET. 

<br>

Complete Schematic is ready :P
<br>

![](https://fabricate.hackclub-assets.com/ebfd39811a8903590bbe33fc65af2b7f8fdf832a94de2b3167cfdc36e810922c/image-jpeg/schematic_page.jpg)

<br> 
Also got a sanity check, and it should be working yayyy...........

**Total time spent: 1h 30m**

# November 18, 2026: Routing PCB

I routed part of the schematic but was unable to continue further on 4 layer PCB, I was left with about 23 connection unrouted... My Aim was to make it as compact as possible, resembling Pico 2 or Black Pill Dev board...
<br>
So I switched to 6 layer layout 4 mil / 4mil config...
Using the 6 layers as
<br>
Signal
Ground
Power
Ground
Signal
<br>
I completed routing all 79 nets with 263 Pads, 142 Vias, 544 Track Segments.... Also spent some time giving the board a new look, also organized the silkscreen lowkey looks fire :D
<br>
![](https://fabricate.hackclub-assets.com/6ade2663f40c5dc82feb03036b8444d724fa6afe6ff08c92c6e1ddd7ffb40e21/image-png/image.png)
<br>

The end result looks amazing.... can't wait to have them fabricated and tested !!!!

![](https://fabricate.hackclub-assets.com/e65ad68f4afac5052f546e0bf28f273faec07c2f798773f91b25564a0a081576/image-png/Black%20Panther%20Back.png)
![](https://fabricate.hackclub-assets.com/dfad4bfc1dffef22bd069ab9f4d877d403a962b77cca3a4433b73f2214c7e2b6/image-png/Black%20Panther%20Front.png)

**Total time spent: 2h 24m**

# November 18, 2026: Inspection

Upon Running the DRC on PCB and ERC on Schematic, I got a couple or errors thank god those were pretty moderate to simple errors not like having your differential pairs not pairing up... Or a component mismatch... .
<br>
For the schematic, I got power related errors, upon searching on web I got a suggestion more like a solution to add Power_Flag next to the +3V3, GND, VSS lines as they were being treated separate from the main power symbols.
<br>
And for the PCB, routing was like a mess, don't know where to start, don't know where to end, Solved this by solving up the power channel, i.e. from the moment the power enters the PCB, I routed in that flow, so I was sort also able understand or more like get an idea on how the Board works.
<br>
![](https://fabricate.hackclub-assets.com/33f9627e6138bf522b1ba01c33cd40a13a6d284df5503f73cad23a9426f0d838/image-png/image.png)
![](https://fabricate.hackclub-assets.com/7557d0f77b4abea8c6ed37f94bcb63cf9accef04f2ee8ed81b077d5a9bb410a7/image-png/image.png)

**Total time spent: 42m**

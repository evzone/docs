# Peripherals
!!! info "Peripherals are sensors, displays, controllers, LEDs and other devices wired to available GPIO pins of your device"

!!! tip
    A peripheral must have correctly wired power, GND and data pins to the device prior to booting in order for Tasmota to detect it and initialize it properly.
    
## [Supported peripherals](Supported-Peripherals.md)

## Tasmota Settings
<img src="https://user-images.githubusercontent.com/5904370/68432161-2a154700-01b4-11ea-8ba9-adb7b717490d.png" style="float:right;height:15em;margin:10px 0">
Tasmota allows for easy selection of peripherals and assignment to GPIO pins. 

Configuration is possible in the webUI ***Configuration - Configure Module*** page

or by using commands: [`Module`](Commands.md#module) and [`GPIO`](Commands.md#gpio), or [`Template`](Commands.md#template).

### `Module`
First select desired module for the device (Wait for the restart). Depending on the type of [Module](Modules), only certain GPIO pins are user configurable. Module Generic (18) has all the GPIOs configurable.   

_[`Modules`](Commands.md#modules) shows supported modules_

### `GPIO`
Assign a [component](Components) to a GPIO.
   
- `GPIO14 2` configures sensor AM2301 to GPIO14_    
- `Backlog GPIO14 5; GPIO4 6` sets I<sup>2</sup>C SCL to GPIO14 and I<sup>2</sup>C SDA to GPIO4
   Tasmota will auto-detect all connected and supported I<sup>2</sup>C devices. If you have conflicting I<sup>2</sup>C addresses see [I2CDEVICES](I2CDEVICES)

_[`GPIOs All`](Commands.md#gpios) shows list of all available components by name and index_

**For a peripheral to show up you may need to power cycle your device instead of a soft restart.**

### `Template`
Instead of using `Module` and `GPIO` you can define everything using `Template`. [Read more...](Templates#template-configuration-with-commands)

## Additional Options

#### Measurement Units
Temperature units can be set to Celsius or Fahrenheit with [`SetOption8`](Commands.md#setoption8) command.

Pressure units can be set to hPa or mmHg with [`SetOption24`](Commands.md#setoption24) command. 
#### Update Interval
To change the update interval (TelePeriod) of MQTT messages change the [`TelePeriod`](Commands.md#teleperiod). Default interval is 300 seconds but can be set between 10 and 3600 seconds.    
`TelePeriod 10` will set the update interval to 10 seconds, so the sensor will update 6 times a minute.

#### Peripheral Specific
Some peripherals offer, or even require, additional commands. See [Sensor commands](Commands.md#sensors) page for peripheral specific commands.

!!! tip
    To make a link between the different naming schemes the [Pin Definition overview](https://github.com/esp8266/esp8266-wiki/wiki/Pin-definition) in the ESP8266 wiki is quite helpful.

## Examples

#### Connect switch
If you take a Sonoff Basic and connect a switch between pin4 (GND) and pin5 (GPIO14) of the 5 pin programming header you now have a second switch connected to the device. You can set this through the module config page as option `Switch1 (9)` or from the command line with `gpio14 9`.

See [Buttons and Switches](Buttons-and-Switches.md) for more information.

#### Connect jack
Instead of connecting a switch, you could connect a 4-pin 2.5mm jack, with the pins wired:  

|Jack|Pin|ESP8266|
|:-:|:-:|:-:|
tip|5 |GPIO14 
R1 | | no connection
R2 | 1 | GND
R3 | 4 | 3.3V

You can then plug a sensor into the jack like you would to a [Sonoff TH](devices/Sonoff-TH.md) and define what sensor you have connected to GPIO14.


## Restrictions  

!!! danger
     If you can avoid it, don't use GPIOs: **0, 1, 2, 6-11, 15 and 16**. That leaves **4, 5, 12, 13, 14** as GPIOs without any constraints. **3** being RX is also good to avoid (PWM is not working on this GPIO).

Others ***can*** be used but you have to mind the constraints outlined in [this document](https://tttapa.github.io/ESP8266/Chap04%20-%20Microcontroller.html).

### Voltage and Current
!!! danger
    The ESP8266 is a 3.3V microcontroller, so its I/O operates at 3.3V as well. The pins are **not** 5V tolerant, applying more than 3.6V on any pin will release the [magic smoke](https://en.wikipedia.org/wiki/Magic_smoke) (fry the chip). The maximum current that can be drawn from a single GPIO pin is 12mA.

### Power Supply
!!! warning "It is important to have a reliable power supply"

The [power supplied to the device](https://www.letscontrolit.com/wiki/index.php?title=Power) is **one of the most important elements** for stable device operation. Many devices on the market have barely adequate power supplies for normal operation. ***Connected peripherals may strain the ability of the power supply on the device to deliver appropriate power to all the components, both on-board as well as externally connected.***

Voltage regulation issues typically result in fatal exception [fault code](https://github.com/esp8266/Arduino/blob/master/doc/exception_causes.rst) `1`. You must ensure that the device receives sufficient power (current **and** appropriate voltage level). Take into account the current that each wired component (f.e. sensor) will draw from the device itself. 

### Electrical Considerations
When you switch a GPIO pin to an input and hang a long wire off of it, that wire can pick up stray signals and cause the voltage on the GPIO pin to vary. This can cause the system to think the switch has changed.

To fix this, there are several things you can do.

1. add a pull-up resistor
2. add a bypass capacitor
3. shielding on the wire
4. use twisted pair wiring

A pull-up resistor is a resistor connected between the GPIO pin and 3.3v. The exact value of this is not critical, 4.7k is a common value to use, as is 10k. This ensures that when the switch it open, the GPIO pin will go high.

A bypass capacitor is a small (pF range) capacitor that is connected between the GPIO and ground. This provides a path for any radio signals that are picked up by the wire to go to ground and not confuse the system.

Shielding or using twisted pair wiring are other ways to reduce the effect of radio signals on the system.

[Example for 10K Resistor](https://user-images.githubusercontent.com/35574450/39960640-8b2735ca-5626-11e8-8128-461b6d9976ad.png)
(issue[#2708](https://github.com/arendst/Tasmota/issues/2708#issuecomment-388574891))

## ESP8266 In Depth
Complete document available from https://tttapa.github.io/ESP8266/Chap04%20-%20Microcontroller.html
## Digital I/O
Just like a normal Arduino, the ESP8266 has digital input/output pins (I/O or GPIO, General Purpose Input/Output pins). As the name implies, they can be used as digital inputs to read a digital voltage, or as digital outputs to output either 0V (sink current) or 3.3V (source current).

### Usable Pins
The ESP8266 and ESP8285 have 17 GPIO pins (0-16) but several are [reserved or have constraints](https://randomnerdtutorials.com/esp8266-pinout-reference-gpios/). Do not use any of the _**reserved**_ pins. If you do, you might crash your program. On the ESP8266, six pins (GPIO 6 - 11) are used to interface the flash memory (the small 8-legged chip usually right next to the ESP8266). The ESP8285 has its flash memory integrated into the chip which frees up GPIO 9 and 10.

GPIO 1 and 3 are used as TX and RX of the hardware Serial port (UART), so in most cases, you can’t use them as normal I/O while sending/receiving serial data.
GPIO 1, 2 and 3 will cause boot failure if LOW on boot - use with care.

![best ESP8266 pins](https://raw.githubusercontent.com/thehookup/Wireless_MQTT_Doorbell/master/GPIO_Limitations_ESP8266_NodeMCU.jpg)

#### Boot Mode Pins
Some I/O pins have a special function during boot: They select 1 of 3 boot modes:

GPIO15 |	GPIO0 |	GPIO2 |	Mode
---------------|-----------|------------|------------------------------------------------------------------
0V |	0V	| 3.3V	| Uart Bootloader
0V |	3.3V |	3.3V |	Boot sketch (SPI flash)
3.3V	| x|	x |	SDIO mode (not used for Arduino)

Note: you don’t have to add an external pull-up resistor to GPIO2, the internal one is enabled at boot.

We have to be sure that these conditions are met by adding external resistors, or the board manufacturer of your board has added them for you. This has some implications, however:

GPIO15 is always pulled low, so you can’t use the internal pull-up resistor. You have to keep this in mind when using GPIO15 as an input to read a switch or connect it to a device with an open-collector (or open-drain) output, like I²C.
GPIO0 is pulled high during normal operation, so you can’t use it as a Hi-Z input.
GPIO2 can’t be low at boot, so you can’t connect a switch to it.
Internal pull-up/-down resistors
GPIO 0-15 all have a built-in pull-up resistor, just like in an Arduino. GPIO16 has a built-in pull-down resistor.

### PWM

**ESP8266** Unlike most Atmel chips (Arduino), the ESP8266 doesn’t support hardware PWM, however, software PWM is supported on all digital pins. The default PWM range is 10-bits @ 1kHz, but this can be changed (up to >14-bit@1kHz). Check [Restrictions](#restrictions).

ESP8266 has only software and supports 5 PWM channels. `PWM` and `PWMi` GPIOs are used in two modes depending on `SetOption15`: either as lights or as pure PWM.

**ESP32** has hardware PWM support, named `ledc`, up to 16 channels depending on CPU type. You can mix lights and pure PWM channels. The first 5 PWM are reserved for lights, unless `SetOption15 0`. For pure PWM GPIOs, you can assign any PWM number, they don't need to be continuous. For example you can use `PWM 1/2/3` for a 3-channel RGB light, and `PWM 6` & `PWM 10` for pure PWM at the same time.

CPU type|PWM channels
:---|:---
ESP32|16 channels
ESP32S2|8 channels
ESP32C3|6 channels

Channels are assigned to GPIOs in a first-in-first-serve way, and PWM GPIOs are assigned first. If `ledc` channel are exhausted, an error will appear in logs.

The following GPIOs use `ledc` PWM channels:

GPIO type|Description
:---|:---
`PWM` or `PWMi`|`PWM 1..5` are used for lights, `PWM O6..11` are general purpose PWM.
`LedPwmMode`|Assigns a `Led` GPIO to a PWM channel
`Buzzer`|If `BuzzerPwm` is used
`Backlight`|PWM backlighting for displays
`XCLK`|Used as a clock generator for webcam

Example of `PWM` console output with 16 PWM assigned. By default PWM range is 0..1023.

```
RSL: RESULT = {"PWM":{"PWM1":410,"PWM2":286,"PWM3":286,"PWM4":0,"PWM5":0,"PWM6":0,"PWM7":0,"PWM8":0,"PWM9":0,"PWM10":0,"PWM11":0,"PWM12":0,"PWM13":0,"PWM14":0,"PWM15":0,"PWM16":0}}
```

#### Auto-phasing of PWM

(ESP32 only) By default, phases of consecutive PWM are disaligned so that a PWM pulses starts when the pulse of the previous PWM channels ends. This helps in distributing over time all pulses and have a smoother effect on power supply.

With auto-phasing:
![PWM_autophase](https://user-images.githubusercontent.com/49731213/151606010-cf24d1bc-348f-493d-b5aa-9aa867c41b77.jpg)


You can revert this with `SetOption134 1`; all phases are synces and all pulses start at the same moment.
![PWM_sync](https://user-images.githubusercontent.com/49731213/151606020-e1e95f27-40ae-4e1b-8e13-64309e74e1c4.jpg)


#### H-bridge

H-bridge is an electronic circuit that switches the polarity of a voltage applied to a load. It uses 2 PWM outputs to control the current sent to each polarity.

When auto-phasing is enabled, you can use 2 consecutive PWM to drive a H-bridge siunce PWM phases don't overlap - under the condition that the sum of both PWM don't exceed `1023`.

**Important**: you must always ensure that the sum of both PWM channels is less or equal than `1023`. Values over this threshold can damage the circuit.



### Analog Input
The ESP8266 has a single analog input, with an input range of 0 - 1.0V. If you supply 3.3V, for example, you will damage the chip. Some boards like the NodeMCU have an on-board resistive voltage divider, to get an easier 0 - 3.3V range. You could also just use a trimpot as a voltage divider.

The ADC (analog to digital converter) has a resolution of 10 bits.

## Communication 
### Serial
The ESP8266 has two hardware UARTS (Serial ports):
UART0 on pins 1 and 3 (TX0 and RX0 resp.), and UART1 on pins 2 and 8 (TX1 and RX1 resp.), however, GPIO8 is used to connect the flash chip. This means that UART1 can only transmit data.

UART0 also has hardware flow control on pins 15 and 13 (RTS0 and CTS0 resp.). These two pins can also be used as alternative TX0 and RX0 pins.

### I²C
ESP8266 doesn’t have a hardware TWI (Two Wire Interface) but it is implemented in software. This means that you can use pretty much any two digital pins. By default, the I²C library uses pin 4 as SDA and pin 5 as SCL. (The data sheet specifies GPIO2 as SDA and GPIO14 as SCL.) The maximum speed is approximately 450kHz.

### SPI
The ESP8266 has one SPI connection available to the user, referred to as HSPI. It uses GPIO14 as CLK, 12 as MISO, 13 as MOSI and 15 as Slave Select (SS). It can be used in both Slave and Master mode (in software).

### GPIO Overview
[NodeMCU Labelled Pin](https://techtutorialsx.com/2017/04/02/esp8266-nodemcu-pin-mappings/) | GPIO# | 	Function | 	State	 | Restrictions
-|-|-|-|-
D3 | 0 | 	Boot mode select	 | 3.3V	 | No Hi-Z
D10 | 1 | 	TX0 | 	-	 | Not usable during Serial transmission - Boot will fail if LOW at boot
D4 | 2 | 	Boot mode select TX1 | 	3.3V (boot only) | 	Don’t connect to ground at boot time - boot will fail. Sends debug data at boot time
D9 | 3 | 	RX0 | 	-	 | Not usable during Serial transmission
D2 | 4 | 	SDA (I²C) | 	-	 | - 
D1 | 5 | 	SCL (I²C) | 	-	 | -
 x | 6 - 8  | 	Flash connection | 	x | 	Not usable, and not broken out
 x | 9, 10  | 	Flash connection * |  | 	*Only available on the ESP8285*
 x | 11 | 	Flash connection | 	x | 	Not usable, and not broken out
D6 | 12 | MISO (SPI) | 	- | 	-
D7 | 13 | 	MOSI (SPI)	 | - | 	-
D5 | 14 | 	SCK (SPI)	 | - | 	-
D8 | 15 | 	SS (SPI)	 | 0V	 | Pull-up resistor not usable (extern pull down resistor)
D0 | 16 | 	Wake up from sleep	 | - | 	No pull-up resistor, but pull-down instead Should be connected to RST to wake up

<!-- 
I2C: 
<img src="https://img.shields.io/static/v1?label=&message=i2c&color=blue" style="float:right"> </img> 

GPIO:
 <img src="https://img.shields.io/static/v1?label=&message=gpio&color=purple"  style="float:right"> </img> 

ADC:
 <img src="https://img.shields.io/static/v1?label=&message=adc&color=orange"  style="float:right"> </img> 

Serial:
 <img src="https://img.shields.io/static/v1?label=&message=serial&color=seagreen"  style="float:right"> </img> 

Serial:
 <img src="https://img.shields.io/static/v1?label=&message=spi&color=slategrey"  style="float:right"> </img> 

Tuya:
 <img src="https://img.shields.io/static/v1?label=&message=tuya&color=orangered"  style="float:right"> </img> 
 -->

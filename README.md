# BeeperBand

The BeeperBand is an open hardware project to build a minimalist wrist band to notify smartphone message notification with haptic, RGB lights, and interact with physical buttons via BLE connection.

Current prototype intentionally not fitting any display, there are only 3 RGB LEDs, for visual indications. A Linear Haptic Driver can notify user with patterned vibration with minimum audible noise, for non-intrusive usage in various social occasions.

The hardware design is a basically a re-layout PCB of a Adafruit nRF52 Express, to minimize initial firmware development efforts. It also has more sophisticated charger, peripherals required by project specs, and various design decisions for power saving and MCU IO voltage level translations, to utilize software configurable voltage features of nRF52 integrated power converters.

## System Design

nRF52840 is a 2.4GHz RFIC with integrated power converters and flexible IO mapping, with USB2.0 full speed PHY, but has no DFU rom(will brake when no proper firmware present). It can be powered directly with USB host or Li-ion battery, with two stage convert, or single stage convert (skipping first high voltage conversion); two stage conversion could both choose between integrated LDO or buck converter, while buck converter require extra inductors and capacitors to be functional.

To maximize battery life, the BeeperBand implemented two stage buck converter design, all peripheral I/Os have dedicated level translation, and core of nRF52840 works on default 1.8v.

Current stage there’s no functional firmware for BeeperBand hardware (v0.41), all testing is done with Adafruit Feather nRF52840 express bootloader and accompanying BSP and Arduino.

## Peripherals

- 6 GPIO buttons, low-active (enabled when connected to GND)
- 3 x RGB LEDs, SW2812 serial protocol, daisy chained
  - powered by dedicated 3.3V buck converter, with EN pin
  - buck converter is powered by battery
- 1x Linear Haptic Driver
  - dedicated level transfer chip for I2C and GPIO
  - powered by battery, wide voltage range tolerate by level translation
- All unused IO fanned out by 30pin B2B connector
  - with battery output pins
  - 20 IO pins
- 2 LEDs, inherent from Adafruit nRF52840 Feather Express Design
  - RED for Arduino serial activity
  - BLUE for BLE connection activity
  - testing purpose, so the brightness is set to very low (1k resistor on 1.8V IO)

## Charger Circuit

Charger chip is BQ25619 from Texas Instrument with QFN footprint. A WLCSP footprint version is available with part# BQ25618. If PCB size requires further miniaturization, charger and MCU chip can  both replaced with software compatible WLCSP (0.4mm pitch BGA) footprint version.

Like many other BQ chargers from TI, it has a register map for extensive internal controls. But preliminary charging functions does no require MCU control presents. The I/O pins connected to the charger are unused in Feather Express design, so application developer can ignore its presents. To reach 4.2V charge voltage (default 4.1V), and charging current control, a driver firmware is required, and it will work with no intervention to application programs. The power source transition between charging and battery is also seamless, with no program control needed.

BQ chip has 4 power terminals, a little back ground may be helpful in first encounter.

- USB input (VBUS)
  - charge power input
- USB output (PMID)
  - if USB power output as a USB host is required, BQ chip works can work in boost mode to deliver 5V output from battery voltage.
- Battery output (VSYS)
  - Li-ion battery have a minimum voltage required to prevent over discharge, this output will shutdown output to protect battery. If system is overheat, or charge voltage is over-voltage, BQ will also disconnect battery connection.
- Battery input (BAT)
  - This is the dedicated battery connection node. BQ chip has no discharge meter to implement an accurate gas gauge function, but a battery voltage monitor circuit exist in system, to save power this circuit could be shutdown to prevent power leak.

## Testing procedure

PCBA is hand built, this is a long process and require lots of craftsmanship. The SWD port used for upload firmware is not a standard pulg but four small soldering pads, to save space. Fresh nRF52 chip has no per-programmed boot-loader, nor the USB has a hardware DFU mode to boot is from desktop operating system.  To test hardware functionalities, the Adafruit Feather nRF52 Express bootloader is flashed.

Testing procedures are done with Arduino sketches, requires user to upload them separately for each function:

- Buttons
  - not fully implemented due to default Ardruino BSP include only one PIN\_BUTTON(7)
- RGB leds
  - PIN\_NEOPIXEL (8))
- Haptic
  - Requires Adafruit\_DRV2605.h
  - Change writeRegister8(DRV2605\_REG\_OVERDRIVE, 1);
  - call “use\_LRA() in setup()

Currently charger driver is not implemented, but charger I2C communication could be verified.

## Firmware Development

Due to lack of BLE communication software nor data model design, a serial based BLE protocol to manipulate system peripheral called Firmata is recommended to make use cases mock up. During this testing phase, there’s no sophisticated radio power control method could be implemented. This mock up communication firmware will have significant more power consumption than a production firmware, due to communication protocol overhead and onboard program overhead.

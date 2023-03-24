# **BeeperBand Firmware Dev 101**

Assuming you are a proficient C Kernel developer, or a CPP application programmer, this is the first time to work with an embedded system, this document will guide you through the first adventure to make BeeperBand blinking as you wish.

## **Porting BeeperBand as a new Arduino hardware**

Before your program could be uploaded (old fashioned embedded programmer would say, it’s downloaded) to the FLASH resides inside MCU, you will need a BSP in your IDE to invoke hardware specific commands with generic function call naming conventions, such as LEDPIN or PULLUP etc, so you don’t have to search through MCU datasheet’s register table or vendor’s library source code; And another thing required is the bootloader could use USB to talk with IDE’s firmware flashing program, or you will have to use an extra USB dongle generically called “JTAG”, to flash your binary file with specific location to boot the MCU.

This is why BeeperBand schemaitc has a weird pin mapping

<img width="246" alt="CleanShot 2023-03-24 at 08 53 42@2x" src="https://user-images.githubusercontent.com/1048265/227576582-2f47ed8d-7e68-4742-98fb-fd9de3819ce9.png">


This mapping is how the BSP, specifically, nRF52840 chip’s Arduino port, from “Adafruit Feather nRF52840 Express” dongle, has done. BeeperBand’s RGB LED is connected to NEOPIX pin is connected to P0.16 pin, existing program works with NEOPIX pin will work with BeeperBand as well, all you need to do is refer to [https://learn.adafruit.com/introducing-the-adafruit-nrf52840-feather/arduino-bsp-setup](https://learn.adafruit.com/introducing-the-adafruit-nrf52840-feather/arduino-bsp-setup), pretending BeeperBand is a Feather board.

But BeeperBand has extended this BSP, you will need to figure out how BSP package is resided in your specific OS (windows, linux, MacOS), and try to update vanilla BSP project to make new definitions for BeeperBand hardware work, e.g. P1.07, to enable or disable the NEOPIX LED’s power converter, to save some few milliamp current, where this pin is not fanned out in Adafruit Feather nRF52840 Express. BSP project is on GitHub: [https://github.com/adafruit/Adafruit_nRF52_Arduino](https://github.com/adafruit/Adafruit_nRF52_Arduino), inside the “variants” folder, create a new folder for BeeperBand, and copy a sibling folder as a template.

Note before uploading new program to BeeperBand:

- has your new board entry appear in your Arduino IDE?
- New firmware compiled with your BSP project indeed working? Try toggle some newly defined pins
- HS and LS pins are same in function, but LS pins’ traces inside chip are located near wireless part, use them for bus data will degrade it’s communication performance, so use LS pins for low speed IO commands such as chip enable etc.
- All pins on nrf52840 could be used for specific bus functions, e.g. I2C or SPI. AIN_pins are analog input pins, not usually used for these purposes.

There’s another project you will need to take care of, it’s the Bootloader: [https://github.com/adafruit/Adafruit_nRF52_Bootloader](https://github.com/adafruit/Adafruit_nRF52_Bootloader)

## **Update Bootloader**

This is a “miniaturized” BSP project, resides in FLASH, could be invoked by pressing RESET button during power on the BeeperBand, or automatically running when no program exists in your BeeperBand. The mapping file located in Adafruit_nRF52_Bootloader/src/**boards**/ folder.

Be careful if this program not running properly, your device is bricked. Brick means there’s no booting firmware program exists to bring up basic functions, e.g. USB device to communicate with Arduino. You will have to connect SWD port with a SWD download device, with vendor’s flash program.

Refer to [flashing guide.odt] to un-brick.

Bootloader’s job is to get user program running (aka, the binary BSP built with your *.ino in Arduino IDE), unless there’s no working user program or user choose to boot to Bootloader. Once BSP program is loaded, there’s no effect what Bootloader has defined, so if your BSP and Bootloader project have different hardware mappings, they will not conflict each other.

## **BeeperBand’s firmware develop goals**

After you can work with BSP and Bootloader project properly, you can begin reading datasheets of each functional component try to build “device driver”. Arduino IDE has its way to organize your driver project from BSP project, and it’s also OS dependent. You are on your own to make this work again, and Adafruit’s tutorial again is a good starting point.

A list of onboard devices required for new drivers:

- Haptic motor, existing driver is incomplete, and some modifications are required to get most power output (in README.md)
- 3x RGB LED array, requires modification of BSP to get the chain length right.
- Battery Charger, will need to define charging current and temperature protection custom criteria
- MCU’s ADC setup for proper battery voltage reading
- All power enable/disable GPIOs for power saving

Then you are ready for BLE development, this is beyond this document.

EOF.

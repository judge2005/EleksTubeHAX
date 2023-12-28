# EleksTubeHAX - An aftermarket custom firmware for the EleksTube IPS clock
Buy your own clock under the name "EleksTube IPS Clock" on eBay, Banggood, etc. Look for places that offer 30-day guarantee, otherwise it can be a fake product!

[Reddit discussion on the original hack is here.](https://www.reddit.com/r/arduino/comments/mq5td9/hacking_the_elekstube_ips_clock_anyone_tried_it/)

[Original documentation and software from EleksMaker.](https://wiki.eleksmaker.com/doku.php?id=ips)

This is just the original project that has been modified to be built using platform.io. The benefits of doing this are:
* The library dependencies are handled automatically
* The correct ESP32 board and the binary sizes are pre-defined
* You no longer have to modify any files in TFT_eSPI
* The images (in the data directory) can be built and uploaded without having to install any tools and without having to install an old version of the Arduino IDE.

I plan to make more changes, but only if I find time.

# Hardware modification
Original EleksTube has a few problems in the hardware design. Most notably it forces 5V signals into ESP32 which is not happy about it. And it is outside of safe operating limits. 
This will extend the lifetime of ESP32. Mine died because of this...
## Conversion
CH340 chip, used for USB-UART conversion can operate both on 5V and 3.3V. On the board it is powered by 5V. Cut one trace on the bottom side of the board that supplies the chip with 5V and route 3.3V over the resistors / capacitors to VDD and VREF.
See folder "Hardware modification" for the photo.

# Backup first!
If you mess-up your clock, it's only your fault. Backup images from other uses DO NOT WORK as the firmware is locked by MAC address of ESP32.
### Install the USB Serial Port Device Driver
[EleksTube instructions](https://wiki.eleksmaker.com/doku.php?id=ips) instruct for installing a serial port driver.
On Windows 10, plug-in the cable and run Windows Update. It will find and install the driver. 
On Linux it works out of the box.
### Save your original FW
* Install ESP32 support (see below)
* Navigate to folder `\Arduino\..\packages\esp32\tools\esptool_py\3.0.0` or `\original-firmware\`
* Copy file `\original-firmware\_ESP32 save flash 4MB.cmd` and `_ESP32 write flash.cmd` to this folder
* Change COM port number inside both files
* Run "save flash" file and wait until it saves the contents of your flash to `backup1.bin`
* if you want to restore your original firmware run `_ESP32 write flash.cmd`

# How to build this firmware
These instructions assume you already know how to use platform.io, and just need to know WHAT to do.

## Download this code
You're either reading this file after downloading it already, or you're reading it on github.  I'll assume you can figure out how to get the code from github and put it somewhere on your local machine.  This is your preference.

## Setup Platform IO
If you don't already have platform.io installed, go to [https://platformio.org/](https://platformio.org/install/ide?install=vscode) and follow the instructions there. This project uses the VSCode version.

### Modify platformio.ini
* Change _upload_port_ and _monitor_port_ to match whatever port your clock is plugged in to. 
* For "SI HAI clock" also add RTC by Makuna (developed on 2.3.5) https://github.com/Makuna/Rtc/wiki

IPgeolocation and NTPclient libraries were copied into the project and heavily updated (mostly bug fixes and error-catching).

## Upload New Firmware
Make sure you configured everything in `USER_DEFINES.h`.
* Your MQTT credentials :: Register on [SmartNest.cz](https://www.smartnest.cz/), create a Thermostat device, copy your username, API key and Thermostat Device ID.
* Uncomment MQTT service (if in use)
* Your Geolocation API :: Register on [Abstract API](https://www.abstractapi.com/), select Geolocation API and copy your API key.
* Select hardware platform (Elekstube, NovelLife, or SI_HAI) :: un-comment appropriate hardware define 
* Select if you prefer WPS or hardcoded credentials for WIFI (comment 'WIFI_USE_WPS' line and add credentials if desired)

Connect the clock to your computer with USB.  You'll see a new serial port pop up.
### Compile and Upload the Code
Compile and upload the code.  At this point, it should upload cleanly and successfully.  You'll see the clock boot up and ask for WPS.  But it doesn't have any bitmaps to display on the screen yet.

### Upload Bitmaps
The repository comes with a set of CLK and BMP files, modified images from the original firmware, in the `data/` directory. See below if you want to make your own.

### Custom Bitmaps (Optional)
If you want to change these:
* Create your own BMP files.  Resolution must be max 135x240 pixels, 24bit RGB. Can be smaller, it will be centered on the display. Cut away and black border, this only eats away valuable Flash storage space!
* Name them `10.bmp` through `19.bmp`; `20.bmp` to `29.bmp`, and so on. You can add as many as you can fit into SPIFFS space.
* Run the tool `\Prepare_images\Convert_BMP_to_CLK.exe` (windows only sadly).
* Select all prepared BMP files at once. It will create CLK files with smaller size.
* Put them in the `\data` directory.
* Then use platform.io to upload them again.
Each set (1x, 2x, etc) can be chosen in the menu.

### Configure your WiFi network
For WPS:  When prompted by the clock, press WPS button on your router (or in web-interface of your router). Clock will automatically connect to the WiFi and save data for future use. No need to input your credentials anywhere in the source code.  THe clock will remember WiFi connection details even if you unplug the clock.

# Development Process:
See SmittyHalibut on GitHub (original author of this alternative FW) for details.

Notes are in the document  `Hardware pinout.xlsx`

*Happy hacking!*
   - Aljaz

## P-Suite

<img align="right" src="images/nodemcu.png">
An integrated suite of micropython files for ESP8266.
Based on [micropython.org version](http://micropython.org/download).

## May 2018 - This is no longer being maintained by the original author.
## Please feel free to fork, copy, adapt if you find it useable

## Overview:

This suite of files is intended primarily for use with ESP12-based
boards (including NodeMCU and Wemos D1-mini), but it does work for
ESP-01, within its gpio limits. pSuite is modelled on the earlier eLua [eSuite](https://github.com/BLavery/esuite-lua).

<img align="left" src="images/esp-12.png">pSuite automates the standard startup including escape time, wifi connection and time
setting. This leaves you to concentrate just on your project scripting: on exactly
what you want to control. Included are libraries for blynk and oled display. More libraries are intended later.

<img align="right" src="images/esp01.jpg">The pSuite projects are intended to be used as client (“STATION” mode)
in conjunction with a nearby wifi access point. The ESP8266 is a
wifi-capable chip, and merely using an isolated “blink a LED” project
misses its point!


The micropython environment uses the SOC native numbers for GPIO pins (0 1 ... 16). This is the chip's native GPIO numbering as used also by the arduino-esp environment. You may need to cross reference from the D0 D1 labelling seen on nodemcu devkit boards.

## Disclaimer/Claimer:

<img align="right" src="images/d1.jpg">As at October 2017, this project is merely "bare-bones". Please consider it as 0.1 alpha. However, it DOES run.

It is groundwork for a proposed class starting in 2018. My classes tend to prefer interpreter environments rather than compiler. The nodemcu eLua interpreter based on the "non-OS" SDK has occupied this position for ESP8266 until now. It has been stable, well documented and effective.

For the ESP32, there are currently at least 3 lua development attempts "out there". Espressif has terminated the "non-OS" SDK used by the original "nodemcu" lua, and is now offering only the free-rtos SDK (a much better long-view choice). I can't find a lua worth using, or usably documented. Maybe the wind is out of the lua sails.

I believe the interpreter future is probably micropython. The ESP8266 micropython has reached workable status, and its documentation is reasonable if not quite finished. 

*The ESP32 micropython build is published for trying, but clearly a lot of work remains. It seems to be undocumented, but perhaps it will become a fairly faithful clone of the ESP8266 version.*

And faux-python is a lot more mainstream than lua. And more student friendly than faux-arduino faux-C++ or hardcore SDK/rtos C.

All that said, where is micropython/esp really headed? At micropython, the esp8266 kickstarter campaign is about done. Adafruit's CircuitPython fork is racing away separately. On ESP32, the trumpetted marriage 9 months ago between Pycom & the main micropython seems marked by silence.

So the efforts as documented below are laid out in hope they may be useful to some. Will they expand to a class here in 2018? We'll see.

## Common startup files:

1. boot.py
1. main.py
1. wifi.py
1. sntp.py
1. settings.py

<img align="right" src="images/init.png">These are always used. boot.py and main.py are as mandated by micropython. main.py imports wifi and sntp to start communication and fetch real time. 
main then passes control to
your individual “project” file. So the standard minimum is five files, plus your project.

The compulsory **settings.py** is intended as a general-purpose config file easily importable by other modules. It includes your wifi credentials, i2c pin choice, blynk token, etc. And importantly, you **edit and re-upload the settings file to designate a new project file**.


**boot.py** simply ensures that wifi has enabled the auto-reconnect to your wifi router/AP.

**main.py** turns on (for 2 seconds) the led inbuilt to the ESP12 submodule. At the end of that period, the flash button ("D3" / gpio-0) is sampled. If pressed, processing terminates there. This gives you an escape mechanism in the case a script error is causing repeated reboots, although this is less a problem in micropython than in Nodemcu lua.

<img align="right" src="images/wifi.png">main.py then calls **wifi.py**. If your settings have nominated an AP mode password, then the ESP8266 AP server will start up on 192.168.4.1.  

If wifi has by now auto-connected as client to your local network, there is no more connecting to do. Otherwise, each router credential listed in settings.py (and provided it is seen in a scan) will be tried for login. The successful connected IP will display to terminal.

wifi.py finishes by starting webrepl on the network(s) active.

<img align="left" src="images/time1.png">Then main.py calls **sntp.py**, a more robust sync function than inbuilt ntptime module. Sntp makes as many as 4 attempts to fetch internet time. You nominate in your settings file two ntp servers to use.

This library sets a 3-hour repeating timer for the time-sync operation, correcting any RTC drift of the ESP8266.

Finally, you are left with a useful asctime() function (readable timestamp) callable anytime:<img align="right" src="images/project.jpg">

	import snpt
	print(sntp.asctime())

main.py has one more job: to launch your nominated **project file**, which you nominate in the settings file. Obviously you must build your own project file(s), but any of the examples files could be a starting point.

*Maybe you have a single "project" to construct. In my classroom environment, students swap (just) the project file perhaps several times in a class.*

## Blynk Library:

On micropython, a key consideration is shortage of RAM memory, and blynk needs a large and complex library. There is however a blynk library included here in pSuite.

[For the blynk API see HERE.](pblynk.md)



## Oled library:

<img align="right" src="images/oled.jpg">In the classrom environment where eSuite & pSuite are targetted, 
an oled display is used in most projects, usually the 128x64 "0.96 inch" ubiquitous module. It is easy, cheap and versatile.

**Oled.py** is a wrapper to the usual/official sd1306.py module for ESP8266/ESP32. If an I2C scan shows oled is present, Oled.py initialises the display and presents initial info: IP number and time.  Thereafter, you use oled calls as per the framebuf documentation. 

## "Build" and IDE?

ESPlorer as used on the nodemcu lua environment does NOT support the current micropython, despite what it may claim.

The options for communicating with your micropython ESP are:

1.  plain terminal (puTTY, gtkterm, etc) - no file transfer
1.  uPyCraft on Windows - not featured, temperamental
1.  AMPY cli from Adafruit - I found some commands crashed
1.  rshell cli
1.  mpfshell cli - similar to AMPY and **my preference for CLI**
1.  Browser based using webrepl interface
1.  **uPyLoader - MY CHOICE**, [here](https://github.com/BetaRavener/uPyLoader), altho no folder support at MCU

The executable I use for uPyLoader (linux version) is here on /IDE folder.

The [ESP8266 micropython binary](http://micropython.org/download#esp8266) I fetched direct from micropython website. The bin I used is here in the /bin folder.

You need recent esptool.py to flash the binary to the board. There is a copy here in /bin folder.

In all cases you ought to fetch your own copies of these tools.

uPyLoader IDE currently supports folders at PC end but not at the ESP end. So while "library" files in this repository can happily live in a /lib folder at PC, upload them all in together into the base folder at the ESP. Same applies to your selection of example or "project" files.

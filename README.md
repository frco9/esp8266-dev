## Why this?
Short: I do not like the developing environment provided by Espressif. Then I rewrite Makefile to:
- reuse common code, e.g. MQTT, flash file system, and driver for SoC peripherals.
- have other json parsing. Current json parsing of SDK (take from Contiki OS) is not good enough, I have no way to check if this/that string is a valid JSON.
- quick setup, e.g. make, make flash, make login, make OTA=1 fash.
- make firmware over-the-air update easy, e.g. make RELEASE=MINOR/MINOR/PATCH. This is the on going process.

All examples have been testing with 512KB Module-01 and 4MB Module-12.

## Setup env

Use with SDK v1.1.2, setup with esp-open-sdk. With location of esp-open-sdk, change ESPRESSIF_ROOT in Makefile.common to point to this SDK. We will use lib, include, ld codes/scripts from SDK, ah, together with compiling tools off-course.

To use OTA update firmware with 512KB, change linker file ld/eagle.app.v6.new.512.app1/2.ld, section irom0_0_seg to have more spaces. Please note that Module with 512KB is very small for current SDK, recommend to use bigger flash.

```
<  irom0_0_seg :                         org = 0x40241010, len = 0x2B000
---
>  irom0_0_seg :                         org = 0x40201010, len = 0x31000
```

I update tools/gen_appbin.py to avoid compiling error. Off-course I can change $PATH of shell to avoid touching this Python script.
```
<         cmd = 'xtensa-lx106-elf-nm -g ' + elf_file + ' > eagle.app.sym'
---
>         cmd = '~/esp-open-sdk/xtensa-lx106-elf/bin/xtensa-lx106-elf-nm -g ' + elf_file + ' > eagle.app.sym'
```

## OTA server
A simple enough to use OTA server could be found at https://github.com/ubisen/ota-update/, wrote in nodejs with mongodb to store images. It's recommended to setup a server locally, but the code is good enough to run in public domain.
Normally, a proxy like nginx is recommended to run nodejs server.

OTA-client, which upload images (1, 2 for ESP) and register new version, is found at tools/fotaclient. Need more update to use with make (pull request is more than welcome, since I am quite low in bandwidth now).

## OTA client
Found at /tools/fotaclient. APIKEY should be provided by go to OTA server >> Profile, copy "apiKey" from return JSON.

## To use

```
make OTA=1/0                # default OTA=1
make OTA=1 IMAGE=1/2        # generate image user1/user2
make OTA=1 IMAGE=1/2 flash  # program with OTA enable
make FLASH_SIZE = 512/4096  # choose flash size
make register               # upload, register new version. Must setup OTA server first
```

## How to write makefile for program:

Declare ROOT, this is the root of the project, from which could access apps and Makefile.common

Example of Makefile for program under ```/examples```:
```
ROOT = ../..
APPS += fota
include ${ROOT}/Makefile.common
```

## Using APP
Please (me), check for frequence update

+ mqtt https://github.com/tuanpmt/esp_mqtt
+ json parsing https://bitbucket.org/zserge/jsmn
+ flash file system https://github.com/pellepl/spiffs

## Todo
+ makefile with all options of flash
+ flash file system testing

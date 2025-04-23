---
tags:
  - printing
---
# JKR-s-E3Ez-setup
my personal setup for a BTT E3Ez with a CB1 compute unit.

This guide will give you a basic clue of how you may set up the board, though i don't guarantee it'll work.
you will also find the relevant setup file (printer.cfg) in this repo, though the version uploaded here is a bit out of date.
this repo is mainly for my own use, so please don't expect to much.

# Basic BigTreeTech Manta E3EZ CB1Klipper  setup walkthrough / Guide

Getting started with Klipper can be frustrating for beginners. That's why I made this Walkthrough
---

 More than that, it can be neigh impossible without more competent help. Therefore, I have decided to write up this small walkthrough. I call it that, because frankly, I too have no clue what is going on. I am just outlining steps I know work.

This article is heavily paraphrased from [this](https://3dpandme.com/2022/10/02/tutorial-btt-manta-m8p-cb1-klipper-guide/) other article, which is for another BTT-board, and which I found to be a tad too technical for me, and BTT['s Manual for the E3EZ](https://github.com/bigtreetech/Manta-E3EZ/blob/master/BIGTREETECH%20Manta%20E3EZ%20V1.0%20User%20Manual.pdf). You should keep at least the manual at hand, as it includes some useful info.
I also recommend taking a gander at [this article](https://os.ratrig.com/docs/boards/btt/manta-e3ez/), as it is useful for wiring.
# Requirements:
### Hardware:
|amount|What|Note|
|--|--|--|
|1| Manta E3EZ ||
|1|CB1||
|1|micro-USB-cable|MICRO, not type C|
|1|Jumper|Was delivered in the box for me|
|2|micro-SD cards|These will be **permanently deployed** with the E3EZ. **Quality and size of the matters**, as they are what your operating system / Firmware will be running of.|
|1|Ethernet cable|**Optional**, as the E3EZ can use Wi-Fi, though the activity indicator light of the Ethernet jack is surprisingly useful, at least during setup|
|1|PC / laptop|While I will try to give instructions for all, I used Linux. I give no guarantee what i say actually works on Windows or non-fedora distros|

### Software:
|What|why|Notes|
|---|---|---|
|[Raspberry Pi Imager](https://github.com/raspberrypi/rpi-imager)|Flash drive image to SD card|anything that can flash .IMG files works|
|SSH|connect to the E3EZ during setup|SSH on Windows may differ from SSH on Linux, I do not know.|
|SCP|Move files from E3EZ to Computer|I will use gnome file manager.|
|Web browser|Download files / read walkthrough||

# Some explanations
all sections will include "explanation" subsections, in case you encounter issues following this walk through. Blindly following along *should* work, but understanding what you are doing always helps, especially once you encounter issues.

# 1. flashing the OS
### For your understanding
The E3EZ requires 2 SD-cards. One holds the operating system, and files you regularly use. This means things like the config, the actual website, and one for the Firmware.

These SD's have to always be inserted. There is no way to flash firmware onto the device, and while there probably is a way to use external hard drives for the OS and personnel files, I do not know that way, and it would be overkilled to include in a walkthrough ostensibly targeted at beginners.

BTT supplies a full disk image for the OS-sd, we therefore do not have to bother thinking about how to format it. The Firmware SD is different. It **has to be** formatted as fat32. Not doing that will mean your E3EZ won't work.

If the SD's you chose for these are defective, the E3EZ will also not work. While you can try using your old, used digicam's 1.5GB micro SD,  this may not work, and will fail significantly sooner than just using a proper SD.
Furthermore, especially your OS-SD should be big enough. It will hold your GCODE files, and those can pile up to quite significant Sizes.

***
as seen in the requirements, your E3EZ will permanently require 2 (micro)-SD's. These should be of proper quality and size, where the size of your OS-SD, the one we will set up now, should be bigger, if you have differently sized SD's.
We will set up one of these now, as the other requires files from this one.

Download and extract [CB1_Debian11_Klipper_kernel(numers).img.xz ](https://github.com/bigtreetech/CB1/releases), and extract the .xz file. This may take a second.

> I used version 2.3.4. Things may change in the future. If you encounter issues, consider re-trying with this version. you should find it by scrolling down on the github page.

Use your imaging tool to load the resulting .img file to your bigger SD card, if your cards differ in size.

img

once your SD is imaged, you can configure it to automatically connect to your local WIFI network. This is not required if you want to use Ethernet.

If you are done, eject the card, and insert it into the Primary SD slot on the board.
![the OS micro-SD slot](https://cdn.discordapp.com/attachments/761694936444698625/1274658543341142118/OS_slot.png?ex=66c30dc6&is=66c1bc46&hm=f4d922df24b89cdb636500ab64382330813d32d205c21329a957242a9e45d547&)

Before powering on the board, It is important to add a jumper on the VBUS pins, as micro-usb power supply won't work otherwise.

![Wehre to place the jumper that enables micro-USB power](https://cdn.discordapp.com/attachments/761694936444698625/1274659384835702826/image.png?ex=66c30e8f&is=66c1bd0f&hm=c03743efc320c15f40bc766e15c5aed06c62d18dcad849e090265a98218d744c&)

at this point, the micro-USB power supply can be plugged in. It is important to not have anything else, except for what was described so far in this walkthrough, plugged in. You may fry parts of your board otherwise.

# Building firmware.bin

once you powered up your E3EZ, it should conect to your local network. If you have attached it with an ethernet cable, you will notice the Ethernet-port's lights flickering up at this point.
Either way, you can now find the Devices IP in your networks management page. It should be something along the lines of:
`192.168.NNN.NNN`
Once you think you found the corect IP, try connecting "pinging" it.
`ping [Your IP]`
The response should look something like this:
(With `NNN.NNN.NNN.NNN` being your IP)
```ping NNN.NNN.NNN.NNN
PING NNN.NNN.NNN.NNN (NNN.NNN.NNN.NNN) 56(84) bytes of data.
64 bytes from NNN.NNN.NNN.NNN: icmp_seq=1 ttl=64 time=19.4 ms
64 bytes from NNN.NNN.NNN.NNN: icmp_seq=2 ttl=64 time=1.48 ms
64 bytes from NNN.NNN.NNN.NNN: icmp_seq=3 ttl=64 time=12.0 ms
...
```
once you are shure there is a conection, you can close the ping with `ctrl+c`

If this fails, look back into your networks management tools. you Probably tried pinging the wrong device. otherwise, go back and re-try every step of the process. sometimes your E3EZ may just not connect to your WIFI or Ethernet every now or then.
if the ping succeded, write down your IP.
 we can move on by connecting to the E3EZ using SSH.
```
ssh biqu@[your ip]
```
`biqu` is your E3EZ's user name.
You should now be prompted to input a password. This is also `biqu`.
once you are in, which may take a while, we will compile your firmware.

navigate to the klipper directory, and adjust some settings.
```
cd ~/klipper/ 
make menuconfig
```
Chose options with the arrow keyes, adjust options using enter

| option-name                                  | what to chose                    |     |
| -------------------------------------------- | -------------------------------- | --- |
| Enable extra low-level configuration options | [*]                              |     |
| Micro-controllrer Architecture               | STMicroelectronics STM32         |     |
| Processor model                              | STM32G0B1                        |     |
| Bootloader offset                            | 8KiB bootloader                  |     |
| Clock Reference                              | 8 MHz crystal                    |     |
| comunication interface                       | USB (on PA11/PA12)               |     |
| USB ids                                      | -------------------------------- |     |
| GPIO pins to set at micro-controller startup | ()                               |     |


After all is set, quit with `Q`, and select save.

run: 
```
make
```
This will take a while now. Go grab snacks.
# Flashing Firmware
Once `make` has run its cource, it will have created a `klipper.bin` file in `home/pi/klipper/out`.

Plug in your Firmware-SD into your PC now. Format it to fat32. Do not name the new partition.

we must now extract this file to our computer. You can use SCP, like so:
```
scp biqu@[your IP]/home/klipper/out/klipper.bin [Where you want the file to be saved at on your PC]
```
I do not recomend that.
I rather recomend using your native file manager (or [winSCP](https://winscp.net/eng/download.php)).
In gnome file manager, you can simply connect to the E3EZ's file system by adding
```
ssh://[your IP]
```
under Other locations, and pressing connect.
Your file-system should now look like this:

img

You allready see where you need to navigate.
Copy the `firmware.bin` file into the newly formated SD now, and rename it to Firmware.bin.
> Do not forget to make a Backup of firmware.bin on your local drive

now run
```
ls /dev/serial/by-id/
```
Copy the outcome.

once you have done this, eject the drive and power down the E3EZ.
Plug in the Firmware-SD, and power on the E3EZ to flash the firmware. you should now see the E3EZ's LED's flash about for some time. 

now SSH-connect to your E3EZ again.
```
ls /dev/serial/by-id/
```
should now output something different to the  results you marked down earlyer, and something like this:

img


now power the E3EZ back down. Extract the Firmware-SD, and plug it into your PC. you should now see that the `firmware.bin` has been renamed to `FIRMWARE.CUR`. If this happened, everything succeded. Re-insert the Firmware-SD, and power on the E3EZ.
if it has not succeded, retry this step. Try leaving the E3EZ on for longer while Flashing the firmware. If this does not work either, retry everything.

# Post-install configuration

You should be able to navigate to the webUI of Mainsail by simply inserting your IP into the adress bar of your browser now.
Doing so may again take a minute, especially since your E3EZ may be setting up some things on first startup.

once there, we must create our printer-configuration file.
For this, create a `printer.cfg` file on your local mashine.
Paste the [generic config from Bigtreetech](https://github.com/bigtreetech/Manta-E3EZ/blob/master/Firmware/Klipper/generic-bigtreetech-manta-e3ez.cfg)'s Github into this.
Now navigate to the `MASHINE` side panel, and select `config` as the root.
You will see a large error on this page. We'll fix that now.
img

upload your `printer.cfg` file.
Open the file by double-Clicking, and scroll untill you see the `[mcu]` option.
Here, you set your Printers serial interface.

We now have to look a few things up, and experiment a bit.
If you press `DEVICES` at the top of the editor-page, you can select SERIAL, and press `REFRESH`. 

![The Menu ought to look a bit like this. The options will only appear if you press the refresh-button](https://cdn.discordapp.com/attachments/761694936444698625/1274506012161999000/image.png?ex=66c27fb8&is=66c12e38&hm=7835bbd98fedbdb9a0601a10aca766020b91c532305898672e6c6ecf6d716911&)

you will see 3 options. For me, The first worked. For you, it may be another.
You may have to try several options

Here, replace the `serial: [something]` line with:
```
serial: [your option]
```
Once you have done so, restart the E3EZ using the prominent `RESTART` button at the top-right of the Error in the `MACHINE` tab.

Once you have done so, you should be able to conect to the board with your browser, and adjust everything from there.











TODO: https://docs.mainsail.xyz/setup/configuration

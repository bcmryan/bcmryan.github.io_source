+++
title = "Hardware"
weight = "20"
+++

The hardware used in this project is based around the Raspberry Pi 4 single-board computer and other consumer grade parts.
The hardware, just like the software in this build tutorial, was chosen because it is easy to find and easy to replace if needed.
Nothing used here is artisan or botique or customed designed; it is there to be used, reused and desposed of as needed.
But do not fret, the end product does not look that bad.

## Desktop
Throughout this process you will need to be able to communicate with your Raspberry Pi run as a headless server (i.e. without a monitor attached to it) via the command line on a terminal emulator on another computer.
Practically this means that you will need have the Raspberry Pi on the same network as the device with a terminal emulator (e.g. desktop, laptop, Android device with Termux, etc.), preferably plugged into the same router via Ethernet.
The commands that I will be giving through the rest of tutorial are based on Linux. If you do want to follow the tutorial step by step, I suggest that you use a Linux operating system, possibly even by just booting from a [live USB](https://en.wikipedia.org/wiki/Live_USB).

## Raspberry Pi 4
The [Raspberry Pi 4 Model B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/) was chosen for this build, as it is pretty much the de facto single-board computer and is widely availble.
As running a server can be pretty demanding, I would absolutely recommend getting either the 4 or 8 GB model.
I do not think that I have ever been limited by RAM with my 4 GB model, but, as with hard drive space, more RAM is better, everything else being equal.

Also, if you ever decide to do something else with your Pi, you can always just erase or remove your microSD card and try something else (like retro gaming, at which the Pi excels).
This is truly the power of chosing a platform that is so widely supported by its manufacturer, third parties and the community at large!

Finally, if you ever get stuck with any problem related to the Raspberry Pi, I would highly recommend that you give a look at the [official documenation](https://www.raspberrypi.org/documentation/).
It is written for beginners but also includes helpful tips for users of all skill levels.
If you have any questions about running a particular program, it may be useful to look at the manual page for that program (e.g. `man ls`).

As the Raspberry Pi is sold just as a single-board computer, you will also need to purchase a microSD card, case, power supply and drive enclosure with hard disks.

## microSD card
When selecting a microSD card, I looked for one from a reputable brand that is intended to be read from and written to continuously and is considered reliable.
The [SanDisk Max Endurance](https://shop.westerndigital.com/products/memory-cards/sandisk-max-endurance-uhs-i-microsd), marketed to be used for security cameras and dashcams, checked all these boxes for me.
The YouTube channel [ExplainingComputers](https://www.youtube.com/channel/UCbiGcwDWZjz05njNPrJU7jA) also has a [comparison of different microSD cards for use in single-board computers](https://www.youtube.com/watch?v=YUResed38uo) in which the related SanDisk High Endurance is compared to other microSD cards.

Capacity is not a real constraint in my Raspberry Pi server build, as the microSD card is really only there to host the rather small headless operating system, installed packages and a few other documents, such as config files and data caches.
As such, I am confident that a 32 GB microSD card should be sufficient for most people.

## Case
While active cooling (i.e. with a fan) might be perfered to keep the temperature of the board in check, I have never really run into the problem of overheating while using the Raspberry Pi as a simple file server (including for hosting videos that are accessible over my local network).
Thus, I have been perfectly content with the fanless [Flirc Raspberry Pi 4 Case](https://flirc.tv/more/raspberry-pi-4-case) with some heat sinks added directly to the board.
Note that running any piece of electronic equipment at a high temperature will shorten its lifespan, so choosing to go fanless may not be right for you.

## Power supply
Buying the [official power supply](https://www.raspberrypi.org/products/type-c-power-supply/) is definitely the prefered way to go, but I personally really like third-party cables that contain an on--off switch (instead of unplugging and replugging the power cord to reboot the machine) and the use of a standard USB charger.
Note that if you go this route, you will absolutely need to ensure that the USB power supply meet the minimum requirements (e.g. 3.0A DC output).

## SATA drive enclosure and hard drives
As the microSD card will not be storing any of the data accessible via the file server, you will need to also attach storage devices to the Raspberry Pi.
For this, I am using standard 3.5" SATA drives in a Mediasonic ProBox (sold under the name fantec in other parts of the world) drive enclosure connected via USB 3 to the Raspberry Pi itself (thus technically making it [direct-attached storage (DAS)](https://en.wikipedia.org/wiki/Direct-attached_storage) and not [network-attached storage (NAS)](https://en.wikipedia.org/wiki/Network-attached_storage)).
Before I put any new hard drive into long-term use, I run it through [a grueling `badblocks` test](https://github.com/Spearfoot/disk-burnin-and-testing) to make sure that there are no nasty surprises waiting for me in a few days, weeks or months regarding data corruption.
After this, the drives will need to be [formatted](https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux) using a standard file system; I perfer [ext4](https://en.wikipedia.org/wiki/Ext4) for maximum compatibility and its journaling capabilities.

While I am not so sure about the long-term use of USB in a server setting, USB-A is rated for [10,000 cycles of plugging and unplugging](https://web.archive.org/web/20150427001335/http://www.usb.org/developers/docs/devclass_docs/CabConn20.pdf) (p. 6), so it might just be good enough for home use.


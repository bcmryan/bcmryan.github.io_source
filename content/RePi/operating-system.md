+++
title = "Operating system"
weight = "30"
+++

The operating system is a computer's main intersection point between hardware, software and firmware.
In this build we will mainly be communicating with the Raspberry Pi server's operating system through the use of a [terminal emulator](https://en.wikipedia.org/wiki/Terminal_emulator).
We will for instance tell the operating system to download or install applications (or lower-level firmware or kernel updates), thus affecting the system's software, and even shut down the server via a command, thus physically affecting the hardware's state by use of a terminal emulator.

For me, there are only two real contendors for the choice of an operating system on a Raspberry Pi when it should be used as a server: [Ubuntu Server](https://ubuntu.com/download/raspberry-pi) and [Raspberry Pi OS](https://www.raspberrypi.org/software/operating-systems/).
As both operating systems are based on [Debian](https://en.wikipedia.org/wiki/Debian), they function relatively similarly and run much the same software, so the choice as to which operating system to put into production comes down to hardware support.

Raspberry Pi OS was ultimately chosen for this build, as its direct support from the Raspberry Pi Foundation means that it is custom distribution tailored to work with the Pi.
While running Ubuntu Server on the Raspberry Pi is absolutely a possibility, you may run into hiccups along the way.
One such hiccup that heavily influenced my decision to base my server on Raspberry Pi OS is that hardware acceleration for video transcoding with the Jellyfin media server which will be installed below [does not seem to work when using something other than the 32-bit distribution of Raspberry Pi OS](https://github.com/raspberrypi/firmware/issues/1366#issuecomment-612902082).
While I have run Ubuntu Server on my Raspberry Pi for some time and I perfer that I am working with the ([probably, although no exact numbers are shared](https://ubuntu.com/desktop/statistics)) most used Linux variant with a 64-bit architecture ([which has had to fight its way to default status in the Linux world](https://www.phoronix.com/scan.php?page=news_item&px=The-32-bit-Ubuntu-20.04-Debs), this server build is not about hacking together working solutions but rather coming up with the quickest way to create a stable server on a Raspberry Pi.
For that, it is clear that [Raspberry Pi users are using Raspberry Pi OS](https://rpi-imager-stats.raspberrypi.org/), so instead of fighting that trend, I have decided to embrace it, with the hope that the [64-bit beta version](https://www.raspberrypi.org/forums/viewtopic.php?t=275370) will soon become the standard while still supporting everything that the standard 32-bit version does.

Continuing below, I will show my exact commands using my Ubuntu desktop computer with my desktop labeled as `bcmryan@desktop` and my Raspberry Pi OS install labeled as `pi@raspberrypi` (until changed to `pi@repi` below).
Note also that my path is given immediately following the colon (`:`), with a tilde (`~`) representing my `/home` directory; a following dollar sign (`$`) means that the command is run with standard privledges, while a pound sign (`#`) means that the command is being run as `root`.

Before beginning, it is always a good idea to make sure that you have the latest updates downloaded and installed. The following command means that I will first update (download) all the newest packages without confirmation (`--yes`) and then fully upgrade (install all downloaded packages; similarly without confirmation, `--yes`).
Ass two ampersands (`&&`) are being used in this combined command, the second command will only start once the first command has succesfully completed (thus all packages must be successfully downloaded before the installation process can begin).
Since upgrading and updating both require `sudo` (temporary `root`) privledges, I will be prompted for my password (which I input and then confirm with the Enter key; note that you do not get feedback as to how many keys you have entered while inputing a password).

`bcmryan@desktop:~$ sudo apt update --yes && sudo apt full-upgrade --yes`

Now I will download the lastest version of Raspberry Pi OS (which itself is based on Debian 10 Buster) using a permanent link to the most updated version (note that this link may need to change once Debian releases a new version and/or if Raspberry Pi OS's name is changed).
Note the options that are used with [`curl`](https://en.wikipedia.org/wiki/CURL): `--location` follows the permanent link to its final location; `--output` following a path ending with a filename and extension determines where to save the file that is being downloaded (in this case an `.img` image file of Raspberry Pi OS within a compressed `.zip` file); and finally `--write-out %{url_effective}` determines the URL that was last fetched (i.e. not the redirect page, but the actual final file to be downloaded).
Before entering the command install `curl` to make sure it is on your system.

`bcmryan@desktop:~$ sudo apt install --yes curl`

`bcmryan@desktop:~$ curl --location --output ~/Downloads/rpi.zip --write-out %{url_effective} https://downloads.raspberrypi.org/raspios_lite_armhf_latest`

The on-screen output (standard output, [stdout](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout))) first shows the redirect to the final URL and then the downloading of the actual file, which takes significantly longer.

Now with image within a `rpi.zip` file in your `Downloads` directory, it is time to write the image to the microSD card.
(For extension documentation about writing the image to a microSD card, see the [official documentation](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md).
Note that the documentation also includes information on verifying that the image was properly transfered, which is not included below.)

With your microSD card inserted into your computer (I am using a microSD-to-USB converter, but if you have a SD card slot, a microSD-to-SD converter might work for you), use the `lsblk` (list block devices) command to see if it is recognized and auto-mounted on your desktop.

`bcmryan@desktop:~$ lsblk`

Near the bottom of the list of my block devices, I see that there is a 29.7 GB device that I do not otherwise recognize at `/dev/sdd` with its partition mounted at `/dev/sdd1`.
This is my microSD card.
Before I proceed, I will need to unmount the device, as it is considered unsafe to run [`dd`](https://en.wikipedia.org/wiki/Dd_(Unix)) on a mounted partition, since other activity (e.g. reading from the device) may otherwise be taking place on that device while `dd` is writing to it.
That would be bad and could result in corruption.
To unmount a disk, I must specify the mount location, which for me is located in my `/media` mount location following the pattern `/media/bcmryan/1234-5678` (with 1234-5678 specifying the eight-digit [universally unique identifier](https://en.wikipedia.org/wiki/Universally_unique_identifier), UUID, of a FAT-formatted device).
Note that this requires elevated privledges.

`bcmryan@desktop:~$ sudo umount /media/bcmryan/1234-5678`

After umounting the partition, run `lsblk` again to confirm that it was succesfully unmounted.

`bcmryan@desktop:~$ lsblk`

I note that the row beginning with `sdd1` no longer has a mount point in its final column. That means that the microSD card has been unmounted.
Before we can transfer the data from the computer to the microSD card, we first need to install `unzip` so that the image within the `.zip` file can be properly extracted.

`bcmryan@desktop:~$ sudo apt install --yes unzip`

**Note that the following step uses the infamous `dd` program.
Please make sure you know what you are doing before continuing.
If you do not feel comfortable continuing, do not.**
Now it is time to write the image file to the microSD card.
The following commmand uses `unzip -p` to [pipe](https://en.wikipedia.org/wiki/Pipeline_(Unix)) the contents of `~/Downloads/rpi.zip` to the standard input ([stdin](https://en.wikipedia.org/wiki/Standard_streams#Standard_input_(stdin))) of `dd` which then writes the image to the disk specified with `of` (output) and with a blocksize (`bs`) of 4 MB.
To make sure that the contents have been properly copied, `dd` also flushes (`conv=fsync`) all data.
The option `status=progress` displays the status of the writing of the image in the terminal.
Finally, to double-check that the contents have fully been flushed, `&& sync` is also run (with `&&` telling the command to only run once the previous command has been properly completed).
**Remember to change your command to the proper disk location (i.e. change /dev/sdd to your microSD card's location in /dev).
The device location I have entered is /dev/sdd (and not the partition location of /dev/sdd1).
Modify your location accordingly, otherwise it may result in data loss.**

`bcmryan@desktop:~$ unzip -p ~/Downloads/rpi.zip | sudo dd of=/dev/sdd bs=4M conv=fsync status=progress && sync`

In total, this process may take a few minutes.
Once this process is complete (returning you back to a blank command line), run `lsblk` again to see where the newly written microSD card is now located and/or mounted.

`bcmryan@desktop:~$ lsblk`

Knowing that the two partitions of the microSD card are located at `/media/bcmryan/boot` and `/media/bcmryan/rootfs` on my computer named `desktop`, I can move onto the next step: preparing the microSD card for its first boot.


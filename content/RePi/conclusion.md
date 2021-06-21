+++
title = "Conclusion"
weight = "90"
date = 2021-06-15
+++

So there you have it.
Welcome to your fully functional file and media server.

Let us review what you did in this tutorial.
First, you downloaded Raspberry Pi OS and flashed it to a microSD card.
From your desktop, you then set up SSH and gave yourself permanent access to the terminal emulator of the Raspberry Pi with an SSH key.
Additionally, you set static IP addresses on both your desktop and Raspberry Pi so that you do have to fumble around with dynamically allocated addresses.
For this same reason, you also set the Raspberry Pi's hostname.
Next, in order to control access to your files, you created different users with different permissions.
With mergerfs, you then merged externally attached hard drives into a single directory.
This single directory was then shared across the network via Samba, with all devices on the local network given read-only access and a single device given read-write access.
Using Docker and Docker Compose, a containerized version of the Jellyfin media server was created with read-only access to the merged directory so as to prevent data loss.
Using `apt` and a custom script, you can now keep your system up to date and your backups secured.

By the standards of most any home user, this is a pretty impressive system, and it is just as amazing that it was built with readily available hardware and [free and open-source](https://en.wikipedia.org/wiki/Free_and_open-source_software) software.
Remembering that this project was about creating a modular and reproducible server, I have a challenge for you: break it.
You read that correctly.
Rip out a hard drive; delete a couple packages; try to destroy your Jellyfin Docker container, or just delete your entire Docker Compose config file.
The point of this project is not only to create a file and media server but also to help you understand that modularity and reproducibility are fundamental building blocks of a good computing experience.
You should not be afraid of your system, and you should not be left out in the dark when you inevitably need to restore from a backup or need to just completely wipe a machine.
So, build your machine (and your network) with the knowledge that when (not if) the worst happens, you are prepared.

Personally, I am not finished with this project.
My next step is create a script that can run directly from your desktop that will allow you to define your SSH key, static IP address, hostname, etc. before you even powering on your Raspberry Pi for the first time.
A following script should then install all necessary packages, set up user accounts, etc. directly when executed directly from the Raspberry Pi.
My hope is that the knowledge you have gained from completing a build command by command will empower you to be able to use these scripts to restore your Raspberry Pi server following a catastrophic (or even not-so-catastrophic such as a major version upgrade) failure.

## Acknowledgments
Along with the great [official documentation](https://www.raspberrypi.org/documentation/) from the Raspberry Pi Foundation and the links cited throughout, this project has been greatly influenced by the [Perfect Media Server](https://perfectmediaserver.com/) project as well as general knowledge picked up over the years on the [r/DataHoarder](https://www.reddit.com/r/DataHoarder/) subreddit and in technical documentation and in forum posts in general.

## Troubleshooting
I hope that this guide was thorough enough, but if you still have any other questions, you may find some helpful hints below.

### What can I do if I do not have access to a Linux desktop?
The commands that are given throughout this guide are based on Linux.
While I am nearly certain that they should also work on macOS via the Terminal app, I have not tested them.
On Windows, you can try Windows Subsystem for Linux and/or PowerShell, but I cannot guarantee compatibility.
I strongly urge you to [create a Linux live USB stick](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows) so you can follow along as intended.

### How can I get even bleeding-edge firmware and kernel updates?
You can update the Raspberry Pi's firmware with [`rpi-update`](https://github.com/Hexxeh/rpi-update).
(Note that upgrading the firmware is not without risk.
Although [Jellyfin's official documentation does consider this important for hardware acceleration](https://jellyfin.org/docs/general/administration/hardware-acceleration.html#raspberry-pi-3-and-4), it is not absolutely necessary.
Nevertheless, using `rpi-update` will allow you to have the newest kernel and firmware updates available.
Proceed with caution.)

`pi@raspberrypi:~$ sudo apt install --yes rpi-update && sudo rpi-update`

You will be asked to confirm that you are comfortable with this upgrade.
If you are, enter `y`.
The firmware and kernel updates will then be applied.
You system should then be on the absolute bleeding edge.

### I keep trying to reach raspberrypi.local, but it just will not resolve.
If you are having trouble resolving raspberrypi.local, you can try to use the process of elimination in order to determine your dynamic IP address.
To do this, ping every currently used IP address on your local network to see what is in use with the program [`Nmap`](https://en.wikipedia.org/wiki/Nmap), which you will first need to install.

`bcmryan@desktop:~$ sudo apt install --yes nmap`

From there, use `Nmap` to check which IP addresses are being used in your subnet.
I know from its initial setup that my router can be accessed at `192.168.0.1`, so since my Raspberry Pi and desktop are directly connected to my router, I know that both devices should have IP addresses between `192.168.0.2` and `192.168.0.255` (a total of 254 addresses, which is definitely enough for all the devices in a standard home network).
The command used for this is pretty easy: `-sn` performs a ping scan, which means that the program is just seeing if the IP is in use (akin to ringing anyone's doorbell in an apartment building and seeing who responds); `192.168.0.0-255` simply defines the range of all IP addresses to be scanned.

`bcmryan@desktop:~$ nmap -sn 192.168.0.0-255`

A possible result might say that all addresses between 192.168.0.1 (typically a router) and 192.168.0.8 are in use.
(It is assumed that all of these are dynamic IP addresses, given on a first-come-first-served basis to whatever desktop, smartphone, tablet, smart device, etc. signed onto the network first.)

Since you do not know which IP address is used by the Raspberry Pi, try to log in to each address via SSH with the default username and password until you find it.
For instance, start with 192.168.0.2 and work your way up to 192.168.0.8.

`bcmryan@desktop:~$ ssh pi@192.168.0.2`

By continuing to repeat this command (click the up arrow on your keyboard to get the most recent command) with a different last number on the IP address, you should be able to connect with the Raspberry Pi at its dynamic IP address.

### I am happy with just using Samba. Is installing Jellyfin absolutely necessary?
This is your system.
You can do with it whatever you please.
I personally used a simple Samba share for at least a year before I saw the need to install Jellyfin.
Adding the nice GUI of Jellyfin increased the [appeal to my family members](https://en.wikipedia.org/wiki/Wife_acceptance_factor) who did not have the same urge to browse through directories to find backups of discs that we physically own (often causing them to just use the physical DVD or Blu-Ray instead of connecting to Samba at all) that I did.
Additionally, the ability to create users and track what has been watched has been an absolute godsend.

But, I am also the first to admit that Jellyfin is not perfect.
ISO (i.e. an image of a full disc instead of ripped individual titles saved, for instance, as MKV files) playback is lacking to say the least (which is particularly interesting considering that Kodi and VLC, two other open-source projects, both play back the same files with no problems), and I honestly do not know if a file is transcoding correctly or not (or, for instance, if an uncompressed rip of a UHD Blu-Ray might just be too much to transcode on a Raspberry Pi 4 with 4 GB of RAM).
But to me, the pros outweigh the cons.
And since my media files are available to me on my Samba share anyway, I can always just play any ISO file directly in VLC, and through the [Kodi plugin](https://jellyfin.org/docs/general/clients/kodi.html#jellyfin-for-kodi), I could also just play it using that interface.
That is truly the power of open-source software.
It may not be perfect, but there are usually ways to make it do what you want if you put a bit of time into managing it.

For this tutorial, Jellyfin served two purposes: I wanted to exemplify how easy it is to set up a container with Docker Compose, and I personally wanted to document installing Jellyfin with hardware acceleration for future use.
If Jellyfin does not suit your needs, that is perfectly fine.
There are many [other great self-hosted applications for you to try](https://perfectmediaserver.com/day-two/top10apps/).

### Is there a graphical way to set system settings?
For changing settings like the hostname in Raspberry Pi OS, you can use the powerful tool `raspi-config`.
As the program changes system settings, it requires elevated privileges.

`pi@raspberrypi:~$ sudo raspi-config`

For changing the hostname, for example, make sure that the line `1 System Options` is highlighted, and hit enter.
Now hit the down arrow until `S4 Hostname` is highlighted, and again hit enter.
You will now be given a warning about valid characters in a hostname.
Make sure that yours complies.
Enter a valid hostname like repi, and push the down arrow key so that `<Ok>` is highlighted, and hit enter.

The new hostname should now be set.
From here, you can choose to look at other settings (such as locale and timezone in `5 Localisation Options`).
Here again, [the official documentation](https://www.raspberrypi.org/documentation/configuration/raspi-config.md) is very extensive.
Once you are finished, select `<Finish>` with the right arrow key, and hit enter to exit.

### I chose the exFAT file system so that my drive could be easily read by Windows, macOS and Linux, but I am having trouble getting it to properly work.
While the choice of exFAT as a file system makes sense for an external hard drive that is used with multiple operating systems, if the drive is being used in a file server with Samba, there will be no problem reading or writing to a file system like ext4, which has the benefit of [journaling](https://en.wikipedia.org/wiki/Journaling_file_system).

That said, if you would still like to use exFAT (as I did for many months before transitioning my files over to ext4), you will need to install `exfat-fuse` and `exfat-utils` (`sudo apt install --yes exfat-fuse exfat-utils`) and add an entry to `/etc/fstab` such as `UUID=1234-5678 /mnt/diskMovies1 exfat-fuse defaults,uid=200,gid=200 0 0`.
Note that UID and GID (for user storagerw and group storage) are defined here, as I personally have had issues with permissions with exFAT (an alternative would be to use 1000 and 1000 for the current user and current group if problems still persist).

### When turning on the hard drive enclosure and Raspberry Pi, sometimes the drives do not properly mount.
I believe hard drives within an enclosure not properly mounting is a problem with the drives timing out when they are not mounted quickly enough.
Thus, before the operating system has had a chance to fully start, the enclosure has already stopped spinning the drives, since the drives have not yet been mounted.
To avoid this problem, wait about 10 seconds from the time that you turn on the Raspberry Pi to turn on the hard drive enclosure.
This allows the drives to be available and spinning when the operating system is ready to mount everything as listed in `/etc/fstab`.
If you wait too long (or if you need to turn on the enclosure after it has timed out and turned itself off), these drives will not be automatically mounted.
In this case, simply use the `mount` command with elevated privileges and the `--all` option to mount everything as listed in `/etc/fstab` as it would be at startup.

`pi@repi:~$ sudo mount --all`
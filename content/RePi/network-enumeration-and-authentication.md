+++
title = "Network enumeration and authentication"
weight = "40"
+++

Network enumeration and authentication are all about knowing which device is which on a network and proving to a device that a user is who they say there are.

## SSH
In order to issue commands to a server over the network from our local computer, we rely on the Secure Shell Protocol ([SSH](https://en.wikipedia.org/wiki/Secure_Shell_Protocol)).

As SSH is not enabled by default in Raspberry Pi OS, it must be activated before the first boot.
[To do this])(https://www.raspberrypi.org/documentation/remote-access/ssh/README.md), we are going to [`touch`](https://en.wikipedia.org/wiki/Touch_(command)) (in this case create; if a file were to already exist at that location, it would have its access and/or modification date updated) a blank file on the `boot` partition of our microSD card (which is located in this example at `/media/bcmryan/boot`).

`bcmryan@desktop:~$ touch ssh /media/bcmryan/boot`

It is now time to unmount and physically remove the microSD card from your computer.
Remember to use your correct mount points.

`bcmryan@desktop:~$ sudo umount /media/bcmryan/boot /media/bcmryan/rootfs`

Confirm with `lsblk` that the devices have been unmounted.
To be extra safe before removing the microSD card, it may be worth it to fully power off the USB device with [`udisksctl`](http://manpages.ubuntu.com/manpages/bionic/man1/udisksctl.1.html) before physically removing it.

`bcmryan@desktop:~$ udisksctl power-off --block-device /dev/sdd`

## Network enumeration
Now it is time to remove insert the microSD card into the Raspberry Pi for its first boot.
There is just one problem: we know that SSH is enabled and that we should be able to access it, but we do not know the actual local IP address of the server.
No worries, there is a simple Linux command line tool that will help us figure out the IP addresses that are being used on the local network.
From there, you should be able to figure out the IP address that belongs to the Pi.

Now I am going to ping raspberrypi.local, and [as long as my desktop supports mDNS](https://www.raspberrypi.org/documentation/remote-access/ip-address.md), I should be able to find my Raspberry Pi's IP address.
(If you get an error about the hostname not being able to be resolved, see the FAQs below.)

`bcmryan@desktop:~$ ping raspberrypi.local`

Great.
Now I see that my Rasbperry Pi has been assinged the dynamic IP address 192.168.0.5.
([Dynamic IP addresses](https://en.wikipedia.org/wiki/IP_address#IP_address_assignment) are given on a first-come-first-served basis, starting from the lowest available number, such as 192.168.0.2, since 192.168.0.1 is usually taken by a router, whereas a static IP address is assigned to a specific device and can be anywhere in the available range of the [subnet](https://en.wikipedia.org/wiki/Subnetwork), from 192.168.0.2 to 192.168.0.255.
The benefits to having a static IP adddress are that you always know the exact location of your device on the network and that no other device will be assigned that address, thus making something such as logging in via SSH much easier.) 
Before I can start trying to log in via SSH onto the Raspberry Pi, I will first need to enable SSH.
I do so by installing `openssh-server`.
From this point on, you should be able to relatively easily use a recent macOS (through the [Terminal app](https://en.wikipedia.org/wiki/Terminal_(macOS))) or Windows 10 (via [Windows Subsystem for Linux](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux) and/or [PowerShell](https://en.wikipedia.org/wiki/PowerShell)) computer, as both systems now support OpenSSH out of the box (but if you have problems, the rest of the tutorial is still geared towards Linux, so you may have to do a bit of troubleshooting to solve your specific problem).

`bcmryan@desktop:~$ sudo apt install --yes openssh-server`

The command to SSH into another machine is pretty self explanatory: `ssh` is followed by the username `pi` followed by `@`  (this could be left out if the username were the same as my current user on my desktop, i.e. `bcmryan`) before the local IP address.

`bcmryan@desktop:~$ ssh pi@192.168.0.5`

(An alternative to using the IP address here would be to use the hostname followed by .local, `raspberrypi.local`, i.e. `bcmryan@desktop:~$ ssh pi@raspberrypi.local`.
I would not recommend this though as you may run into issues resolving the hostname on other operating systems.
For instance, this method does not work when trying to SSH into the Raspberry Pi while using [Termux on Android](https://f-droid.org/en/packages/com.termux/).)

I am asked if I want to continue, and I physically type in `yes`.

From here, I am prompted for the password (which is `raspberry`).

If you have followed me this far, congratulations, you have succesfully logged into a device on your local network via SSH.
And you did so with a terminal, you little hacker, you.

Before we get any futher, the first thing we should do is change the password for the user pi on the Raspberry Pi.
To do so, simply follow the prompt you are first shown while logging in with the command `passwd`.

`pi@repi:~$ passwd`

Enter your current password (`raspberry`) and a new password twice (for this documentation, I will be using `passwordpi`, as there will be a few passwords that will be set throughout; of course you should use a much more secure password on your machine).

Now, before we do anything else, it is important to make sure that this newly installed system is fully up to date.
To do this, we first can update our system via `apt`.

`pi@raspberrypi:~$ sudo apt update --yes && sudo apt full-upgrade --yes`

You may have noticed that you are not prompted for a password when connected to your server via SSH.
It is definitely nice to not need to enter it so many times, but you also need to remember that entering a password is a security feature, so you may want to double-check even more thoroughly before entering a command with `sudo`.

[Network enumeration](https://en.wikipedia.org/wiki/Network_enumeration) is the act of gathering information about the devices, users, etc. within a network, and with a bit of planning, you can make the job much easier for yourself in the future.
For instance, coming up with a good [network naming scheme](https://en.wikipedia.org/wiki/Computer_network_naming_scheme) will allow you to give your devices a hostname (the `desktop` or `raspberrypi` that follows the username in the commands throughout) and/or local static IP address (`192.168.0.XYZ`) that is meaningful to you and/or your organization.

My personal hostnames are based on [genus names of dinosaurs](https://en.wikipedia.org/wiki/List_of_dinosaur_genera) that end in -saurus.
My thinking behind this goes that my personal network will never run out of the 1000 or so genus names of Dinosauria ending in -saurus.
Additionally, I can give my devices names that fit characteristics of that device: my desktop is actually called `tyrannosaurus`, as it is the king of all devices on my network.
Were I to upgrade my computer, I could continue calling it `tyrannosaurus`, or I could choose to retire the name ([because there might be a limit to how many components can be changed before an object is no longer the same object](https://en.wikipedia.org/wiki/Ship_of_Theseus)) and call it [`tarbosaurus`](https://en.wikipedia.org/wiki/Tarbosaurus) or [`zhuchengtyrannus`](https://en.wikipedia.org/wiki/Zhuchengtyrannus), [genera also belonging to the taxonomic family of the tyrant king](https://en.wikipedia.org/wiki/Tyrannosauridae).
This is very clearly a silly excercise that I have undertaken, but I found it to be rewarding, and it has kept me from needing to rename my computer `desktopoffice`, `ubuntu2004` or `bcmryan1` whenever a physical machine changes location, operating system or owner.
Of course on a larger scale, this is untenable.
In this regard, do as I say, not as I do.
For a home with a few connected devices, naming machines based on fun charactersitics might be feasible, but once printers, routers, terminals, etc. are involved in a multinational corporation with branches on different levels, inside different buildings and within different cities, it is absolutely vital to come up with a more sustainable naming scheme, potentially based on device type and location (preferably not owner, as this is bound to change much more frequently than an office building's address).

Similarly, all static IP addresses within my network have some kind of meaning.
I have reserved the first 20 IP addresses (up to 192.168.0.19) for dynamic addresses, including all those devices which might hop on and off without having been assigned a static IP address (e.g. a tablet which I am testing out but which has not yet been configured).
From there, every current (and potentially future) member of my household gets 10 IP addresses.
This is where 192.168.0.20 and 192.168.0.25 from earlier come into play: those are my my main desktop (20) and my phone (25).
These personal addresses end at 192.168.0.99, and I have reservered 192.168.0.100 for this server.
I hope that this IP addressing scheme will continue to serve me well, as I have yet to run into any problems.
Of course, this would be a completely useless excercise if I did not have proper documentation to tell me which IP address is allocated to which person (dynamic, bcmryan, etc.), under which hostname (e.g. `tyrannosaurus`) on which device (e.g. processor type in desktop computer).
I have found this information incredibly easy to store in a simple spreadsheet on my computer so that I never have to wonder what the IP address is of any computer I am trying to connect to via SSH.
How to change a computer's or smartphone's static IP address will of course depend on your device's operating system, be it [Ubuntu](https://help.ubuntu.com/stable/ubuntu-help/net-manual.html.en), [Windows](https://support.microsoft.com/en-us/windows/change-tcp-ip-settings-bd0a07af-15f5-cd6a-363f-ca2b6f391ace), [macOS](https://support.apple.com/guide/mac-help/use-dhcp-or-a-manual-ip-address-on-mac-mchlp2718/11.0/mac/11.0), [iOS](https://support.apple.com/en-us/HT202693) or [Android](https://www.businessinsider.com/how-to-change-ip-address-on-android).
(If you have a link to official Android documentation regarding setting an IP address, please let me know, and I will include it here.)

Now that we have logged into the Raspberry Pi, it is a good time to change the hostname and set a static IP address so that we always know where to find the machine on our local network.

## Hostname
Changing a hostname can be completed using the very easy-to-use [hostnamectl](http://manpages.ubuntu.com/manpages/focal/man1/hostnamectl.1.html) on both Raspberry Pi OS and Ubuntu Server.
Simply enter `hostnamectl set-hostname` followed by the new hostname.

`pi@raspberrypi:~$ hostnamectl set-hostname repi`

You will be asked for your password, and after you enter it, the hostname will be changed, although you will notice that the old hostname is still in use on the command line once you exit.
That is ok.
It will change once the system is rebooted after we change the static IP address.

To make sure that the hostname has been changed throughout the machine, [you should also edit the file /etc/hosts](https://www.cyberciti.biz/faq/ubuntu-change-hostname-command/) and update the bottommost line with the old hostname.
To do this, we will use a CLI text editor such as `nano` with `sudo` privileges.
([Arguments can be very heated regarding text editors!](https://en.wikipedia.org/wiki/Editor_war))

`pi@raspberrypi:~$ sudo nano /etc/hosts`

Use your down arrow key to select the appropriate line and the right arrow key to edit it.
Delete raspberrypi and replace it with repi.
Save the new document with Ctrl+S, and exit with Ctrl+X.

## Static IP
As I have stated before, by default, the Raspberry Pi is allocated a dynamic IP address.
The good news is that [Raspberry Pi OS makes it relatively straightforward to set a static IP address](https://www.raspberrypi.org/documentation/configuration/tcpip/README.md). (It should be noted that this is different than [setting an IP address on Ubuntu Server](https://ubuntu.com/server/docs/network-configuration).)

But to show the power of the command line, we will instead just text append (add text to the end of a text file) using [`tee`](https://en.wikipedia.org/wiki/Tee_(command)).
The following command will append the configuration via `tee -a` with `sudo` privileges to `/etc/dhcpcd.conf` while defining `eth0` (Ethernet) as the interface we would like to use with the static IP address 192.168.0.100 with a router at 192.168.0.1 and Google's DNS server at 8.8.8.8.
A title can be given to the configuration with a title (Static IP configuration) following a hash (#), which notes that what follows on that line is a comment.
END on both sides of new lines act as bookmarks to show which text should be appended, and the two less-than signs point to the file to which the text should be appended.

```
pi@raspberrypi:~$ sudo tee -a /etc/dhcpcd.conf << END

# Static IP configuration:
interface eth0
static ip_address=192.168.0.100/24    
static routers=192.168.0.1
static domain_name_servers=8.8.8.8
END
```

After entering the command, you will be warned that the host raspberrypi does not exist.
That is completely accurate, as we just changed the hostname, but do not worry, this is not a problem, and your new hostname and static IP address will work fine.

To verify that the text has been correctly added to `/etc/dhcpcd.conf`, you can see the files contents by using the [`cat`](https://en.wikipedia.org/wiki/Cat_(Unix)) (concatenate) command.

`pi@raspberrypi:~$ cat /etc/dhcpcd.conf`

From here, we need to reboot the Raspberry Pi to have all changes take effect.

`pi@raspberrypi:~$ sudo reboot`

Before we try logging onto the Raspberry Pi for the first time with the new static IP address (192.168.0.100), it is a good idea to delete fingerprint that we accepted for the old dynamic IP address (192.168.0.5).
Otherwise, there may be problems in the future if you ever try to SSH into another machine that shares the same dynamically given IP address.
**This is potentially dangerous if you use SSH with multiple devices, so make sure that you know which fingerprint of which IP address you are deleting.**

`bcmryan@desktop:~$ ssh-keygen -f "/home/bcmryan/.ssh/known_hosts" -R "192.168.0.5"`

Now we it is time to cross our fingers and try to log onto the Raspberry Pi with a static IP address.

`bcmryan@desktop:~$ ssh pi@192.168.0.100`

To accept the new fingerprint, physically type in `yes`.
To complete the login, enter in the password.

And it was successful!
The Raspberry Pi is now on its statically given IP address of 192.168.0.100, and the command line now reads `pi@repi:~$`.
What is also neat is that the IP address of the desktop from which you connected via SSH to the Raspberry Pi is also shown.
In my case, that is 192.168.0.20.
Having a logical system in place for IP addresses tells me that I last succesfully logged in from the desktop that I am working from.

## SSH keys
We usually authenticate our identity using a password, but that is not the only way (as you may know from certain applications that require a fingerprint or [multi-factor authentication](https://en.wikipedia.org/wiki/Multi-factor_authentication) for which you for instance have to verify your identity with a password and a SMS or email code).
Authentication for SSH can be done through at least two different methods, by use of a password and/or a key.
With password authentication, you are tell your desktop through a terminal command that you would like to log on to a server (or other remote machine) at a specific IP address with a certain username.
Just like when entering a command with `sudo`, you are asked for the password of that other user on that other machine.
With authentication via SSH keys, it is not required that you provide a password (although you can enable two-factor authentication requiring a password).
Instead, a private key (which is just a text file with a few hundred random characters) saved onto your desktop sees if it fits to a public key saved on the server.
Although these two keys are not identical, if a specific algorithm determines that they are a match, you can log on via your desktop to to the server.

In this section, we are going to share SSH keys between the desktop and Raspberry Pi server so that we no longer need to type in a password following the command `ssh pi@192.168.0.100`.
Following the [official documentation](https://www.raspberrypi.org/documentation/remote-access/ssh/passwordless.md) we are going to first generate new SSH keys on the desktop with `ssh-keygen`.

`bcmryan@desktop:~$ ssh-keygen`

You will be asked where to save the key.
Hit enter to save it in its default location (this will create a public key at `~/.ssh/id_rsa.pub` and a private key at `~/.ssh/id_rsa` on the desktop).
You will also be asked for a passphrase.
You may choose to not enter anything here (it encrypts the key) and accept the default setting by again hitting enter.
Enter your passphrase again and/or hit enter to continue.

Now you will share public key with your server with `ssh-copy-id`.
This will log you into the user (`pi`) given at the IP address (192.168.0.100).
It is of course important that the static IP address is set at this point so that the desktop shares the key with the correct server and that its address does not change, which would cause problems while attempting to connect at a later time.

`bcmryan@desktop:~$ ssh-copy-id pi@192.168.0.100`

Once we enter the command, we are prompted for the password of user `pi` on 192.168.0.100.
This is the password that was set above, i.e. `passwordpi` (thus adding the contents of the public key on the desktop at `~/.ssh/id_rsa.pub` to the Raspberry Pi at `~/.ssh/authorized_keys`).

From here, you should be able to SSH into the Raspberry Pi server (provided that both the desktop and server have static IP addresses) without being prompted for a password.

`bcmryan@desktop:~$ ssh pi@192.168.0.100`

And it worked!
Although `ssh-copy-id` will not natively run on Windows 10 using PowerShell, there is a [work-around PowerShell one-line script](https://www.chrisjhart.com/Windows-10-ssh-copy-id/) that affectively does the same thing by copying the contents of `$env:USERPROFILE\.ssh\id_rsa.pub` (the Windows equivalent of `~\.ssh/id_rsa.pub`) on a Windows desktop and appending (`cat >>`) it to `~/.ssh/authorized keys` on the Raspberry Pi server.

`PS C:\Users\bcmryan> $env:USERPROFILE\.ssh\id_rsa.pub | ssh pi@192.168.0.100 "cat >> .ssh/authorized_keys"`

To recap, we can now access the terminal emulator as user `pi` of the Raspberry Pi server with a hostname of `repi` and a static IP of 192.168.0.100 across the local network using SSH without needing to enter a password.
That is pretty spectacular, and we are well on our way to being able to set up and administer a full server.


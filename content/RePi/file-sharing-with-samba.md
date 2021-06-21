+++
title = "File sharing with Samba"
weight = "60"
date = 2021-06-15
+++

With a central directory merging all of your large storage drives together, it is time to make this single mount point accessible over your local network with [Samba](https://en.wikipedia.org/wiki/Samba_(software)), a reimplementation of the [Server Message Block (SMB) protocol](https://en.wikipedia.org/wiki/Server_Message_Block).
Although SMB usually is targeted at use with Windows computers, as it allows for networking between Windows, Linux and macOS (all of which are represented in my house), it is what I use on my local network.
(A commonly used alternative in the Linux world is [Network File System, NFS](https://en.wikipedia.org/wiki/Network_File_System).)

Before you can begin to share the contents of your file system across your local network, you must first install `samba` via `apt`.
I have included lines above the install command to automatically answer the question "Modify smb.conf to use WINS settings from DHCP?" to "No" by use of [`debconf-set-selections`](http://manpages.ubuntu.com/manpages/bionic/man1/debconf-set-selections.1.html) (see https://unix.stackexchange.com/questions/546470/skip-prompt-when-installing-samba).

```
pi@repi:~$ echo "samba-common samba-common/workgroup string WORKGROUP" | sudo debconf-set-selections
pi@repi:~$ echo "samba-common samba-common/do_debconf boolean true" | sudo debconf-set-selections
pi@repi:~$ echo "samba-common samba-common/dhcp boolean false" | sudo debconf-set-selections
pi@repi:~$ sudo apt install --yes samba
```

As there are a number of dependencies to install, this may take a minute.
The next step is to save a copy of the original Samba config file at `/etc/samba/smb.conf`.
As this file acts more like documentation than configuration, it is best to keep a copy of it were you to ever need to look at it again.
To do so, use the `mv` (move; also for renaming files) command with elevated privileges to rename the file as `/etc/samba/smb.conf.bak` ([`.bak`](https://en.wikipedia.org/wiki/Bak_file) is a commonly used file extension for backups; also note that Linux still knows that this is a text file, unlike a Windows machine for which the file extension is important).

To write a new `/etc/samba/smb.conf`, we will again use `tee`.
The exact parameters are defined in the [official documentation](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html), so I will only summarize them here.
For global (thus for all Samba shares, even though only `[storage]` is defined here) the `map to guest = Bad User` option means that users, whether they have a valid or invalid username or no username at all (i.e. everyone on your local network), can be given guest access.
Logs are kept in the directory `/var/log/samba/`, although `log level = 1` means that extensive logging needed for debugging is not provided.
The share defined here is called `[storage]` and allows access to files at `path = /mnt/storage`.
The option `guest ok = yes` means that anyone can access the share (even those without an account), but `read only = yes` means that general users are given read-only access.
Read-write access, on the other hand, is given to user storagerw through the option `write list = storagerw`.
To avoid permissions issues, the options `force user = storagerw` and `force group = storage` are set for Samba so that the same user and group that owns `/mnt/storage` also has read-write access to the files in `/mnt/storage` via Samba; note, however, that this still requires a user to be on the `write list`, or else they are given read-only access.
Comments are inserted throughout with the use of a hash.

```
pi@repi:~$ sudo tee --append /etc/samba/smb.conf << END
[global]
        map to guest = Bad User
        log file = /var/log/samba/%m.log
        log level = 1

[storage]
        # Public read access and read-write access for storagerw.
        path = /mnt/storage
        guest ok = yes
        read only = yes
        write list = storagerw
        force user = storagerw
        force group = storage
END
```

As storagerw is now a defined user for the Samba share, you need to define this user's Samba password.
To do this, use the [`smbpasswd`](https://www.samba.org/samba/docs/current/man-html/smbpasswd.8.html) command with option `-a` to specify that this is a new password.
When prompted for a password (and when prompted to repeat it), for this tutorial, I am using `passwordsamba`.
But this is an absolutely terrible password, and you should use your own.

`pi@repi:~$ sudo smbpasswd -a storagerw`

For connecting to a Samba share, the following steps are applicable to a Linux operating system.
(Here are steps for [Windows 10](https://www.techrepublic.com/article/how-to-connect-to-linux-samba-shares-from-windows-10/) using `//192.168.0.100/storage` and [macOS](https://www.techrepublic.com/article/how-to-connect-your-macos-device-to-an-smb-share/) using `smb://192.168.0.100/storage`.
Again, I would appreciate links to official documentation if they exist.)
Here you will define your local user as the Samba user storagerw so that your desktop has read-write permissions to the Samba share.
Thus your desktop will be the only one able to create, modify or delete directories or folders on the Raspberry Pi's share at `/mnt/storage`, but all other users on all other computers will be able to access it with read-only permissions.
This may be useful depending on how your Samba share is to be used.
You, for instance, may want anyone on the network to be able to access all of the files in a specific share (in this tutorial, video files, but this is also applicable to something like family photos), but to protect against data loss, you may only want a single user on a single computer to be able to create, modify or delete those video files.
That way, if a family member comes over to visit, they can view your videos using their device on your local network, but you can be sure that no one will accidentally delete them.
Following [Ubuntu's documentation](https://wiki.ubuntu.com/MountWindowsSharesPermanently), the first step is to install `cifs-utils` via `apt`.
([The name CIFS actually refers to SMB 1](https://en.wikipedia.org/wiki/Server_Message_Block#SMB_/_CIFS_/_SMB1) from 1983 -- Windows 10 introduced SMB 3.1.1 -- but it seems to have stuck around.)

`bcmryan@desktop:~$ sudo apt install --yes cifs-utils`

From here, you need to create a mount point (directory) for the share using `mkdir` with elevated privileges.
I will be using `/mnt`, as this standard directory for mount points that do not change (unlike `/media` which is for removable devices like USB drives).

`bcmryan@desktop:~$ sudo mkdir /mnt/repi`

As this directory was created with elevated privileges, you will now need to change the owner (bcmryan) and group (bcmryan) (with the owner preceding the group, separated by a colon) to that of your local user account with `chown`.

`bcmryan@desktop:~$ sudo chown --recursive bcmryan:bcmryan /mnt/repi`

Next, you need to edit your desktop's `/etc/fstab` file to add your information about the mount.
Use `tee` to append the following text (making sure to edit your username).
The first three space-separated columns define the Samba share location and mount point on your desktop.
The third column defines `cifs` (i.e. Samba) as the protocol used, and the fourth column defines the following options ([which are more clearly defined in a German-language tutorial](https://www.elektronik-kompendium.de/sites/raspberry-pi/2102201.htm)): the options `noauto,nofail,x-systemd.automount,x-systemd.requires=network-online.target` help the system overcome problems with booting if the share is not available on the network at bootup; UID and GID are defined to avoid permissions issues, thus allowing the user to use the share as if it were a local disk; a save location for `credentials` allows the user to store their password in their `/home` directory so that not every system user can access it; and clearly defining the character set as `iocharset=utf8` helps to avoid issues with filenames in with non-[ASCII](https://en.wikipedia.org/wiki/ASCII) characters.
The fifth and sixth columns define dump and pass as 0.

```
bcmryan@desktop:~$ sudo tee --append /etc/fstab << END
//192.168.0.100/storage /mnt/repi cifs noauto,nofail,x-systemd.automount,x-systemd.requires=network-online.target,uid=1000,gid=1000,credentials=/home/bcmryan/.smbcredentials,iocharset=utf8 0 0
END
```

Now it is time to create `.smbcredentials` in your `/home` directory as defined in the entry in `/etc/fstab`.
For this, input the username and password for the Samba share as defined earlier on the Raspberry Pi.
(Again make sure to use your local username.)
Note here that `tee` does not need elevated privileges to save this file.

```
bcmryan@desktop:~$ tee --append /home/bcmryan/.smbcredentials << END
username=storagerw
password=passwordsamba
END
```

As storing a password in a plain-text file may allow others access to it using default permissions, use the [`chmod`](https://en.wikipedia.org/wiki/Chmod) (change mode) command to grant read-write permissions  to only the file's owner (6) and to no one in their group (0) or not in their group (0).

`bcmryan@desktop:~$ sudo chmod 600 /home/bcmryan/.smbcredentials`

You can verify this by listing all directories and files in your `/home` directory and confirming that `.smbcredentials` is preceded by `-rw-------` (read-write permissions only for the owner).

`bcmryan@desktop:~$ ls -l --all --human-readable /home/bcmryan`

Back on the Raspberry Pi, it is now time to restart [`smbd`](https://www.samba.org/samba/docs/current/man-html/smbd.8.html) (Samba daemon) with [`systemctl`](https://manpages.ubuntu.com/manpages/focal/man1/systemctl.1.html) so that the newly configured changes will enacted.
When asked for your password, you will need to enter the password of user pi, which is `passwordpi`.

`pi@repi:~$ systemctl restart smbd`

Now that the changes have been made on the Raspberry Pi, go back to your desktop and remount everything defined in `/etc/fstab` by completely restarting your machine with `reboot`.
(Note that `sudo mount --all` did not allow me to access the Samba share at `/mnt/repi`, so using the `reboot` command was required.)

`bcmryan@desktop:~$ reboot`

Once you log back into your desktop, you are able to access all files on `/mnt/repi` as if you were on the Raspberry Pi at `/mnt/storage`.
You can test this by listing all directories and files available at that mount point.

`bcmryan@desktop:~$ ls -l --all --human-readable /mnt/repi`
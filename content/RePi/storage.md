+++
title = "Storage"
weight = "50"
+++

With the base of the system set up, the next component to add your Raspberry Pi server to get it off the ground as a file server is storage.
As stated above, this tutorial follows the following setup: an array of hard drives (which do not need to be from the same manufacturer or of the same size; formatted in this example as ext4) inside an enclosure connected to the Raspberry Pi via USB 3.
In this section we will merge the indepedent file systems so as to make your Raspberry Pi show a continuous file system (thus, for instance, four 8 TB hard drives would appear as a single 32 TB file system making it much easier to point a media server to a single directory instead of needing to add multiple).

## User permissions
Before we even start with the process of merging file systems, it is a good idea to create two users and one group on the system that serve a singular purpose: to be own the files on the hard drives.
Without going too far into [file system permissions](https://en.wikipedia.org/wiki/File-system_permissions), our purpose is to create one user who has read and write access to the files and another user who can only read them.
These two users will belong to the same group.
This will allow us to grant read-write permissions to certain applications or when we specifically request them while defaulting to read-only permissions if we do not want to change anything.
This, for instance, can protect against data loss if a program, such as Jellyfin, which will be installed below, [attempts to delete user data](https://www.reddit.com/r/jellyfin/comments/f6hwsr/jellyfin_deleting_files/), due to either user or system error.
Regardless of who is at fault for a specific error, if a program does not have read-write access to a directory, it cannot modify it.

Using the [`groupadd`](http://manpages.ubuntu.com/manpages/focal/man8/groupadd.8.html) command, we are going to create the group storage with the specific GID (`--gid`; group identification number) 200 as a system (`--system`) group.
Having a preset GID comes in handy when granting specific programs (such as Jellyfin) the permissions of a certain group.
A [system group is not inherently different from a normal group](https://askubuntu.com/a/524010) but is conventionally used for technically processes, not users.
For instance, no human user will belong to the group storage, as it is solely to resolve or avoid permission issues, whereas a group called family might allow multiple users on a Linux computer to share photos with one another in a single directory, with each user being able to add, modify or deletes files therein.

`pi@repi:~$ sudo groupadd --system --gid 200 storage`

Similarly, we now create two users belonging to the group storage: storagerw (rw for read-write permissions; this one will be the owner of our shared directory) and storagero (ro for read-only permissions; this one will belong to the group storage and thus be given permission to access files but not modify or delete them).
Using the [`useradd`](http://manpages.ubuntu.com/manpages/focal/man8/useradd.8.html) command, for the same reasons as above we specify the UID (`--uid`; user identification number) and GID (`--gid`; so that both accounts belong within the group storage with a GID of 200) and set both accounts as system accounts (`--system`).
Note that user storagerw is given the UID 200, while user storagero is 201.
This should be somewhat memorable: the UID that matches the GID has read-write permissions, while the UID that is slightly different is a single number off.

`pi@repi:~$ sudo useradd --system --uid 200 --gid 200 storagerw && sudo useradd --system --uid 201 --gid 200 storagero`

So that no one attempts to log in using these user accounts (potentially to gain access to the system), we can lock them with the `usermod` command and the `--lock` option (repeated for both users).

`pi@repi:~$ sudo usermod --lock storagerw && sudo usermod --lock storagero`

To confirm that the new user accounts have been properly added to the group storage and to confirm their UID, we can use the `id` command followed by the user name (repeated for both users).

`pi@repi:~$ id storagerw && id storagero`

To verify that both accounts are system accounts (and thus not intended for human use), if you list the directories in `/home`, you will note that only user pi is given a `/home` directory.
For the [`ls`](https://en.wikipedia.org/wiki/Ls) (list) command, note that I perfer adding the options for horizontally (`l`) listing all (`a`; thus including hidden files) directories and files with human-readable (`h`; thus in kilobytes, megabytes, etc., not bytes) file sizes.

`pi@repi:~$ ls -lah /home`

## mergerfs
Following [The Perfect Media Server 2017](https://blog.linuxserver.io/2017/06/24/the-perfect-media-server-2017/) build guide (along with its [accompanying YouTube video](https://www.youtube.com/watch?v=tbCMfm-jJ5Y) and [general information on mergerfs](https://perfectmediaserver.com/tech-stack/mergerfs/); from which this section is very heavily inspired, including many commands), we install [mergerfs](https://github.com/trapexit/mergerfs), a union file system that allows for individual block storage devices (e.g. hard drives) to appear as a single device to the operating system and programs.
As stated in the [Storage](https://github.com/bcmryan/repi#storage), this allows for four individual drives, including those of different sizes and from different manufacturers, to be housed in a single directory.
Thus, instead of needing to add four different paths to a media server (i.e. one for each hard drive), you can add a single path which combines all the hard drives in one directory.

Now, aftering confirming that your enclosure with hard drives installed is physically connected to the Raspberry Pi, turn on the enclosure.
If you run `lsblk`, you will notice that your individual drives are not listed.
This is exactly as it should be because the devices have not been given a mount point.
To confirm that your Raspberry Pi sees the hard drives, you should instead run `sudo blkid` (block device identification).

`pi@repi:~$ sudo blkid`

Here you are shown all block devices connected to your Raspberry Pi first identified by their location as a `/dev` directory followed by information such as their label, UUID (an ID that identifies a specific block device and will probabistically never be used for another block device), type (i.e. file system), etc.
The devices that represent the attached hard drives with be listed as e.g. `/dev/sda1` to `/dev/sdd1` for four devices with a single partition each.
I know that my enclosure lists the hard drives from top to bottom, i.e. with the topmost hard drive as `/dev/sda1` and the bottommost as `/dev/sdd1`.
If you are also so lucky as to have standardized scheme, it may help you to physically group your hard drives so that you know which data are held in a specific area (e.g. with the top two hard drives holding only family photos so that you know that all photos are in `/dev/sda1` and `/dev/sdb1`).
You should now note the UUID of each hard drive, as these will be used below.

Before beginning, you must install mergerfs and fuse via `apt`.

`pi@repi:~$ sudo apt install --yes mergerfs fuse`

Now it is time to create mount points for all the hard drives.
To do this, we use the `mkdir` (make directory) command with elevated privileges in the `/mnt` (mount) directory to create directories for each individual hard drive and a combined directory for all hard drives together.
Using a feature of the [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) shell (the command line interpreter into which you have been typing) called [brace expansion](https://en.wikipedia.org/wiki/Bash_(Unix_shell)#Brace_expansion), you can create multiple directories at once so long as the unique parts are entered within curly brackets and separated with commas.
For this example, I am creating 16 directories with 10 representing the content stored on each of the hard drives, 5 representing parity drives for use with SnapRAID and 1 representing the combined contents of all others via mergerfs: `/mnt/diskMovies1` to `/mnt/diskMovies5`, `/mnt/diskSeries1` to `/mnt/diskSeries5`, `/mnt/parity1` to `/mnt/parity5` and `/mnt/storage`.
Note that as my hard drives only hold media (backups of my extensive DVD and Blu-Ray collection, which is [legal in my jurisdiction](https://en.wikipedia.org/wiki/Ripping#Legality)), I am labeling the hard drive locations according to the type of media they are holding (i.e. movies and series).
(Note also that my files are structured according to Jellyfin standards for [movies](https://jellyfin.org/docs/general/server/media/movies.html) and [shows](https://jellyfin.org/docs/general/server/media/shows.html).)
You should of course modify this to meet your individual needs (e.g. if you are hosting family photos and documents, `/mnt/diskPhoto` and `/mnt/diskDoc`).
(Labeling the directories with trailing numbers and creating more directories than I have hard drives give me flexibility were I to combine two hard drives into a single one with a larger capacity or add more drives in a larger enclosure.)

`pi@repi:~$ sudo mkdir /mnt/{disk{Movies,Series}{1,2,3,4,5},parity{1,2,3,4,5},storage}`

From here, it is time to automatically mount the hard drives when the Raspberry Pi boots up and merge the contents into one directory, namely `/mnt/storage`.
To do this, we have to add entries to [`fstab`](https://en.wikipedia.org/wiki/Fstab) (file systems table; at `/etc/fstab`).
The entries include the UUID noted earlier, mount location (e.g. `/mnt/diskMovies1`), file system type (which is assumed to be ext4), options, dump and pass.
We add the following information (of course with your individual UUIDs replacing longstring) to `/etc/fstab` with the `tee` command as in [Static IP](https://github.com/bcmryan/repi#static-ip).
The final line is the actual mergerfs setup.
The first column groups all directories that begin with `/mnt/disk` (using the [globbing wildcard asterisk](https://en.wikipedia.org/wiki/Glob_(programming)) to signify anything or nothing following) and mounts them together in `/mnt/storage` (second column) as the file system fuse.mergerfs (third column).
The fourth column lists options following the default setup in the [mergerfs documentation](https://github.com/trapexit/mergerfs) with certain changes: first, option cache.files=off is removed, as it is not recognized via the standard upstream installation with `apt`; second mergerfs is now path perserving (i.e. if a directory already exists, a newly created file will go into the directory on the same hard drive instead of making a new directory on a different hard drive); and third it has been given the name mergerfs when listed using `df -h` (disk free space in human-readable terms).
Finally, dump and pass are both left as 0.

```
pi@repi:~$ sudo tee -a /etc/fstab << END

# Mount points for individual hard drives for use with mergerfs at /mnt/disk{Movies,Series}{1,2,3,4,5}
UUID=longstring /mnt/diskMovies1 ext4 defaults 0 0
UUID=longstring /mnt/diskMovies2 ext4 defaults 0 0
UUID=longstring /mnt/diskMovies3 ext4 defaults 0 0
UUID=longstring /mnt/diskSeries1 ext4 defaults 0 0

# mergerfs mount point at /mnt/storage
/mnt/disk* /mnt/storage fuse.mergerfs allow_other,use_ino,dropcacheonclose=true,category.create=epmfs,fsname=mergerfs 0 0
END
```

You can confirm that `/etc/fstab` has been correctly edited using the `cat` command.

`pi@repi:~$ cat /etc/fstab`

Now to mount all devices according to the entries in `/etc/fstab` (i.e. mount the drives we just defined into their new mount points), use the `mount` command with the `--all` option with elevated privileges.

`pi@repi:~$ sudo mount --all`

To verify that all directories have been properly mounted, list all directories in `/mnt` and you will see that the mounted directories are no longer owned by user root and group root.

`pi@repi:~$ ls -lah /mnt`

To view the new combined directory of `/mnt/storage`, list all directories in `/mnt/storage`.

`pi@repi:~$ ls -lah /mnt/storage`

Now that we can access all directories and files in the merged directory of `/mnt/storage`, it is time to use the `chown` (change owner) command with the `--recursive` option and elevated privileges to define user storagerw and group storage as the directory's owner.
Keeping the default permissions (`drwxr-xr-x`, shown when using `ls -lah /mnt` annd looking at `/mnt/storage`), this means that user storagerw will have read-write (and execute) privileges, while anyone belonging to group storage (such as user storagero) will only be able to read (and execute) from the directory but neither modify, delete nor create files or directories within it.
For this command, you can either specify the UID and GID (i.e. 200:200) or the user's and group's name (storagerw:storage).
Note that depending on how many directories and files are stored within `/mnt/storage`, this may take a few minutes (as it needs to change the permissions on every individual directory and file).

`pi@repi:~$ sudo chown --recursive storagerw:storage /mnt/storage`

Once you are returned to a blank command line, you can now see the changes to the ownership of `/mnt/storage` by listing all directories in `/mnt`.
Additionally, you will notice that all directories that merge into `/mnt/storage` also are now owned by user storagerw and group storage.

`pi@repi:~$ ls -lah /mnt`

Due to the change in permissions, user pi (the one which we are using) can read but not modify, delete or create files within `/mnt/storage`.
This is not a problem, as one can always elevate their privileges with `sudo` to be able to have read-write permissions or grant themselves permission by specifying which user to use for a certain application (this functionality is called [force user](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html#FORCEUSER) and [force group](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html#FORCEGROUP) for Samba and [PUID and PGID](https://docs.linuxserver.io/general/understanding-puid-and-pgid) for Docker containers from [LinuxServer.io](https://www.linuxserver.io/)). 

## SnapRAID
This section is currently empty, as I have not yet had the opportunity to configure SnapRAID.
For now, you may want to view the [official SnapRAID website](http://www.snapraid.it/) and [Github page](https://github.com/amadvance/snapraid) and the tutorial for [The Perfect Media Server 2017](https://blog.linuxserver.io/2017/06/24/the-perfect-media-server-2017/) build guide (along with its [accompanying YouTube video](https://www.youtube.com/watch?v=Ir5ZsUIbHXA)). 


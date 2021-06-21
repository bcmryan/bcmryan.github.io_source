+++
title = "Maintenance"
weight = "80"
date = 2021-06-15
+++

Once you have successfully set up and started using your Raspberry Pi as a full-fledged file and media server, it is crucial that you keep your system up to date and keep your application data backed up.
Luckily, with the skills you have acquired or reinforced throughout this tutorial, you should be absolutely fine doing so.
As you have seen throughout this tutorial, Linux is an incredibly powerful base to the Raspberry Pi OS, and its standard tools work very well for the maintenance of your system.

## Backups
In general, backing up your data saved on your hard drives (i.e. everything in `/mnt/storage`) is of course important, and I subscribe to the [3-2-1 backup strategy](https://www.backblaze.com/blog/the-3-2-1-backup-strategy/) of having the original file plus one copy locally and another copy in another location with at least one of the backups on a different type of physical media (e.g. a local copy on a hard drive, a second copy on a hard drive somewhere else in your house and a third copy burned to Blu-Ray and stored at a friend's house) to restore any lost data if something catastrophic happens (e.g. if your house burns down, a local backup will also be lost, but it is less likely that your backup at a friend's house will also be lost).

This section is concerned with backing up application data from Docker itself.
As everything unique to your setup is saved in `/docker`, this folder should be the main focus of your attention regarding the backing up of application data.
(As there are other scripts, configuration files and helpful explanations throughout this tutorial, it might also not be a bad idea to create a local backup of this document as well for future reference.)

Before beginning with creating a backup, you need to create a folder with `mkdir` that is separate from your microSD card (if your microSD card were to have a problem, restoring a backup from it would not be possible).
For this folder location, use for instance `/mnt/diskMovies1/backup`.
This way you know exactly on which physical hard drive the backup is stored (creating `/mnt/storage/backup` might put it on any hard drive in that array).
Note that you need to use elevated privileges, as `/mnt/diskMovies1` is owned by user storagerw and group storage.
Note also that as this directory will be created using elevated privileges, it will be owned by user root and group root and thus will require elevated privileges to access.
This seems sensible, as it will require you to also use elevated privileges before you are able to delete or modify backups.

`pi@repi:~$ sudo mkdir /mnt/diskMovies1/backup`

For creating the backup, use the [`tar`](https://en.wikipedia.org/wiki/Tar_(computing)) (tape archive) command with elevated privileges (as it will be saving a directory not owned by user pi to a folder not owned by user pi).
The following options will be used: `--create` to create a new compressed file, `--gzip` to specify the compression format, `--file` to specify the save location of the new file (with the date appended to the file name using the [`date`](http://manpages.ubuntu.com/manpages/focal/man1/date.1.html) command following the [ISO 8601 standard](https://en.wikipedia.org/wiki/ISO_8601) of year month day) and `--verbose` to list the files being processed.
Finally specify the directory which is being compressed, i.e. `/docker`.

`pi@repi:~$ sudo tar --create --gzip --file /mnt/diskMovies1/backup/docker_$(date +%Y-%m-%d).tar.gz --verbose /docker`

If you were to need to restore from the backup, you would again use the `tar` command with elevated privileges but this time with options `--extract`, `--gzip`, `--file` (with the location of backup file), `--verbose` and `--directory` to specify the extraction location of `/` (as the directory `docker` is at the base of the compressed file).

`pi@repi:~$ sudo tar --extract --gzip --file /mnt/diskMovies1/backup/docker_2021-06-06.tar.gz --verbose --directory /`

Note that following this restoration of your backup, you will need to adjust permissions of `/docker`.

## Updates
Updating your system -- along with keeping best practices in mind regarding the use of passwords and keys and the exposure of your system to the open internet -- is a fundamental part of maintaining a secure system.
As any vulnerability in your system can be a gateway into your local network -- and thus all other computers and smart devices in your home -- it is absolutely imperative to run updates often in order to patch any security holes that may exist.

### apt
Using `apt`, downloading updates and installing upgrades is as simple as `sudo apt update --yes && sudo apt full-upgrade --yes`.
This simple command will make sure that your computing experience remains safe and secure and almost never introduces instability in your system (although this is something that you will always need to keep an eye on).

### Custom scripts
As both Docker and Docker Compose were installed by executing a script downloaded from the internet and as Docker pulls new images from repositories defined in Docker Compose, updates to these will need to be run through a system separate from `apt`.
To keep these systems up to date, I have written a simple script that you should periodically run.
To save this script, again use `tee`.
You will notice that the file `dockerupdate` is being saved with elevated privileges to the directory `/usr/local/bin`.
As this directory is in your [PATH](https://en.wikipedia.org/wiki/PATH_(variable)) (i.e. a list of directories with programs that can be executed without further defining where they are), you will only need to enter `dockerupate` into your terminal emulator to run it.
The script functions as follows.
The script begins with `#!` ([shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))) and `/bin/bash` to define it as a Bash script.
You are first prompted that you will be updating Docker, Docker Compose and Docker images.
As this will close these programs, you should not use this script while they are in use.
The script gives you 30 seconds to end the script (by entering Ctrl+C) via the [`sleep`](http://manpages.ubuntu.com/manpages/focal/man3/sleep.3.html) command.
The `sleep` command is repeated throughout the script to make sure that each process has fully finished before beginning the next one.
The `tar` command creates a simple backup in a compressed file for restoring everything in `/docker` if needed later (as documented above in this section).
As Docker Compose can only be run via relative path, the script changes the current directory to the parent directory of the Docker Compose config file, i.e. `/docker`.
Docker then stops all containers with [`docker container stop`](https://docs.docker.com/engine/reference/commandline/container_stop/) by listing them with `$(docker container ls --all --quiet)` and executing the `stop` command to all listed (i.e. all containers; `--quiet` lists only the container ID so that every stays understandable in the output).
The newest [Docker Compose image from LinuxServer.io](https://docs.linuxserver.io/general/docker-compose) is again pulled, and all dangling images are pruned.
Docker Compose then pulls all images defined in the Docker Compose config file and starts them in the background.
Finally, you are returned to your former working directory (since this was changed to `/docker` at the beginning of the script).
You will notice that before everything dollar sign (`$`) there is a backslash (`\`); this is Bash's [escape character](https://www.gnu.org/software/bash/manual/html_node/Escape-Character.html) and is used so that what literally follows is displayed (and thus while saving the script, the variables themselves are not used; for instance, we would not want `$(date +%Y-%m-%d)` to return the current date, or every backup you would have would be listed as if it were from the date you created this script, i.e. today).

```
pi@repi:~$ sudo tee /usr/local/bin/dockerupdate << END
#! /bin/bash

echo "This script will stop all Docker containers and update Docker, Docker Compose
and all Docker images. Cease the use of any Docker containers immediately. If you
would like to stop this script, you have 30 seconds to do so by entering Ctrl+C."

# sleep for 30 seconds
sleep 30s

# update all programs installed with apt
sudo apt update --yes
sudo apt full-upgrade --yes

# sleep for 5 seconds
sleep 5s

# back up /docker
sudo tar --create --gzip --file /mnt/diskMovies1/backup/docker_\$(date +%Y-%m-%d).tar.gz --verbose /docker

# sleep for 5 seconds
sleep 5s

# stop all Docker containers
docker container stop \$(docker container ls --all --quiet)

# sleep for 5 seconds
sleep 5s

# update Docker
curl https://get.docker.com | sh

# sleep for 5 seconds
sleep 5s

# update Docker Compose
docker pull linuxserver/docker-compose:"\${DOCKER_COMPOSE_IMAGE_TAG:-latest}"
docker image prune --force

# sleep for 5 seconds
sleep 5s

# change directory because the Docker Compose config file can only be run via a relative path (https://github.com/docker/compose/issues/3875#issuecomment-502899871)
cd /docker

# sleep for 5 seconds
sleep 5s

# update all images with Docker Compose
docker-compose --file ./docker-compose/docker-compose.yml pull

# sleep for 5 seconds
sleep 5s

# start all containers in background
docker-compose --file ./docker-compose/docker-compose.yml up --detach

# return to former working directory
cd "\$OLDPWD"
END
```

So that the script can be executed, change the mode to executable using `chmod +x`.

`pi@repi:~$ sudo chmod +x /usr/local/bin/dockerupdate`

Test this script with the newly created `dockerupdate` command.

`pi@repi:~$ dockerupdate`

<!--To schedule automatic updates, you can employ [cron](https://en.wikipedia.org/wiki/Cron), a time-based scheduler for running commands.
Using the correct formatting for cron, the job `* 4 * * 1 /usr/local/bin/dockerupdate >/dev/null 2>&1` is defined as the 04:00 (i.e. 4:00 AM) Monday execution of command `/usr/local/bin/dockerupdate` without logs (by sending all output to `/dev/null`).
While this is possible to input using `crontab -e`, I perfer to directly [create a cron job with a single command](https://stackoverflow.com/questions/4880290/how-do-i-create-a-crontab-through-a-script), thus making it easier for scripting.

`pi@repi:~$ (crontab -l 2>/dev/null; echo "* 4 * * 1 /usr/local/bin/dockerupdate >/dev/null 2>&1") | crontab -`-->

## Nuking and paving
After fully configuring your system, I highly creating a backup image of your microSD card.
As described in [Operating system](./operating-system), first find the mount locations of the partitions on your microSD card and the location of the device in `/dev` with `lsblk`.

`bcmryan@desktop:~$ lsblk`

Now unmount the mount locations.

`bcmryan@desktop:~$ sudo umount /media/bcmryan/boot && sudo umount /media/bcmryan/rootfs`

Following the [official documentation](https://www.raspberrypi.org/documentation/linux/filesystem/backup.md), use `dd` with a block size of 4 MB (`bs=4M`) and create an image of your device (`if=/dev/sdd`; this was found with `lsblk` above and does not include the partition number) to a backup location (`of=/path/to/backups/PiOS_$(date +%Y-%m-%d).img`; in this case appended with the current date).

`bcmryan@desktop:~$ sudo dd bs=4M if=/dev/sdd of=/path/to/backups/PiOS_$(date +%Y-%m-%d).img`

Running this command in reverse flashes your microSD card.
(Again paying attention to the drive letter.)

`bcmryan@desktop:~$ sudo dd bs=4M if=/path/to/backups/PiOS_2021-06-06.img of=/dev/sdd`

That said, given the flexibility and relative ease of setting up a Raspberry Pi server, I genuinely think that the path of least resistance following a catastrophic failure in many cases is to simply rebuild your server and populate your Docker application data from the last local backup.
This rather drastic approach is called [nuking and paving](https://en.wiktionary.org/wiki/nuke_and_pave).
Nuking and paving allows you to reevaluate exactly what you are installing and why and may avoid problems if there were to have been bugs in earlier versions of software that was actually saved in the backup of the microSD card.
Since the applications themselves are not being backed up using the above method for Docker (i.e. creating a backup of `/docker`), this is generally not an issue with the application data of containers.
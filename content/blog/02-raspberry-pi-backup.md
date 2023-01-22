+++
date = "2022-12-08T20:00:00+01:00"
draft = false
title = "Raspberry Pi Backup"
slug = "raspberry-pi-backup"
author = "Christophe Knage"
+++

**This post covers implementing an effective backup strategy for your Raspberry Pi**

Having a plan for how to recover when things break down is essential when you depend on your server.

> This is part 2 of 5 in a mini series where we will configure a headless Raspberry Pi 4 B as an efficient home server, with an effective backup strategy, capable of hosting <a href="https://en.wikipedia.org/wiki/Network-attached_storage" target="_blank">Network Attached Storage (NAS)</a>, <a href="https://support.apple.com/en-gb/HT201250" target="_blank">TimeMachine</a>, <a href="https://www.plex.tv" target="_blank">Plex Media Server</a> and <a href="https://homebridge.io" target="_blank">Homebridge</a>.

**Prerequisites**

Hardware Requirements
- <a href="https://www.raspberrypi.com/products/raspberry-pi-4-model-b/" target="_blank">Raspberry Pi 4 B</a>
- <a href="https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards" target="_blank">microSD</a>
- External Storage (e.g. SSD in a USB 3.0 enclosure)

Software Requirements
- <a href="https://github.com/Drewsif/PiShrink" target="_blank">PiShrink</a>

<br/>

{{<toc>}}

<br/>

In this post I assume you have read [part 1]({{<relref"/blog/01-raspberry-pi-headless-setup">}} "Headless Raspberry Pi Server") in this series and thus have a Raspberry Pi with Desktop you can <a href="https://manpages.debian.org/bullseye/openssh-client/ssh.1.en.html" target="_blank" class="code-doc">`SSH`</a> in to @ `raspberrypi.local`.

## Backing up the Raspberry Pi

The easiest way to get back up and running after it's gone all wrong, is having a full bootable system image ready to flash on to your microSD card with Raspberry Pi Imager.

On your Raspberry Pi, you can create such an image using the <a href="https://manpages.debian.org/bullseye/coreutils/dd.1.en.html" target="_blank" class="code-doc">`dd`</a> command to copy the current running installation to an .img file on some external storage plugged in to the Raspberry Pi's USB port.

<br/>

### Setting up the external storage

First you need to plug your external storage in to the Raspberry Pi's USB port.  

You can use the <a href="https://manpages.debian.org/bullseye/util-linux/lsblk.8.en.html" target="_blank" class="code-doc">`lsblk`</a> tool to get at list of all the block devices currently attached to your Raspberry Pi, and their mount points using this command:
```console
lsblk -f
```
[output]
```
pi@raspberrypi:~ $ lsblk -f
NAME        FSTYPE  FSVER LABEL       UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda                                                                                       
├─sda1      vfat    FAT32 EFI         D2D7-4855                                           
├─sda2      hfsplus       Monterey    99b629a1-2994-45cf-b8c1-cff6d7450694                
├─sda3      hfsplus       Ventura     3516e24d-84c2-4f9e-9559-71de730a7277                
├─sda4      ext4    1.0   TimeMachine fc924351-26c8-4c01-a25f-e44a6869c5f1  614.6G    25% /media/pi/TimeMachine
└─sda5      ext4    1.0   NAS         665507c3-aee6-41bc-8be8-3594a65468d9    1.3T    45% /media/pi/NAS
mmcblk0                                                                                   
├─mmcblk0p1 vfat    FAT32 boot        444F-BE04                             200.3M    21% /boot
└─mmcblk0p2 ext4    1.0   rootfs      665507c3-aee6-41bc-8be8-3594a65468d9   19.4G    28% /
pi@raspberrypi:~ $ 
```

`mmcblk0` is the microSD card with your full installation on it that you wish to backup.  
`sda` is my SSD with 5 partitions on it. In this post I will use partition `sda5` labeled NAS mounted at `/media/pi/NAS` as the destination for the backup.

Because you installed Raspberry Pi with Desktop (in [part 1]({{<relref"/blog/01-raspberry-pi-headless-setup">}} "Headless Raspberry Pi Server")), removable media will be auto mounted to `/media/pi` by the the <a href="https://manpages.debian.org/bullseye/pcmanfm/pcmanfm.1.en.html" target="_blank" class="code-doc">`pcmanfm`</a> desktop process.

<a href="https://manpages.debian.org/bullseye/pcmanfm/pcmanfm.1.en.html" target="_blank" class="code-doc">`pcmanfm`</a> uses <a href="https://manpages.debian.org/bullseye/udisks2/udisksctl.1.en.html" target="_blank" class="code-doc">`udisksctl`</a> on the backend to mount your drive, so you can also mount the `sda5` partition manually like this:
```console
udisksctl mount -b /dev/sda5
```

<br/>

### Backing up the microSD card

With your drive properly mounted, you can use <a href="https://manpages.debian.org/bullseye/coreutils/mkdir.1.en.html" target="_blank" class="code-doc">`mkdir`</a> to create a directory to store the backup: 
```console
sudo mkdir /media/pi/NAS/Raspberry/BACKUPS
```

Then make your first backup of your mocroSD card with <a href="https://manpages.debian.org/bullseye/coreutils/dd.1.en.html" target="_blank" class="code-doc">`dd`</a>. This will take some time:
```console
sudo dd if=/dev/mmcblk0 of=/media/pi/NAS/Raspberry/BACKUPS/'RaspberryPi-(year-month-date).img' bs=1M
```
<!--
[output]
```
pi@raspberrypi:~ $ sudo dd if=/dev/mmcblk0 of=/media/pi/NAS/Raspberry/BACKUPS/'RaspberryPi-(year-month-date).img' bs=1M
29608+0 records in
29608+0 records out
31046238208 bytes (31 GB, 29 GiB) copied, 754.574 s, 41.1 MB/s
pi@raspberrypi:~ $
```
-->

After some time, you should have the file `RaspberryPi-(year-month-date).img` in your backup directory. the .img file will be the full size of your microSD card, in my case ca. 31 GB.

<br/>

### Compressing the backup with PiShrink

You can then use a script called <a href="https://github.com/Drewsif/PiShrink" target="_blank">PiShrink</a> to shrink the .img files size and compresses it with <a href="https://manpages.debian.org/bullseye/gzip/gzip.1.en.html" target="_blank" class="code-doc">`gzip`</a>. <a href="https://github.com/Drewsif/PiShrink" target="_blank">PiShrink</a> also modifies your image so that it will expand itself to take all the available space on your microSD on first boot.

This is really good because if your current microSD card gives up, your new card might not be the same size as the old one. 

<br/>

Install <a href="https://github.com/Drewsif/PiShrink" target="_blank">PiShrink</a> with <a href="https://manpages.debian.org/bullseye/wget/wget.1.en.html" target="_blank" class="code-doc">`wget`</a>, <a href="https://manpages.debian.org/bullseye/coreutils/chmod.1.en.html" target="_blank" class="code-doc">`chmod`</a> and <a href="https://manpages.debian.org/bullseye/coreutils/mv.1.en.html" target="_blank" class="code-doc">`mv`</a>:
```console
wget https://raw.githubusercontent.com/Drewsif/PiShrink/master/pishrink.sh
chmod +x pishrink.sh
sudo mv pishrink.sh /usr/local/bin
```

Then shrink your image like this. Again, this will take some time:
```console
sudo pishrink.sh -z "/media/pi/NAS/Raspberry/BACKUPS/RaspberryPi-(year-month-date).img"
```
<!--
[output]
```
pi@raspberrypi:~ $ sudo pishrink.sh -z "/media/pi/NAS/Raspberry/BACKUPS/RaspberryPi-(year-month-date).img"
pishrink.sh v0.1.2
pishrink.sh: Gathering data ...
Creating new /etc/rc.local
pishrink.sh: Checking filesystem ...
rootfs: Inode 7516 extent tree (at level 1) could be narrower.  IGNORED.
rootfs: Inode 7589 extent tree (at level 1) could be narrower.  IGNORED.
rootfs: Inode 11747 extent tree (at level 1) could be narrower.  IGNORED.
rootfs: Inode 46049 extent tree (at level 2) could be narrower.  IGNORED.
rootfs: Inode 62207 extent tree (at level 1) could be narrower.  IGNORED.
rootfs: Inode 85311 extent tree (at level 2) could be narrower.  IGNORED.
rootfs: Inode 260670 extent tree (at level 1) could be narrower.  IGNORED.
rootfs: Inode 260673 extent tree (at level 2) could be narrower.  IGNORED.
rootfs: Inode 263112 extent tree (at level 2) could be narrower.  IGNORED.
rootfs: 241629/1854720 files (0.6% non-contiguous), 2220285/7513088 blocks
resize2fs 1.46.2 (28-Feb-2021)
pishrink.sh: Shrinking filesystem ...
resize2fs 1.46.2 (28-Feb-2021)
Resizing the filesystem on /dev/loop0 to 2224074 (4k) blocks.
Begin pass 2 (max = 295292)
Relocating blocks             XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Begin pass 3 (max = 230)
Scanning inode table          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Begin pass 4 (max = 62870)
Updating inode references     XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
The filesystem on /dev/loop0 is now 2224074 (4k) blocks long.

pishrink.sh: Shrinking image ...
pishrink.sh: Using gzip on the shrunk image ...
pishrink.sh: Shrunk /media/pi/NAS/Raspberry/BACKUPS/RaspberryPi-(year-month-date).img.gz from 29G to 4.6G ...
pi@raspberrypi:~ $ 
```
-->

This will leave you with a significantly smaller file `RaspberryPi-(year-month-date).img.gz`, in my case ca. 4.6 GB.

## Automating the process

The backup process can easily be scripted, so it happens periodically on a cadence to your liking.

First, create a script file and make it executable with <a href="https://manpages.debian.org/bullseye/coreutils/touch.1.en.html" target="_blank" class="code-doc">`touch`</a> and <a href="https://manpages.debian.org/bullseye/coreutils/chmod.1.en.html" target="_blank" class="code-doc">`chmod`</a>:
```console
touch ./Documents/sd-card-backup.sh
chmod +x ./Documents/sd-card-backup.sh
```

Then open the file `sd-card-backup.sh` in <a href="https://manpages.debian.org/bullseye/nano/nano.1.en.html" target="_blank" class="code-doc">`nano`</a>:
```console
nano ./Documents/sd-card-backup.sh
```

Add this script to the file `sd-card-backup.sh` (make sure the `SD_CARD` and `BACKUP_DIRECTORY` variables are correct for you)
```bash
#!/bin/bash

CURRENT_DATETIME=`date +"%Y-%m-%dT%H-%M-%S"`
SD_CARD="/dev/mmcblk0"
BACKUP_DIRECTORY="/media/pi/NAS/Raspberry/BACKUPS"
BACKUP_FILENAME="${BACKUP_DIRECTORY}/RaspberryPi-(${CURRENT_DATETIME}).img"

# Exit script if Backup Directory does not exist
if [ ! -d "${BACKUP_DIRECTORY}" ]; then
  echo "${BACKUP_DIRECTORY} does not exist"
  exit 1
fi

# Create dump of SD Card in Backup Directory
sudo dd if=${SD_CARD} of=${BACKUP_FILENAME} bs=1M status=progress

# Shrink backup image with PiShrink
sudo pishrink.sh -z ${BACKUP_FILENAME}

# Delete backups older than 365 days
find ${BACKUP_DIRECTORY} -maxdepth 1 -name "*.img.gz"  -type f -mtime +365  -delete
```

Open <a href="https://manpages.debian.org/bullseye/systemd-cron/crontab.1.en.html" target="_blank" class="code-doc">`crontab`</a> with the commend:
```console
crontab -e
```

At the end of the document, type the frequency of script execution in the following manner
```bash
0 0 1 */1 * /home/pi/Documents/sd-card-backup.sh
```

The above line means that the script with run **At 00:00 on day-of-month 1 in every month**. You can configure this according to your needs. I recommend using <a href="https://crontab.guru/#0_0_1_*/1_*" target="_blank">crontab.guru</a> to get the right settings for you.

<h1 style="font-size: 100%">Sources</h1>

- <a href="https://robotzero.one/headless-pi-zero-backup-clone/" target="_blank">Back Up Headless Raspberry Pi Zero with RPI-Clone or PiShrink</a>
- <a href="https://forums.raspberrypi.com/viewtopic.php?t=276494#p1675675" target="_blank">USB storage auto-mount in /media/pi</a>

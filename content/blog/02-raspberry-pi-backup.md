+++
date = "2022-12-08T20:00:00+01:00"
draft = false
title = "Raspberry Pi Backup"
slug = "raspberry-pi-backup"
author = "Christophe Knage"
+++

**This post covers implementing an effective backup strategy for your Raspberry Pi**

Having a plan for how to recover when things break down is essential when you depend on your server.

> This is part 2 of 5 in a mini series where we will configure a headless Raspberry Pi 4 B as an efficient home server, with an effective backup strategy, capable of hosting [Network Attached Storage (NAS)](https://en.wikipedia.org/wiki/Network-attached_storage), [TimeMachine](https://support.apple.com/en-gb/HT201250), [Plex Media Server](https://www.plex.tv) and [Homebridge](https://homebridge.io).

### Prerequisites

Hardware Requirements
- [Raspberry Pi 4 B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [microSD](https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards)
- External Storage (e.g. SSD in a USB 3.0 enclosure)

Software Requirements
- [PiShrink](https://github.com/Drewsif/PiShrink)

<br/>

In this post I assume you have read [part 1](/2022/12/raspberry-pi-headless-setup/) in this series and thus have a Raspberry Pi with Desktop you can `SSH` in to @ `raspberrypi.local`.

## Backing up the Raspberry Pi

The easiest way to get back up and running after it's gone all wrong, is having a full bootable system image ready to flash on to your microSD card with Raspberry Pi Imager.

On your Raspberry Pi, you can create such an image using the `dd` command to copy the current running installation to an .img file on some external storage plugged in to the Raspberry Pi's USB port.

<br/>

### Setting up the external storage

First you need to plug in your external storage to the Raspberry Pi's USB port.  
Because you installed Raspberry Pi with Desktop, removable media will be auto mounted to `/media/pi` by the the [`pcmanfm`](https://manpages.debian.org/bullseye/pcmanfm/pcmanfm.1.en.html) desktop process.

you can use the `lsblk` tool to get at list of all the block devices currently attached to your Raspberry Pi, and their mount points using this command:
```bash
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

<br/>

### Basking up the microSD card

<br/>

### Compressing the backup with PiShrink

## Automating the process


```bash
touch ./Documents/sd-card-backup.sh
chmod +x ./Documents/sd-card-backup.sh
```

Open the file `sd-card-backup.sh` in `nano`

```bash
nano ./Documents/sd-card-backup.sh
```

Add below content to the file `sd-card-backup.sh` (make sure `BACKUPFILENAME` is correct)
```bash
#!/bin/bash

CURRENT_DATETIME=`date +"%Y-%m-%dT%H-%M-%S"`
BACKUP_DIRECTORY="/media/pi/NAS/Raspberry/BACKUPS"
BACKUP_FILENAME="${BACKUP_DIRECTORY}/RaspberryPi-(${CURRENT_DATETIME}).img"

# Exit script if Backup Directory does not exist
if [ ! -d "${BACKUP_DIRECTORY}" ]; then
  echo "${BACKUP_DIRECTORY} does not exist"
  exit 1
fi

# Create dump of SD Card in Backup Directory
sudo dd if="/dev/mmcblk0" of=${BACKUP_FILENAME} bs=1M status=progress

# Shrink backup image with PiShrink
sudo pishrink.sh -z ${BACKUP_FILENAME}

# Delete backups older than specified number of days
find ${BACKUP_DIRECTORY} -maxdepth 1 -name "*.img.gz"  -type f -mtime +365  -delete
```

Open crontab by typing 

```bash
crontab -e
```

and navigate to the end of the document
Type the frequency of script execution in the following manner

```bash
0 0 1 */1 * /home/pi/Documents/sd-card-backup.sh
```

The above line means that the script with run **"At 00:00 on day-of-month 1 in every month."** You can configure this according to your needs. I recommend using [crontab.guru](https://crontab.guru/#0_4_*/21_*_*) to get the right settings for the cron job.

#
### Sources

- [Back Up Headless Raspberry Pi Zero with RPI-Clone or PiShrink](https://robotzero.one/headless-pi-zero-backup-clone/)
- [USB storage auto-mount in /media/pi](https://forums.raspberrypi.com/viewtopic.php?t=276494#p1675675)

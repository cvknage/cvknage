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

## Backing up the Raspberry Pi

The easiest way to get back up and running after it's gone all wrong, is having a full system image at the ready.

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
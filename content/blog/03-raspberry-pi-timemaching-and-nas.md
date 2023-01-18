+++
date = "2023-01-17T22:00:00+01:00"
draft = false
title = "Raspberry Pi as TimeMachine and NAS"
slug = "raspberry-pi-timemachine-and-nas"
author = "Christophe Knage"
+++

**This post covers configuring a Raspberry Pi 4 B as a TimeMachine and NAS Server.**

Raspberry Pi offers a great <a href="https://www.raspberrypi.com/tutorials/nas-box-raspberry-pi-tutorial/" target="_blank">guide</a> on how to setup a NAS server (that also supports TimeMachine) using <a href="https://www.openmediavault.org" target="_blank">openmediavault</a>.

This post however takes a different approach, where you will configure <a href="https://www.samba.org/" target="_blank">Samba</a> manually to make your TimeMachine and NAS available on your network.

> This is part 3 of 5 in a mini series where we will configure a headless Raspberry Pi 4 B as an efficient home server, with an effective backup strategy, capable of hosting <a href="https://en.wikipedia.org/wiki/Network-attached_storage" target="_blank">Network Attached Storage (NAS)</a>, <a href="https://support.apple.com/en-gb/HT201250" target="_blank">TimeMachine</a>, <a href="https://www.plex.tv" target="_blank">Plex Media Server</a> and <a href="https://homebridge.io" target="_blank">Homebridge</a>.

### Prerequisites

Hardware Requirements
- <a href="https://www.raspberrypi.com/products/raspberry-pi-4-model-b/" target="_blank">Raspberry Pi 4 B</a>
- <a href="https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards" target="_blank">microSD</a>
- External Storage (e.g. SSD in a USB 3.0 enclosure)
- Second External Storage (e.g. HDD in a USB enclosure) (Optional)

Software Requirements
- <a href="https://www.samba.org/" target="_blank">Samba</a>
- <a href="https://www.avahi.org/" target="_blank">Avahi</a>

## Setting up the external storage

### Prevent macOS recovery partition auto mount

edit <a href="https://manpages.debian.org/bullseye/mount/fstab.5.en.html" target="_blank" class="code-doc">`fstab`</a>

```bash
sudo nano /etc/fstab
```

Add partitions to <a href="https://manpages.debian.org/bullseye/mount/fstab.5.en.html" target="_blank" class="code-doc">`fstab`</a> with **noauto**

```conf
####################### fstab ########################
proc                    /proc           proc    defaults            0   0
PARTUUID=15ea4d29-01    /boot           vfat    defaults            0   2
PARTUUID=15ea4d29-02    /               ext4    defaults,noatime    0   1

UUID=99b629a1-2994-45cf-b8c1-cff6d7450694   /media/pi/Monterey  hfsplus  noauto  0  0
UUID=3516e24d-84c2-4f9e-9559-71de730a7277   /media/pi/Ventura   hfsplus  noauto  0  0

# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that
```

## Setup TimeMachine and NAS

### Install dependencies

<br/>

### Configuring Samba

<br/>

### Configuring Avahi

## NAS Backup (Optional)

Make sure the BACKUP drive is mounted before running the script. 

### <a href="https://manpages.debian.org/bullseye/rsync/rsync.1.en.html" target="_blank" class="code-doc">`rsync`</a> backup script

<a href="https://manpages.debian.org/bullseye/coreutils/touch.1.en.html" target="_blank" class="code-doc">`touch`</a>  
<a href="https://manpages.debian.org/bullseye/coreutils/chmod.1.en.html" target="_blank" class="code-doc">`chmod`</a>
```bash
touch ./Documents/nas-backup.sh
chmod +x ./Documents/nas-backup.sh
```

```bash
BACKUP_DIRECTORY="/media/pi/BACKUP/NAS"

# Exit script if Backup Directory does not exist
if [ ! -d "${BACKUP_DIRECTORY}" ]; then
  echo "${BACKUP_DIRECTORY} does not exist"
  exit 1
fi

START_DATETIME=`date +"%Y-%m-%dT%H-%M-%S"`
echo "rsync start: ${START_DATETIME}" >> ./Documents/nas-backup.log
echo "rsync start: ${START_DATETIME}"

# rsync entire NAS to BACKUP 
/usr/bin/rsync -aP --exclude="lost+found" /media/pi/NAS/ "${BACKUP_DIRECTORY}" >> ./Documents/nas-backup.log

END_DATETIME=`date +"%Y-%m-%dT%H-%M-%S"`
echo "rsync finished: ${END_DATETIME}" >> ./Documents/nas-backup.log
echo "rsync finished: ${END_DATETIME}"
```

Follow `nas-backup.log`

<a href="https://manpages.debian.org/bullseye/coreutils/tail.1.en.html" target="_blank" class="code-doc">`tail`</a>
```bash
tail -F -s20 ./Documents/nas-backup.log
```

To delete files in the destination folder that are not in the source folder (deleted files, temporary rsync files, etc.)

```bash
# dry-run
/usr/bin/rsync -aPn --del --exclude="lost+found" /media/pi/NAS/ /media/pi/BACKUP/NAS

# DELETE FILES - verify with dry-run first
/usr/bin/rsync -aP --del --exclude="lost+found" /media/pi/NAS/ /media/pi/BACKUP/NAS
```

#
### Sources

- <a href="https://mudge.name/2019/11/12/using-a-raspberry-pi-for-time-machine/#configuring-avahi" target="_blank">Using a Raspberry Pi for Time Machine</a>

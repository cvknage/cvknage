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

I use a cheap USB 3.0 enclosure with a 2 TB SSD inside and I want to use one partition for Time Machine and another partition for my NAS.  
(I also have a EFI bootloader partition as well as 2 bootable macOS recovery partitions.)

If you need to format and partition your drive, here is a nice guide: <a href="https://thesecmaster.com/how-to-partition-and-format-the-hard-drives-on-raspberry-pi/" target="_blank">How to Partition and Format the Hard Drives on Raspberry Pi?</a> 

> **Note:**  
> If you are looking for a new external storage disk, it is worth noting that the Raspberry Pi 4 is known to be picky about what adapters will work in the USB 3.0 ports  
A community built list of adapters that are known to work with the Raspberry Pi 4, as well as those known not to work out of the box can be found <a href="https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/" target="_blank">here</a>.
>
> If you already own a disk that is causing problems, a solution may be found <a href="https://www.pragmaticlinux.com/2021/03/fix-for-getting-your-ssd-working-via-usb-3-on-your-raspberry-pi/" target="_blank">here</a>.

<br/>

### Mounting the disk

After connecting your external storage in to the Raspberry Pi's USB port, you can use the <a href="https://manpages.debian.org/bullseye/util-linux/lsblk.8.en.html" target="_blank" class="code-doc">`lsblk`</a> tool to get at list of all the block devices currently attached to your Raspberry Pi, and their mount points using this command:
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

`sda4` is my TimeMachine partition mounted as `/media/pi/TimeMachine`.  
`sda5` is my NAS partition mounted at `/media/pi/NAS`.  

Because you installed Raspberry Pi with Desktop (in [part 1]({{<relref"/blog/01-raspberry-pi-headless-setup">}} "Headless Raspberry Pi Server")), removable media will be auto mounted to `/media/pi` by the the <a href="https://manpages.debian.org/bullseye/pcmanfm/pcmanfm.1.en.html" target="_blank" class="code-doc">`pcmanfm`</a> desktop process.

<a href="https://manpages.debian.org/bullseye/pcmanfm/pcmanfm.1.en.html" target="_blank" class="code-doc">`pcmanfm`</a> uses <a href="https://manpages.debian.org/bullseye/udisks2/udisksctl.1.en.html" target="_blank" class="code-doc">`udisksctl`</a> on the backend to mount your drive, so you can also mount the `sda5` partition manually like this:
```console
udisksctl mount -b /dev/sda5
```

If you installed Raspberry Pi OS Lite, you can [enable the VNC server]({{<relref"/blog/01-raspberry-pi-headless-setup#enable-vnc-server">}} "Enable VNC Server") and [create a Virtual Desktop]({{<relref"/blog/01-raspberry-pi-headless-setup#troubleshooting-creating-a-virtual-desktop">}} "Creating a Virtual Desktop"), which loads the default desktop session with the a <a href="https://manpages.debian.org/bullseye/pcmanfm/pcmanfm.1.en.html" target="_blank" class="code-doc">`pcmanfm`</a> desktop process, which in turn enables auto mounting.  
Or you can use this <a href="https://github.com/bswebdk/scripts/blob/master/uamount.sh" target="_blank">`uamount.sh`</a> script to generate <a href="https://manpages.debian.org/bullseye/udev/udev.7.en.html" target="_blank" class="code-doc">`udev`</a> mounting rules and add their corresponding entries to <a href="https://manpages.debian.org/bullseye/mount/fstab.5.en.html" target="_blank" class="code-doc">`fstab`</a> to make your partitions auto mount at your desired location.

<details>
<summary style="cursor: pointer">Click to expand: <code><a href="https://manpages.debian.org/bullseye/udev/udev.7.en.html" target="_blank" class="code-doc">udev</a></code> mounting rule example</summary>

**Using the `uamount.sh` script**

Install the `uamount.sh` script:
```console
wget https://github.com/bswebdk/scripts/blob/master/uamount.sh
```

To get help run (or just read the top of the script):
```console
./uamount.sh --help
```

To create a <a href="https://manpages.debian.org/bullseye/udev/udev.7.en.html" target="_blank" class="code-doc">`udev`</a> mounting rule for `/dev/sda` on `/media/MYDRIVE` use:
```console
sudo ./uamount.sh /dev/sda MYDRIVE
```

To remove the mounting rule again use:
```console
sudo ./uamount.sh /dev/sda
```

<br/>

**Manually create a <a href="https://manpages.debian.org/bullseye/udev/udev.7.en.html" target="_blank" class="code-doc">`udev`</a> mounting rule**

Edit <a href="https://manpages.debian.org/bullseye/mount/fstab.5.en.html" target="_blank" class="code-doc">`fstab`</a> in <a href="https://manpages.debian.org/bullseye/nano/nano.1.en.html" target="_blank" class="code-doc">`nano`</a>:
```console
sudo nano /etc/fstab
```

And add an entry for the desired partition:
```
UUID=665507c3-aee6-41bc-8be8-3594a65468d9   /media/pi/NAS  ext4    defaults,noauto  0  0
```
The parameter `noauto` is important here as it tels the system not to mount during the init process.

Then create a new <a href="https://manpages.debian.org/bullseye/udev/udev.7.en.html" target="_blank" class="code-doc">`udev`</a> mounting rule:
```console
sudo nano /lib/udev/rules.d/automount-nas.rules
```

And add the actions for **"add"** and **"remove"**
```
ACTION=="add", ENV{ID_FS_UUID_ENC}=="665507c3-aee6-41bc-8be8-3594a65468d9", RUN+="/bin/mount /dev/%k"
ACTION=="remove", ENV{ID_FS_UUID_ENC}=="665507c3-aee6-41bc-8be8-3594a65468d9", RUN+="/bin/umount /dev/%k"
```
</details>

<br/>

> **Note:**  
Auto mounting removable media directly from <a href="https://manpages.debian.org/bullseye/mount/fstab.5.en.html" target="_blank" class="code-doc">`fstab`</a> without using a <a href="https://manpages.debian.org/bullseye/udev/udev.7.en.html" target="_blank" class="code-doc">`udev`</a> mounting rule is not recommended.  
Since <a href="https://manpages.debian.org/bullseye/mount/fstab.5.en.html" target="_blank" class="code-doc">`fstab`</a> is read during the system init process, removing the drive from the Raspberry Pi will cause the system to hang for a long time on boot or even fail to boot completely.  
Using a <a href="https://manpages.debian.org/bullseye/udev/udev.7.en.html" target="_blank" class="code-doc">`udev`</a> mounting rule defers the mounting procedure in to user space which will prevent boot failure.

<br/>

#### Prevent partitions from auto mounting

I don't want the 2 macOS recovery partitions to be mounted on the Raspberry Pi.

To prevent this, edit <a href="https://manpages.debian.org/bullseye/mount/fstab.5.en.html" target="_blank" class="code-doc">`fstab`</a> in <a href="https://manpages.debian.org/bullseye/nano/nano.1.en.html" target="_blank" class="code-doc">`nano`</a>:
```console
sudo nano /etc/fstab
```

And add an entry for each partition:
```
####################### fstab ########################
proc                    /proc           proc    defaults            0   0
PARTUUID=15ea4d29-01    /boot           vfat    defaults            0   2
PARTUUID=15ea4d29-02    /               ext4    defaults,noatime    0   1

UUID=99b629a1-2994-45cf-b8c1-cff6d7450694   /media/pi/Monterey  hfsplus  noauto  0  0
UUID=3516e24d-84c2-4f9e-9559-71de730a7277   /media/pi/Ventura   hfsplus  noauto  0  0

# a swapfile is not a swap partition, no line here
# use  dphys-swapfile swap[on|off]  for that
```

## Setup TimeMachine and NAS

With your disk properly mounted, it's time setup the Raspberry Pi to host your TimeMachine and NAS.

### Install dependencies

First you need to install the two software packages <a href="https://www.samba.org/" target="_blank">Samba</a> and <a href="https://www.avahi.org/" target="_blank">Avahi</a>:
```console
sudo apt update
sudo apt install samba avahi-daemon
```

<a href="https://www.samba.org/" target="_blank">Samba</a> is a free software re-implementation of the SMB networking protocol, which is officially supported by <a href="https://support.apple.com/en-us/HT202784#nas" target="_blank">Apple</a>.  
Since macOS Mavericks Apple has preferred SMB over AFP and AFP seems to have been discontinued entirely as of macOS Big Sur.

<a href="https://www.avahi.org/" target="_blank">Avahi</a> is a system which facilitates service discovery on a local network via the mDNS/DNS-SD protocol suite. In Apple language, this is know as <a href="https://developer.apple.com/bonjour/" target="_blank">Bonjour</a>.

<br/>

### Configuring Samba
 
In order for <a href="https://www.samba.org/" target="_blank">Samba</a> to work well with macOS you need to configure <a href="https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html" target="_blank" class="code-doc">`vfs_fruit`</a> - Enhanced OS X and Netatalk interoperability. 

You do this by editing the <a href="https://www.samba.org/" target="_blank">Samba</a> configuration file<a href="https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html" target="_blank" class="code-doc">`/etc/samba/smb.conf`</a>  in <a href="https://manpages.debian.org/bullseye/nano/nano.1.en.html" target="_blank" class="code-doc">`nano`</a>:
```console
sudo nano /etc/samba/smb.conf
```

Near the top of the file you will find something like this:
```make
#======================= Global Settings =======================
# https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html

[global]
```

The `[global]` tag marks global section of the file.  
Parameters in this section apply to the server as a whole, or are defaults for sections that do not specifically define certain items.

Add the following to the `[global]` section:
```make
### Basic Samba Options ###
# https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html

   min protocol = SMB2
   ea support = yes
   inherit permissions = yes


### Make Samba like Apples by teaching it about fruit ###
# https://www.samba.org/samba/docs/current/man-html/vfs_fruit.8.html

   vfs objects = catia fruit streams_xattr
   fruit:metadata = stream
   fruit:model = MacSamba
   fruit:posix_rename = yes
   fruit:veto_appledouble = no
   fruit:nfs_aces = no
   fruit:wipe_intentionally_left_blank_rfork = yes 
   fruit:delete_empty_adfiles = yes 
   fruit:zero_file_id = yes
```

Further down the file, you will find a heading like this:
```make
#======================= Share Definitions =======================
```

This is where you tell <a href="https://www.samba.org/" target="_blank">Samba</a> what to share on your network.

At the bottom of this section add the following:
```make
### Adding Shares ###
# https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html#idm51

[TimeMachine]
    comment = TimeMachine
    path = /media/pi/TimeMachine
    valid users = pi
    read only = no
    inherit acls = yes
    fruit:time machine = yes

[NAS]
    comment = NAS    
    path = /media/pi/NAS
    valid users = pi
    read only = no
    inherit acls = yes
;    spotlight backend = elasticsearch
```

This creates 2 shares "TimeMachine" and "NAS" which can only be accessed by the `pi` user. 

You can now save and close the configuration file.

Even though the `pi` user already exists, because you created it with raspberry Pi Imager in [part 1]({{<relref"/blog/01-raspberry-pi-headless-setup">}} "Headless Raspberry Pi Server"), you need to explicitly add the user to <a href="https://www.samba.org/" target="_blank">Samba</a>'s password file with <a href="https://www.samba.org/samba/docs/current/man-html/smbpasswd.8.html" target="_blank" class="code-doc">`smbpasswd`</a> in order to connect with it from macOS:
```console
sudo smbpasswd -a pi
```

If you want to add other users you can follow this guide: <a href="https://www.thegeekdiary.com/how-to-add-or-delete-a-samba-user-under-linux/" target="_blank">How to add or delete a samba user under Linux</a>

Now you can test if your configuration is free from errors with <a href="https://www.samba.org/samba/docs/current/man-html/testparm.1.html" target="_blank" class="code-doc">`testparm`</a>:
```console
sudo testparm -s
```
[output]
```
pi@raspberrypi:~ $ sudo testparm -s
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Weak crypto is allowed
Server role: ROLE_STANDALONE

# Global parameters
[global]
	log file = /var/log/samba/log.%m
	logging = file
	map to guest = Bad User
	max log size = 1000
	obey pam restrictions = Yes
	pam password change = Yes
	panic action = /usr/share/samba/panic-action %d
	passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
	passwd program = /usr/bin/passwd %u
	server min protocol = SMB2
	server role = standalone server
	unix password sync = Yes
	usershare allow guests = Yes
	fruit:nfs_aces = no
	fruit:delete_empty_adfiles = yes
	fruit:wipe_intentionally_left_blank_rfork = yes
	fruit:zero_file_id = yes
	fruit:posix_rename = yes
	fruit:veto_appledouble = no
	fruit:model = MacSamba
	fruit:metadata = stream
	idmap config * : backend = tdb
	inherit permissions = Yes
	vfs objects = catia fruit streams_xattr


[TimeMachine]
	comment = TimeMachine
	inherit acls = Yes
	path = /media/pi/TimeMachine
	read only = No
	valid users = pi
	fruit:time machine = yes


[NAS]
	comment = NAS
	inherit acls = Yes
	path = /media/pi/NAS
	read only = No
	valid users = pi
pi@raspberrypi:~ $ 
```

Your output should look similar to the mine just above. ⬆

Then restart the <a href="https://www.samba.org/" target="_blank">Samba</a> service to reload the configuration changes:
```console
sudo service smbd reload
```

You can now connect to your <a href="https://www.samba.org/" target="_blank">Samba</a> shares from your Mac by pressing `Command-K` in Finder and enter the IP of your Raspberry Pi like this: `smb://192.168.0.200/TimeMachine`  
<img alt="RealVNC Viewer Raspberry Pi Desktop" src="/img/blog/03/Connect_to_server__TimeMachine.png" class="blog-image"/>  
 Authenticate with your user credentials when prompted.

<br/>

### Configuring Avahi

You can make the process of logging in to your TimeMachine and NAS from your Mac a lot easier by configuring <a href="https://www.avahi.org/" target="_blank">Avahi</a> to advertise the <a href="https://www.samba.org/" target="_blank">Samba</a> shares on your network.

You have to create a new service in `/etc/avahi/services/samba.service`:
```console
sudo nano /etc/avahi/services/samba.service
```

Then add the following configuration that I shamelessly stole from <a href="https://mudge.name/2019/11/12/using-a-raspberry-pi-for-time-machine/#configuring-avahi" target="_blank">here</a> (where a detailed description is available):
```xml
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">%h</name>
  <service>
    <type>_smb._tcp</type>
    <port>445</port>
  </service>
  <service>
    <type>_adisk._tcp</type>
    <port>9</port>
    <txt-record>dk0=adVN=TimeMachine,adVF=0x82</txt-record>
    <txt-record>sys=adVF=0x100</txt-record>
  </service>
  <service>
    <type>_device-info._tcp</type>
    <port>9</port>
    <txt-record>model=MacPro7,1@ECOLOR=226,226,224</txt-record>
  </service>
</service-group>
```
In short, you are declaring 3 services.

The first 2 services configures <a href="https://www.avahi.org/" target="_blank">Avahi</a> to follow Apples <a href="https://developer.apple.com/library/archive/releasenotes/NetworkingInternetWeb/Time_Machine_SMB_Spec/index.html#//apple_ref/doc/uid/TP40017496-CH1-SW1" target="_blank">Advertising Time Machine Availability Through Bonjour</a> documentation by:  
<span>
    1. Advertising that your Raspberry Pi is running a <a href="https://www.samba.org/" target="_blank">Samba</a> on port 445.  
    2. Advertising that the share named "TimeMachine" is available for TimeMachine backups.  
</span>

The 3'rd service configures <a href="https://www.avahi.org/" target="_blank">Avahi</a> to advertise the Raspberry Pi as a <a href="https://support.apple.com/kb/SP810" target="_blank">Mac Pro (Rack, 2019)</a>, which sets a nice icon in macOS Finder.  
You can change this icon to any device name listed in `/System/Library/CoreServices/CoreTypes.bundle/Contents/Info.plist`.  Open the file with Xcode and search for "@ECOLOR" to easily highlight your available options.  
<img alt="RealVNC Viewer Raspberry Pi Desktop" src="/img/blog/03/Xcode_Device_Names_for_Avahi.png" class="blog-image"/>

<a href="https://www.avahi.org/" target="_blank">Avahi</a> should pick up your changes automatically and our Raspberry Pi should now appear in the sidebar of Finder.  
<img alt="RealVNC Viewer Raspberry Pi Desktop" src="/img/blog/03/Raspberry_Pi_in_Finder.png" class="blog-image"/>

If it doesn't, you can restart the <a href="https://manpages.debian.org/bullseye/avahi-daemon/avahi-daemon.8.en.html" target="_blank" class="code-doc">`avahi-daemon`</a> <a href="https://manpages.debian.org/bullseye/init-system-helpers/service.8.en.html" target="_blank" class="code-doc">`service`</a>:
```console
sudo service avahi-daemon restart
```

<br/>

**And there you have it! Your very own Raspberry Pi TimeMachine and NAS server.**

## NAS Backup (Optional)

Make sure the BACKUP drive is mounted before running the script. 

### <a href="https://manpages.debian.org/bullseye/rsync/rsync.1.en.html" target="_blank" class="code-doc">`rsync`</a> backup script

<a href="https://manpages.debian.org/bullseye/coreutils/touch.1.en.html" target="_blank" class="code-doc">`touch`</a>  
<a href="https://manpages.debian.org/bullseye/coreutils/chmod.1.en.html" target="_blank" class="code-doc">`chmod`</a>
```console
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
```console
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

- <a href="https://wiki.samba.org/index.php/Configure_Samba_to_Work_Better_with_Mac_OS_X" target="_blank">Configure Samba to Work Better with Mac OS X</a>
- <a href="https://mudge.name/2019/11/12/using-a-raspberry-pi-for-time-machine/" target="_blank">Using a Raspberry Pi for Time Machine</a>
- <a href="https://jansblog.org/2021/05/16/samba-based-timemachine-with-big-sur/" target="_blank">Samba-based TimeMachine with Big Sur</a>

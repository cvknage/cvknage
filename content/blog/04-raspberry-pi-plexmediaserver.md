+++
date = "2023-01-21T20:00:00+01:00"
draft = false
title = "Raspberry Pi as Plex Media Server"
slug = "raspberry-pi-plexmediaserver"
author = "Christophe Knage"
+++

**This post covers installing Plex Media Server on a Raspberry Pi 4 B and keeping Plex up to date.**

<a href="https://www.plex.tv" target="_blank">Plex</a> is in my opinion the best way to enjoy your media. Whether it's your old DVD collect, Lite TV or family photos, <a href="https://www.plex.tv" target="_blank">Plex</a> can make it look beautiful and easy to access.

The brain of <a href="https://www.plex.tv" target="_blank">Plex</a> is <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a>.  

In this post I will go through 2 methods of installing <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a> on your Raspberry Pi.

> This is part 4 of 4 in a mini series where we will configure a headless Raspberry Pi 4 B as an efficient home server, capable of hosting <a href="https://en.wikipedia.org/wiki/Network-attached_storage" target="_blank">Network Attached Storage (NAS)</a>, <a href="https://support.apple.com/en-gb/HT201250" target="_blank">TimeMachine</a> and <a href="https://www.plex.tv" target="_blank">Plex Media Server</a>.

**Prerequisites**

Hardware Requirements
- <a href="https://www.raspberrypi.com/products/raspberry-pi-4-model-b/" target="_blank">Raspberry Pi 4 B</a>
- <a href="https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards" target="_blank">microSD</a>
- External Storage (e.g. SSD in a USB 3.0 enclosure)

Software Requirements
- <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a>

<br/>

{{<toc>}}

## Download and Install from plex.tv

The easiest way to get <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a> installed on your Raspberry Pi is downloading it right off the official downloads page: 

Browse to: <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">https&#58;//www.plex.tv/media-server-downloads/#plex-media-server</a> 

{{<zoom-image>}}
<img alt="Plex Media Server Downloads Page" src="/img/blog/04/Plea_Media_Server__Download_Page.png" class="blog-image"/>
{{</zoom-image>}}

You need to know which architecture your Raspberry Pi OS is running, use <a href="https://manpages.debian.org/bullseye/util-linux/lscpu.1.en.html" target="_blank" class="code-doc">`lscpu`</a>:
```console
lscpu
```
[output]
```
pi@raspberrypi:~ $ lscpu
Architecture:                    armv7l
Byte Order:                      Little Endian
CPU(s):                          4
On-line CPU(s) list:             0-3
Thread(s) per core:              1
Core(s) per socket:              4
Socket(s):                       1
Vendor ID:                       ARM
Model:                           3
...
```

Look for the line that says `Architecture:`

Then download the Debian version matching your architecture.  
You can do this directly from the console on your Raspberry Pi with <a href="https://manpages.debian.org/bullseye/wget/wget.1.en.html" target="_blank" class="code-doc">`wget`</a> (remember to use an updated link):
```console
wget https://downloads.plex.tv/plex-media-server-new/1.30.2.6563-3d4dc0cce/debian/plexmediaserver_1.30.2.6563-3d4dc0cce_armhf.deb
```

Finally install <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a> with <a href="https://manpages.debian.org/bullseye/dpkg/dpkg.1.en.html" target="_blank" class="code-doc">`dpkg`</a> (replacing the file name with the package you downloaded):
```console
sudo dpkg -i plexmediaserver_1.30.2.6563-3d4dc0cce_armhf.deb
```

You can now access your <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a> in your from your Raspberry Pi's IP on port 32400 e.g. `http://192.168.0.200:32400` or on `http://raspberrypi.local:32400` if you installed <a href="https://www.avahi.org/" target="_blank">Avahi</a> in [part 3]({{<relref"/blog/03-raspberry-pi-timemaching-and-nas#install-dependencies">}} "Install Avahi") (you do not need to configure <a href="https://www.avahi.org/" target="_blank">Avahi</a> - just install it).

<br/>

**There you have it, <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a>, Installed!**  

<br/>

### Update from plex.tv

To update your Plex Media Server installation, you can simply follow the [install instructions](#download-and-install-from-plextv) again from the top, and install the updated version on top of your existing install.

## Install from official Plex repository

You can install <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a> with <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`apt`</a> directly from the official Plex repository.

This however requires a bit of setting up first. It is well worth it in the end though, as it makes updating your Plex Media Server installation a lot easier.

### Setup official Plex repository

First you need to install <a href="https://manpages.debian.org/bullseye/apt/apt-transport-https.1.en.html" target="_blank" class="code-doc">`apt-transport-https`</a>.  
This allows the <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`apt`</a> package manager to install packages over the `https` which the Plex repository uses:
```console
sudo apt update
sudo apt install apt-transport-https
```

Then you need to download the Plex <a href="https://manpages.debian.org/bullseye/gpg/gpg.1.en.html" target="_blank" class="code-doc">`gpg`</a> key using <a href="https://manpages.debian.org/bullseye/wget/wget.1.en.html" target="_blank" class="code-doc">`wget`</a> and add it to your keyring with <a href="https://manpages.debian.org/bullseye/coreutils/tee.1.en.html" target="_blank" class="code-doc">`tee`</a>:
```console
wget -O- https://downloads.plex.tv/plex-keys/PlexSign.key | gpg --dearmor | sudo tee /usr/share/keyrings/plex-repository-keyring.gpg > /dev/null
```

After that you can add the official Plex <a href="https://manpages.debian.org/bullseye/dpkg-dev/deb.5.en.html" target="_blank" class="code-doc">`deb`</a> repository to the sources:
```console
echo deb [signed-by=/usr/share/keyrings/plex-repository-keyring.gpg] https://downloads.plex.tv/repo/deb public | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
```

<br/>

### Install Plex Media Server from official Plex repository

With the official Plex repository added, you need to run <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`apt update`</a> again, to refresh the package list.  
Then you can install <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a>:
```console
sudo apt update
sudo apt install plexmediaserver
```

<br/>

**There you have it, <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a>, Installed!**  

<br/>

### Update from official Plex repository

When a new version is released, you can simply update Plex Media Server with <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`apt`</a>:
```console
sudo apt update
sudo apt install --only-upgrade plexmediaserver
```

#### Troubleshooting: Updating from repository

During the installation process, the official Plex repository is sometimes disabled. To enable the repository, open the `plexmediaserver.list` file in <a href="https://manpages.debian.org/bullseye/nano/nano.1.en.html" target="_blank" class="code-doc">`nano`</a> and uncomment the line starting with "deb":
```console
sudo nano /etc/apt/sources.list.d/plexmediaserver.list
```

Your `plexmediaserver.list` file should look similar to this:
```bash
# When enabling this repo please remember to add the PlexPublic.Key into the apt setup.
# wget -q https://downloads.plex.tv/plex-keys/PlexSign.key -O - | sudo apt-key add -
deb https://downloads.plex.tv/repo/deb/ public main
```

## Update Plex Media Server Automatically

Whether you downloaded and installed Plex Media plex.tv or from the official repository with <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`apt`</a>, it is easy to keep your installation up to date automatically.

> **Note:**  
If you followed [Download and Install from plex.tv](#download-and-install-from-plextv), you first have to [Setup official Plex repository](#setup-official-plex-repository) before continuing.

With the official Plex repository set up on your Raspberry Pi, you are now ready to automate the Plex Media Server update process.

First, create a script file and make it executable with <a href="https://manpages.debian.org/bullseye/coreutils/touch.1.en.html" target="_blank" class="code-doc">`touch`</a> and <a href="https://manpages.debian.org/bullseye/coreutils/chmod.1.en.html" target="_blank" class="code-doc">`chmod`</a>:
```console
touch ./Documents/plex-mediaserver-update.sh
chmod +x ./Documents/plex-mediaserver-update.sh
```

Then open the file `plex-mediaserver-update.sh` in <a href="https://manpages.debian.org/bullseye/nano/nano.1.en.html" target="_blank" class="code-doc">`nano`</a>:
```console
nano ./Documents/plex-mediaserver-update.sh
```

Add this script to the file `plex-mediaserver-update.sh`:
```bash
#!/bin/bash

sudo apt update
sudo apt install --only-upgrade plexmediaserver
```

Open <a href="https://manpages.debian.org/bullseye/systemd-cron/crontab.1.en.html" target="_blank" class="code-doc">`crontab`</a> with the commend:
```console
crontab -e
```

At the end of the document, type the frequency of script execution in the following manner
```bash
0 5 * * 1 /home/pi/Documents/plex-mediaserver-update.sh
```

The above line means that the script with run **At 05:00 on Monday**. You can configure this according to your needs. I recommend using <a href="https://crontab.guru/#0_5_*_*_1" target="_blank">crontab.guru</a> to get the right settings for you.

The script will only install an update if one is available, so it is safe to run as frequently as you like. 

## Backup Plex Media Server Data

*"The Plex Media Server data directory contains nearly all the information about your server installation. This includes the database file with your library information as well as metadata, artwork, caches, and more."* - <a href="https://support.plex.tv/articles/202529153-why-is-my-plex-media-server-directory-so-large/" target="_blank">plex.tv</a> 

When you have spent time getting your Plex library set up just the way you like it. It can be a god idea to have a backup of the data directory, just in case a something goes wrong and your library gets messed up.

> **Note:**  
In [part 2 - Raspberry Pi Backup]({{<relref"/blog/02-raspberry-pi-backup">}} "Raspberry Pi Backup") I describe how to mount external storage to your Raspberry Pi and make a backup of the entire microCD Card.  
You can reference this section: [Setting up the external storage]({{<relref"/blog/02-raspberry-pi-backup#setting-up-the-external-storage">}} "Raspberry Pi Backup").

With external storage mounted to your Raspberry Pi, you can follow steps similar what you just did in: [Update Plex Media Server Automatically](#update-plex-media-server-automatically).

First, create a script file and make it executable with <a href="https://manpages.debian.org/bullseye/coreutils/touch.1.en.html" target="_blank" class="code-doc">`touch`</a> and <a href="https://manpages.debian.org/bullseye/coreutils/chmod.1.en.html" target="_blank" class="code-doc">`chmod`</a>:
```console
touch ./Documents/plex-media-backup.sh
chmod +x ./Documents/plex-media-backup.sh
```

Then open the file `plex-media-backup.sh` in <a href="https://manpages.debian.org/bullseye/nano/nano.1.en.html" target="_blank" class="code-doc">`nano`</a>:
```console
nano ./Documents/plex-media-backup.sh
```

Add this script to the file `plex-media-backup.sh` (make sure the `BACKUP_DIRECTORY` variable is correct for you):
```bash
#!/bin/bash

CURRENT_DATETIME=`date +"%Y-%m-%dT%H-%M-%S"`
BACKUP_DIRECTORY="/media/pi/NAS/Plex/BACKUPS"
BACKUP_FILENAME="${BACKUP_DIRECTORY}/PlexMedia-(${CURRENT_DATETIME}).tar.gz"

# Exit script if Backup Directory does not exist
if [ ! -d "${BACKUP_DIRECTORY}" ]; then
  echo "${BACKUP_DIRECTORY} does not exist"
  exit 1
fi

# tar & gzip Plex Media Server Data
sudo tar czPf "${BACKUP_FILENAME}" --exclude="/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Cache" "/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/"

# Delete backups older than specified number of days
find ${BACKUP_DIRECTORY} -maxdepth 1 -name "*.tar.gz"  -type f -mtime +90  -delete
```

Open <a href="https://manpages.debian.org/bullseye/systemd-cron/crontab.1.en.html" target="_blank" class="code-doc">`crontab`</a> with the commend:
```console
crontab -e
```

At the end of the document, type the frequency of script execution in the following manner
```bash
0 0 */10 * * /home/pi/Documents/plex-media-backup.sh
```

The above line means that the script with run **At 00:00 on every 10th day-of-month**. You can configure this according to your needs. I recommend using <a href="https://crontab.guru/#0_0_*/10_*_*" target="_blank">crontab.guru</a> to get the right settings for you.

<h1 style="font-size: 100%">Sources</h1>

- <a href="https://support.plex.tv/articles/categories/plex-media-server/" target="_blank">Plex Media Server</a>
- <a href="https://support.plex.tv/articles/200288586-installation/#:~:text=Download%20the%20.deb%20package" target="_blank">Install Plex Media Server on Debian Linux</a>
- <a href="https://support.plex.tv/articles/235974187-enable-repository-updating-for-supported-linux-server-distributions/#:~:text=DEB-based%20distros" target="_blank">Enable repository updating on Debian Linux</a>
- <a href="https://support.plex.tv/articles/202915258-where-is-the-plex-media-server-data-directory-located/" target="_blank">Where is the Plex Media Server data directory located?</a>
- <a href="https://linuxize.com/post/how-to-install-plex-media-server-on-ubuntu-20-04/" target="_blank">How to Install Plex Media Server on Ubuntu 20.04</a>
- <a href="https://stackoverflow.com/a/71384057" target="_blank">Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead</a>

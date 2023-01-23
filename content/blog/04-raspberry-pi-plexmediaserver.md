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

> This is part 4 of 5 in a mini series where we will configure a headless Raspberry Pi 4 B as an efficient home server, with an effective backup strategy, capable of hosting <a href="https://en.wikipedia.org/wiki/Network-attached_storage" target="_blank">Network Attached Storage (NAS)</a>, <a href="https://support.apple.com/en-gb/HT201250" target="_blank">TimeMachine</a>, <a href="https://www.plex.tv" target="_blank">Plex Media Server</a> and <a href="https://homebridge.io" target="_blank">Homebridge</a>.

**Prerequisites**

Hardware Requirements
- <a href="https://www.raspberrypi.com/products/raspberry-pi-4-model-b/" target="_blank">Raspberry Pi 4 B</a>
- <a href="https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards" target="_blank">microSD</a>
- External Storage (e.g. SSD in a USB 3.0 enclosure)

Software Requirements
- <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a>

<br/>

{{<toc>}}

## Install Plex Media Server from Downloads Page

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
To update your <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a> installation, continue reading below: [Updating Plex Media Server](#updating-plex-media-server).

## Install Plex Media Server from Repository

You can install <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a> with <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`apt`</a> directly from the official Plex repository.

This however requires a bit of setting up first. It is well worth it in the end though, as it makes [Updating Plex Media Server](#updating-plex-media-server) a lot easier.

### Setup Plex Media Server Repository

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

With the official Plex repository added, you need to run <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`apt update`</a> again, to refresh the package list.  
Then you can install <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a>:
```console
sudo apt update
sudo apt install plexmediaserver
```

<br/>

**There you have it, <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a>, Installed!**  
To update your <a href="https://www.plex.tv/media-server-downloads/#plex-media-server" target="_blank">Plex Media Server</a> installation, continue reading below: [Updating Plex Media Server](#updating-plex-media-server).

## Updating Plex Media Server

### Updating Plex Media Server Automatically

## Backup Plex Media Server Data

```console
pi@raspberrypi:~ $ touch ./Documents/plex-media-backup.sh
pi@raspberrypi:~ $ chmod +x ./Documents/plex-media-backup.sh
```

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

<h1 style="font-size: 100%">Sources</h1>

- <a href="https://www.electromaker.io/tutorial/blog/how-to-install-plex-on-raspberry-pi" target="_blank">How to Install Plex on the Raspberry Pi 4</a>
- <a href="https://pimylifeup.com/raspberry-pi-plex-server/" target="_blank">How to Setup a Raspberry Pi Plex Server</a>
- <a href="https://linuxize.com/post/how-to-install-plex-media-server-on-ubuntu-20-04/#updating-plex-media-server" target="_blank">Updating Plex Media Server</a>


- <a href="https://support.plex.tv/articles/categories/plex-media-server/" target="_blank">Plex Media Server</a>
- <a href="https://support.plex.tv/articles/200288586-installation/#:~:text=Download%20the%20.deb%20package" target="_blank">Install Plex Media Server on Debian Linux</a>
- <a href="https://support.plex.tv/articles/235974187-enable-repository-updating-for-supported-linux-server-distributions/#:~:text=DEB-based%20distros" target="_blank">Enable repository updating on Debian Linux</a>

- <a href="https://stackoverflow.com/a/71384057" target="_blank">Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead</a>
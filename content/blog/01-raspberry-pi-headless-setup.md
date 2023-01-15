+++
date = "2022-12-07T22:00:00+01:00"
draft = false
title = "Headless Raspberry Pi Server"
slug = "raspberry-pi-headless-setup"
author = "Christophe Knage"
+++

**This post covers setting up a headless Raspberry Pi 4 B from scratch.**

[Raspberry Pi](https://www.raspberrypi.com) has excellent documentation on how to setup and configure a Raspberry Pi. This post leans on the official documentation and focus on the parts that are essential to get up and running. 

> This is part 1 of 5 in a mini series where we will configure a headless Raspberry Pi 4 as an efficient home server, with an effective backup strategy, capable of hosting [Network Attached Storage (NAS)](https://en.wikipedia.org/wiki/Network-attached_storage), [TimeMachine](https://support.apple.com/en-gb/HT201250), [Plex Media Server](https://www.plex.tv) and [Homebridge](https://homebridge.io).

### Prerequisites

Hardware Requirements
- [Raspberry Pi 4 B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [microSD](https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards)

Software Requirements
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- [RealVNC viewer](https://www.realvnc.com/en/connect/download/viewer/) (Optional)

## Installing Raspberry Pi OS

Raspberry Pi have made it easy to install Raspberry Pi OS using [Raspberry Pi Imager](https://www.raspberrypi.com/software/) which can download the Raspberry Pi OS image automatically and install it on your micro SD card.  
<img alt="Raspberry Pi Imager" src="/img/blog/01/Raspberry_Pi_Imager.png" class="image"/>

For this install we are going to be using the Recommended Raspberry Pi OS (32-bit) with Desktop.  
<img alt="Raspberry Pi Imager Select OS" src="/img/blog/01/Raspberry_Pi_Imager__Select_OS.png" class="image"/>

For security reasons, WiFi and SSH is not enabled by default. To enable it we have to open the Advanced Menu.  
<img alt="Raspberry Pi Imager Select Advanced Settings" src="/img/blog/01/Raspberry_Pi_Imager__Click_Advanced_Settings.png" class="image"/>

In the Advanced options we are going to Enable SSH, Set username and password, Configure wireless LAN (don't forget to set the correct Wireless LAN country).  
<img alt="Raspberry Pi Imager Advanced Settings" src="/img/blog/01/Raspberry_Pi_Imager__Advanced_Settings.png" class="image"/>

Having chosen the options we want, we select out micro SD card from the menu and write the Raspberry Pi OS image to it.  
<img alt="Raspberry Pi Imager Write Image" src="/img/blog/01/Raspberry_Pi_Imager__Write_Image.png" class="image"/>

Eject the micro SD card, remove it from your computer and insert it in to the Raspberry Pi 4 B.  
Plug the USB-C power supply in to the Raspberry Pi 4 Bs power port.  
Give the Raspberry Pi 4 B some time to boot up - about 90 seconds or so on first boot.  

## Login and setup Raspberry Pi

With our Raspberry Pi 4 B now booted and on the network we can connect to it using `SSH`.  
If you are on **Windows**, and need to get acquainted with `SSH` read this before continuing: [Secure Shell (SSH)](https://learn.microsoft.com/en-us/windows/iot-core/connect-your-device/ssh).  
**macOS** and other [UNIX-like systems](https://en.wikipedia.org/wiki/Unix-like) have `SSH` preinstalled. Here is a quick guide: [OSXDaily - How to SSH](https://osxdaily.com/2017/04/28/howto-ssh-client-mac/)

Your Raspberry Pi 4 B can be found on your network using its hostname (example: `raspberrypi.local`). However to do this on **Windows**, [Bonjour for Windows](https://support.apple.com/kb/DL999) needs to be installed.

<br/>

### Logging in

First lets clear out any previous references to `raspberrypi.local`.  
Open a Terminal window and run the following command:
```console
ssh-keygen -R raspberrypi.local
```
Don't worry if you get a host not found error. This just means there where no previous references to `raspberrypi.local`.

Then sign in to your Raspberry Pi 4 B with the following command:
```console
ssh pi@raspberrypi.local
```
If the command hangs, press `Ctrl-C` and try again.  
If prompted with a warning, hit enter to accept the default (**yes**).  
When prompted, type the password you set in Raspberry Pi Imager and hit `Enter` - don't worry if the prompt doesn't seem to register that you are typing, it is.

<br/>

### Getting the latest updates

Before we start using our Raspberry Pi 4 B, we want to make sure that our system is up to date.

The easiest way to manage installing, upgrading, and removing software is using APT (Advanced Packaging Tool) from Debian.

Before installing software, you should update your package list with `apt update` using this command:
```console
sudo apt update
```

Next, `upgrade` all your installed packages to their latest versions with the following command:
```console
sudo apt full-upgrade
```
Note that `full-upgrade` is used in preference to a simple `upgrade`, as it also picks up any dependency changes that may have been made.

<br/>

**Congratulations! You now have a headless Raspberry Pi 4 B server.**

## RealVNC (Optional)

If you want to gain remote access to the Raspberry Pi's Desktop, read a bit further. 

<br/>

### Enable VNC Server

To gain graphical remote access to the Raspberry Pi 4 B, we have to enable the VNC Server with [`raspi-config`](https://www.raspberrypi.com/documentation/computers/configuration.html#the-raspi-config-tool):
```bash
sudo raspi-config
```
Then Navigate to **Interfacing Options**.  
Scroll down and select **VNC** › **Yes**  

<br/>

### Creating a Virtual Desktop

Because our Raspberry Pi 4 B is headless, it may not be running a graphical desktop. 

VNC Server can create a virtual desktop for you, giving you graphical remote access on demand. This virtual desktop exists only in your Raspberry Pi’s memory.

To create a virtual desktop, run the following command:
```console
vncserver
```
Make note of the IP address/display number that VNC Server will print to your Terminal (e.g. 192.167.5.149:1).  
(Note, the Raspberry Pi's hostname can also be used together with the display number e.g. `raspberrypi.local:1`)

On the device you’ll use to take control, enter this information into VNC Viewer.  

<br/>

To destroy a virtual desktop, run the following command:
```console
vncserver -kill :<display-number>
```
This will also stop any existing connections to this virtual desktop.

<br/>

### Installing RealVNC Viewer

Browse to: https://www.realvnc.com/en/connect/download/viewer/  
Download and install the version applicable for your system.

<br/>

### Connect over VNC

Launch RealVNC Viewer on your computer and enter the name of your Raspberry Pi 4 B server (`raspberrypi.local`) in to the address bar.  
<img alt="RealVNC Viewer New Connection" src="/img/blog/01/RealVNC_Viewer__New_Connection.png" class="image"/>  
When prompted enter your **username** and **password**.

<br/>

**And there you have it, the Raspberry Pi OS Desktop.**  
<img alt="RealVNC Viewer Raspberry Pi Desktop" src="/img/blog/01/RealVNC_Viewer__Raspberry_Pi_Desktop.png" class="image"/>

#
### Sources

- [Raspberry Pi Documentation - Installing the Operating System](https://www.raspberrypi.com/documentation/computers/getting-started.html#installing-the-operating-system)
- [Raspberry Pi Documentation - Updating and Upgrading Raspberry Pi OS](https://www.raspberrypi.com/documentation/computers/os.html#updating-and-upgrading-raspberry-pi-os)
- [Raspberry Pi Documentation - Virtual Network Computing (VNC)](https://www.raspberrypi.com/documentation/computers/remote-access.html#vnc)
- [Headless Raspberry Pi 4 Remote Desktop VNC Setup (Mac + Windows, 13 Steps)](https://desertbot.io/blog/headless-raspberry-pi-4-remote-desktop-vnc-setup)

<!--
<span style="font-weight:300;font-size:12px">
    <p style="margin: 0;">
        Raspberry Pi documentation is copyright &copy; 2012-2023 Raspberry Pi Ltd and is licensed under a <a href="https://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International</a> (CC BY-SA) licence.
    </p>
    <p style="margin: 0;">
        Some content originates from the <a href="http://elinux.org/">eLinux wiki</a> , and is licensed under a <a href="http://creativecommons.org/licenses/by-sa/3.0/">Creative Commons Attribution-ShareAlike 3.0 Unported</a> licence.
    </p>
</span>
-->

<style>
.image {
    width: 35em;
}
</style>

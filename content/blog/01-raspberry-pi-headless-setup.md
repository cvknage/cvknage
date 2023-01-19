+++
date = "2022-12-07T22:00:00+01:00"
draft = false
title = "Headless Raspberry Pi Server"
slug = "raspberry-pi-headless-setup"
author = "Christophe Knage"
+++

**This post covers setting up a headless Raspberry Pi 4 B from scratch.**

<a href="https://www.raspberrypi.com" target="_blank">Raspberry Pi</a> has excellent documentation on how to setup and configure a Raspberry Pi. This post leans on the official documentation and focus on the parts that are essential to get up and running. 

> This is part 1 of 5 in a mini series where we will configure a headless Raspberry Pi 4 B as an efficient home server, with an effective backup strategy, capable of hosting <a href="https://en.wikipedia.org/wiki/Network-attached_storage" target="_blank">Network Attached Storage (NAS)</a>, <a href="https://support.apple.com/en-gb/HT201250" target="_blank">TimeMachine</a>, <a href="https://www.plex.tv" target="_blank">Plex Media Server</a> and <a href="https://homebridge.io" target="_blank">Homebridge</a>.

### Prerequisites

Hardware Requirements
- <a href="https://www.raspberrypi.com/products/raspberry-pi-4-model-b/" target="_blank">Raspberry Pi 4 B</a>
- <a href="https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards" target="_blank">microSD</a>

Software Requirements
- <a href="https://www.raspberrypi.com/software/" target="_blank">Raspberry Pi Imager</a>
- <a href="https://www.realvnc.com/en/connect/download/viewer/" target="_blank">RealVNC viewer</a> (Optional)

## Installing Raspberry Pi OS

Raspberry Pi have made it easy to install Raspberry Pi OS using <a href="https://www.raspberrypi.com/software/" target="_blank">Raspberry Pi Imager</a> which can download the Raspberry Pi OS image automatically and install it on your micro SD card.  
<img alt="Raspberry Pi Imager" src="/img/blog/01/Raspberry_Pi_Imager.png" class="image"/>

For this install you are going to be using the Recommended Raspberry Pi OS (32-bit) with Desktop.  
<img alt="Raspberry Pi Imager Select OS" src="/img/blog/01/Raspberry_Pi_Imager__Select_OS.png" class="image"/>

For security reasons, WiFi and <a href="https://manpages.debian.org/bullseye/openssh-client/ssh.1.en.html" target="_blank" class="code-doc">`SSH`</a> is not enabled by default. To enable it you have to open the Advanced Menu.  
<img alt="Raspberry Pi Imager Select Advanced Settings" src="/img/blog/01/Raspberry_Pi_Imager__Click_Advanced_Settings.png" class="image"/>

In the Advanced options you are going to Enable <a href="https://manpages.debian.org/bullseye/openssh-client/ssh.1.en.html" target="_blank" class="code-doc">`SSH`</a>, Set username and password, Configure wireless LAN (don't forget to set the correct Wireless LAN country).  
<img alt="Raspberry Pi Imager Advanced Settings" src="/img/blog/01/Raspberry_Pi_Imager__Advanced_Settings.png" class="image"/>

Having chosen the options you want, select your micro SD card from the menu and write the Raspberry Pi OS image to it.  
<img alt="Raspberry Pi Imager Write Image" src="/img/blog/01/Raspberry_Pi_Imager__Write_Image.png" class="image"/>

Eject the micro SD card, remove it from your computer and insert it in to the Raspberry Pi.  
Plug the USB-C power supply in to the Raspberry Pis power port.  
Give the Raspberry Pi some time to boot up - about 90 seconds or so on first boot.  

## Login and setup Raspberry Pi

With your Raspberry Pi now booted and on the network you can connect to it using <a href="https://manpages.debian.org/bullseye/openssh-client/ssh.1.en.html" target="_blank" class="code-doc">`SSH`</a>.  
If you are on **Windows**, and need to get acquainted with <a href="https://manpages.debian.org/bullseye/openssh-client/ssh.1.en.html" target="_blank" class="code-doc">`SSH`</a> read this before continuing: <a href="https://learn.microsoft.com/en-us/windows/iot-core/connect-your-device/ssh" target="_blank">Secure Shell (SSH)</a>.  
**macOS** and other <a href="https://en.wikipedia.org/wiki/Unix-like" target="_blank">UNIX-like systems</a> have <a href="https://manpages.debian.org/bullseye/openssh-client/ssh.1.en.html" target="_blank" class="code-doc">`SSH`</a> preinstalled. Here is a quick guide: <a href="https://osxdaily.com/2017/04/28/howto-ssh-client-mac/" target="_blank">OSXDaily - How to SSH</a>

Your Raspberry Pi can be found on your network using its hostname (example: `raspberrypi.local`). However to do this on **Windows**, <a href="https://support.apple.com/kb/DL999" target="_blank">Bonjour for Windows</a> needs to be installed.

<br/>

### Logging in

First lets clear out any previous references to `raspberrypi.local`.  
Open a Terminal window and run the following command:
```console
ssh-keygen -R raspberrypi.local
```
Don't worry if you get a host not found error. This just means there where no previous references to `raspberrypi.local`.

Then sign in to your Raspberry Pi with the following command:
```console
ssh pi@raspberrypi.local
```
If the command hangs, press `Ctrl-C` and try again.  
If prompted with a warning, hit enter to accept the default (**yes**).  
When prompted, type the password you set in Raspberry Pi Imager and hit `Enter` - don't worry if the prompt doesn't seem to register that you are typing, it is.

<br/>

### Getting the latest updates

Before you start using your Raspberry Pi, you want to make sure that your system is up to date.

The easiest way to manage installing, upgrading, and removing software is using APT (Advanced Packaging Tool) from Debian.

Before installing software, you should update your package list with <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`apt update`</a> using this command:
```console
sudo apt update
```

Next, <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`upgrade`</a> all your installed packages to their latest versions with the following command:
```console
sudo apt full-upgrade
```
Note that <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`full-upgrade`</a> is used in preference to a simple <a href="https://manpages.debian.org/bullseye/apt/apt.8.en.html" target="_blank" class="code-doc">`upgrade`</a>, as it also picks up any dependency changes that may have been made.

<br/>

**Congratulations! You now have a headless Raspberry Pi server.**

## RealVNC (Optional)

If you want to gain remote access to the Raspberry Pi's Desktop, read a bit further. 

<br/>

### Enable VNC Server

To gain graphical remote access to the Raspberry Pi, you have to enable the VNC Server with <a href="https://www.raspberrypi.com/documentation/computers/configuration.html#the-raspi-config-tool" target="_blank" class="code-doc">`raspi-config`</a>:
```console
sudo raspi-config
```
Then Navigate to **Interfacing Options**.  
Scroll down and select **VNC** › **Yes**  

<br/>

### Installing RealVNC Viewer

Browse to: <a href="https://www.realvnc.com/en/connect/download/viewer/" target="_blank">https&#58;//www.realvnc.com/en/connect/download/viewer/</a>  
Download and install the version applicable for your system.

<br/>

### Connect over VNC

Launch RealVNC Viewer on your computer and enter the name of your Raspberry Pi server (`raspberrypi.local`) in to the address bar.  
<img alt="RealVNC Viewer New Connection" src="/img/blog/01/RealVNC_Viewer__New_Connection.png" class="image"/>  
When prompted enter your **username** and **password**.

<br/>

**And there you have it, the Raspberry Pi OS Desktop.**  
<img alt="RealVNC Viewer Raspberry Pi Desktop" src="/img/blog/01/RealVNC_Viewer__Raspberry_Pi_Desktop.png" class="image"/>

<br/>

#### Troubleshooting: Creating a Virtual Desktop

If you see a black screen instead of the Raspberry Pi Desktop, it may be because your Raspberry Pi isn't running a graphical desktop. 

VNC Server can create a virtual desktop for you, giving you graphical remote access on demand. This virtual desktop exists only in your Raspberry Pi’s memory.

To create a virtual desktop, run the following command:
```console
vncserver
```
Make note of the IP address/display number that VNC Server will print to your Terminal (e.g. 192.167.5.149:1).  
(Note, the Raspberry Pi's hostname can also be used together with the display number e.g. `raspberrypi.local:1`)

On the device you’ll use to take control, enter this information into VNC Viewer.

To destroy a virtual desktop, run the following command:
```console
vncserver -kill :<display-number>
```
This will also stop any existing connections to this virtual desktop.

#
### Sources

- <a href="https://www.raspberrypi.com/documentation/computers/getting-started.html#installing-the-operating-system" target="_blank">Raspberry Pi Documentation - Installing the Operating System</a>
- <a href="https://www.raspberrypi.com/documentation/computers/os.html#updating-and-upgrading-raspberry-pi-os" target="_blank">Raspberry Pi Documentation - Updating and Upgrading Raspberry Pi OS</a>
- <a href="https://www.raspberrypi.com/documentation/computers/remote-access.html#vnc" target="_blank">Raspberry Pi Documentation - Virtual Network Computing (VNC)</a>
- <a href="https://desertbot.io/blog/headless-raspberry-pi-4-remote-desktop-vnc-setup" target="_blank">Headless Raspberry Pi 4 Remote Desktop VNC Setup (Mac + Windows, 13 Steps)</a>

<!--
<span style="font-weight:300;font-size:12px">
    <p style="margin: 0;">
        Raspberry Pi documentation is copyright &copy; 2012-2023 Raspberry Pi Ltd and is licensed under a <a href="https://creativecommons.org/licenses/by-sa/4.0/" target="_blank">Creative Commons Attribution-ShareAlike 4.0 International</a> (CC BY-SA) licence.
    </p>
    <p style="margin: 0;">
        Some content originates from the <a href="http://elinux.org/" target="_blank">eLinux wiki</a> , and is licensed under a <a href="http://creativecommons.org/licenses/by-sa/3.0/" target="_blank">Creative Commons Attribution-ShareAlike 3.0 Unported</a> licence.
    </p>
</span>
-->

<style>
.image {
    width: 35em;
}
</style>

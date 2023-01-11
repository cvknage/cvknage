+++
date = "2022-12-07T22:00:00+01:00"
draft = false
title = "Headless Raspberry Pi Server"
slug = "raspberry-pi-headless-setup"
author = "Christophe Knage"
+++

**This post covers setting up a headless Raspberry Pi 4 B from scratch.**

> This is part 1 of 5 in a mini series where we will configure a headless Raspberry Pi 4 as an efficient home server, with an effective backup strategy, capable of hosting [Network Attached Storage (NAS)](https://en.wikipedia.org/wiki/Network-attached_storage), [TimeMachine](https://support.apple.com/en-gb/HT201250), [Plex Media Server](https://www.plex.tv) and [Homebridge](https://homebridge.io).

Hardware Requirements
- [Raspberry Pi 4 B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [microSD](https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards)

Software Requirements
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- [RealVNC viewer](https://www.realvnc.com/en/connect/download/viewer/) (Optional)

# Getting Started

[Raspberry Pi](https://www.raspberrypi.com) has excellent documentation on how to setup and configure a Raspberry Pi. This post wil lean heavily on the official documentation and focus on the essential parts to get up and running. 

## Installing Raspberry Pi OS

- Raspberry Pi Imager
- Advanced settings
    - Change hostname and password
    - Enable ssh
    - Add wifi configuration

## Login and setup Raspberry Pi
- ssh login
- update the Raspberry Pi

## RealVNC (Optional)
- Enable VNC
- [Creating a Virtual Desktop](https://www.raspberrypi.com/documentation/computers/remote-access.html#creating-a-virtual-desktop)

#
### Sources

- [Raspberry Pi Documentation - Installing the Operating System](https://www.raspberrypi.com/documentation/computers/getting-started.html#installing-the-operating-system)
- [Raspberry Pi Documentation - Virtual Network Computing (VNC)](https://www.raspberrypi.com/documentation/computers/remote-access.html#virtual-network-computing-vnc)
- [Headless Raspberry Pi 4 Remote Desktop VNC Setup (Mac + Windows, 13 Steps)](https://desertbot.io/blog/headless-raspberry-pi-4-remote-desktop-vnc-setup)

<span style="font-weight:300;font-size:12px">
    <p style="margin: 0;">
        Raspberry Pi documentation is copyright &copy; 2012-2023 Raspberry Pi Ltd and is licensed under a <a href="https://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International</a> (CC BY-SA) licence.
    </p>
    <p style="margin: 0;">
        Some content originates from the <a href="http://elinux.org/">eLinux wiki</a> , and is licensed under a <a href="http://creativecommons.org/licenses/by-sa/3.0/">Creative Commons Attribution-ShareAlike 3.0 Unported</a> licence.
    </p>
</span>

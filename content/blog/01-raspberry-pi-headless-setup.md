+++
date = "2022-12-07T22:00:00+01:00"
draft = false
title = "Headless Raspberry Pi Server"
slug = "raspberry-pi-headless-setup"
author = "Christophe Knage"
+++

This post covers setting up a headless Raspberry Pi 4 from scratch.

> This is part 1 of 4 in a mini series where we will configure a headless Raspberry Pi 4 as an efficient home server, with an effective backup strategy, capable of hosting [Network Attached Storage (NAS)](https://en.wikipedia.org/wiki/Network-attached_storage), [TimeMachine](https://support.apple.com/en-gb/HT201250), [Plex Media Server](https://www.plex.tv) and [Homebridge](https://homebridge.io).

In this post we will focus on:
- Setting up a headless Raspberry Pi
- Backing up the Raspberry Pi 4

Hardware Requirements
- Raspberry Pi 4
- microSD 
- External SSD with USB 3.0

Software Requirements
- [Raspberry Pi OS](https://www.raspberrypi.com/software/)
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- [PiShrink](https://github.com/Drewsif/PiShrink)
- (Optional) [RealVNC viewer](https://www.realvnc.com/en/connect/download/viewer/)

### Sources

- [Headless Raspberry Pi 4 Remote Desktop VNC Setup (Mac + Windows, 13 Steps)](https://desertbot.io/blog/headless-raspberry-pi-4-remote-desktop-vnc-setup)
- [Back Up Headless Raspberry Pi Zero with RPI-Clone or PiShrink](https://robotzero.one/headless-pi-zero-backup-clone/)

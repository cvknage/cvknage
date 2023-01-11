+++
date = "2022-12-08T20:00:00+01:00"
draft = true
title = "Raspberry Pi Backup"
slug = "raspberry-pi-backup"
author = "Christophe Knage"
+++

**This post covers implementing an effective backup strategy.**

> This is part 2 of 5 in a mini series where we will configure a headless Raspberry Pi 4 as an efficient home server, with an effective backup strategy, capable of hosting [Network Attached Storage (NAS)](https://en.wikipedia.org/wiki/Network-attached_storage), [TimeMachine](https://support.apple.com/en-gb/HT201250), [Plex Media Server](https://www.plex.tv) and [Homebridge](https://homebridge.io).

Hardware Requirements
- [Raspberry Pi 4 B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [microSD](https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards)
- External SSD with USB 3.0

Software Requirements
- [PiShrink](https://github.com/Drewsif/PiShrink)

# Backing up the Raspberry Pi

### Sources

- [Back Up Headless Raspberry Pi Zero with RPI-Clone or PiShrink](https://robotzero.one/headless-pi-zero-backup-clone/)
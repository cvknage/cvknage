+++
date = "2022-12-07T22:00:00+01:00"
draft = true
title = "Raspberry Pi as TimeMachine and NAS"
slug = "raspberry-pi-timemachine-and-nas"
author = "Christophe Knage"
+++

**This post covers configuring a Raspberry Pi 4 B as a TimeMachine and NAS Server.**

> This is part 3 of 5 in a mini series where we will configure a headless Raspberry Pi 4 B as an efficient home server, with an effective backup strategy, capable of hosting [Network Attached Storage (NAS)](https://en.wikipedia.org/wiki/Network-attached_storage), [TimeMachine](https://support.apple.com/en-gb/HT201250), [Plex Media Server](https://www.plex.tv) and [Homebridge](https://homebridge.io).

In this post we will focus on:
- Configuring a SMB (Samba) with a NSA and TimeMAchine network share
- (Bonus) Backing up the NAS

Hardware Requirements
- [Raspberry Pi 4 B](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [microSD](https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards)
- External SSD with USB 3.0
- (Bonus) External HDD with USB

Software Requirements
- [Samba](https://www.samba.org/)
- [Avahi](https://www.avahi.org/)

### Sources

- [Using a Raspberry Pi for Time Machine](https://mudge.name/2019/11/12/using-a-raspberry-pi-for-time-machine/#configuring-avahi)

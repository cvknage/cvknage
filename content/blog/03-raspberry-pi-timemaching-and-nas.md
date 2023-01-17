+++
date = "2022-12-07T22:00:00+01:00"
draft = true
title = "Raspberry Pi as TimeMachine and NAS"
slug = "raspberry-pi-timemachine-and-nas"
author = "Christophe Knage"
+++

**This post covers configuring a Raspberry Pi 4 B as a TimeMachine and NAS Server.**

> This is part 3 of 5 in a mini series where we will configure a headless Raspberry Pi 4 B as an efficient home server, with an effective backup strategy, capable of hosting <a href="https://en.wikipedia.org/wiki/Network-attached_storage" target="_blank">Network Attached Storage (NAS)</a>, <a href="https://support.apple.com/en-gb/HT201250" target="_blank">TimeMachine</a>, <a href="https://www.plex.tv" target="_blank">Plex Media Server</a> and <a href="https://homebridge.io" target="_blank">Homebridge</a>.

In this post we will focus on:
- Configuring a SMB (Samba) with a NSA and TimeMAchine network share
- (Bonus) Backing up the NAS

Hardware Requirements
- <a href="https://www.raspberrypi.com/products/raspberry-pi-4-model-b/" target="_blank">Raspberry Pi 4 B</a>
- <a href="https://www.raspberrypi.com/documentation/computers/getting-started.html#sd-cards" target="_blank">microSD</a>
- External SSD with USB 3.0
- (Bonus) External HDD with USB

Software Requirements
- <a href="https://www.samba.org/" target="_blank">Samba</a>
- <a href="https://www.avahi.org/" target="_blank">Avahi</a>

### Sources

- <a href="https://mudge.name/2019/11/12/using-a-raspberry-pi-for-time-machine/#configuring-avahi" target="_blank">Using a Raspberry Pi for Time Machine</a>

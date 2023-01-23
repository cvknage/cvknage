+++
date = "2023-01-21T20:00:00+01:00"
draft = false
title = "Raspberry Pi as Plex Media Server"
slug = "raspberry-pi-plexmediaserver"
author = "Christophe Knage"
+++

**This post covers installing Plex Media Server on a Raspberry Pi 4 B and keeping Plex up to date.**

<a href="https://www.plex.tv" target="_blank">Plex</a> is in my opinion the best way to enjoy your media. Whether it's your old DVD collect, Lite TV or family photos, <a href="https://www.plex.tv" target="_blank">Plex</a> can make it look beautiful and easy to access.

The brain of <a href="https://www.plex.tv" target="_blank">Plex</a> is <a href="https://support.plex.tv/articles/categories/plex-media-server/" target="_blank">Plex Media Server</a>.  

In this post I will go through 2 methods of installing <a href="https://support.plex.tv/articles/categories/plex-media-server/" target="_blank">Plex Media Server</a> on your Raspberry Pi.

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

<h1 style="font-size: 100%">Sources</h1>

- <a href="https://www.electromaker.io/tutorial/blog/how-to-install-plex-on-raspberry-pi" target="_blank">How to Install Plex on the Raspberry Pi 4</a>
- <a href="https://pimylifeup.com/raspberry-pi-plex-server/" target="_blank">How to Setup a Raspberry Pi Plex Server</a>
- <a href="https://linuxize.com/post/how-to-install-plex-media-server-on-ubuntu-20-04/#updating-plex-media-server" target="_blank">Updating Plex Media Server</a>
- <a href="https://support.plex.tv/articles/categories/plex-media-server/" target="_blank">Plex Media Server</a>
- <a href="https://support.plex.tv/articles/200288586-installation/#:~:text=Download%20the%20.deb%20package" target="_blank">Install Plex Media Server on Debian Linux</a>
- <a href="https://support.plex.tv/articles/235974187-enable-repository-updating-for-supported-linux-server-distributions/#:~:text=DEB-based%20distros" target="_blank">Enable repository updating on Debian Linux</a>
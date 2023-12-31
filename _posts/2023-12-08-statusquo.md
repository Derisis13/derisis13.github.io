---
layout: post
title: "Server Setup Part 0 - Status Quo"
tag: "home server"
---

Since August I've been upgrading my home server setup.
It's not yet 100% complete, but most of the architectural decisions are already behind me.
I wish to document this process so that others can learn from it and as a reminder for myself if I ever forget how I did something.
This is part zero of my writeup, which'll be about the hardware and software used prior to the upgrade.
This should serve as a comparison baseline.


## NAS

I had a 2-bay Synology NAS for some years now.
It has been passed down to me from a family member along with drives to populate it.
It's configured in RAID-1 with 2x 3TB HDDs.
This capacity was almost filled up, which meant it was time to migrate from it.

This NAS ran an SMB server, an ISCSI target, a VPN server, a DDNS updater, streamed music and had a BitTorrent client.
The processor and RAM limitations crippled the responsiveness of these services, and the configuration was very limited as well.
It was a good computer, but it no longer satisfied my needs - at least not for the price I wanted to pay.

## Linux server

To expand into additional services that my NAS couldn't provide, I got a cheap laptop with a broken hinge (from a family member as well) and installed the XFCE spin of Fedora workstation on it.
I like this device for how simple it is: 5W idle power draw, 4-core Intel CPU (passively cooled by a piece of metal), the entire board being just one card, except the socketed RAM and the WLAN card (which I found has no black/whitelist).
Its IO is limited to 2 USB, 2 SATA (one for ODD) and 100MB internet.
The keyboard and screen were nice to have when I started out, and the battery came in useful during power outages.

This would have been a terrible fileserver but it ran PiHole and HomeAssistant in docker with great stability until recently the network interface started having issues.
Fun fact: this was also the machine I used for building Fedora packages for ani-cli.

## What won't change now

I'll go over a few devices I use at home but won't change now.
They are still relevant as services will interact with them.

I picked up a decent-size UPS from the trash a few years ago - it turned out to only need a new battery.
Now it has backup power for my NAS and router.

My networking is done by an ISP-provided all-in-one, which I hate but don't want to change just now.
There's also an unmanaged network switch to provide extra wired connections.

For media, I have a Chromecast (TV is not smart), a Pi 3 with Kodi and a Pi 1 with Volumio.
With the exception of the Chromecast they get their files trough mounted SMB shares.

There are a few IoT devices with open firmware I use for home automation and some that aren't connected to the internet because their firmware is proprietary, outdated and they would be a security risk to my network.

## Comming up

1. Host System - the metal that'll run my services
2. Filesystem layout - from disks to directories
3. Networking - connecting to the web
4. Docker and nextcloud - how not to get in your own way
5. Torrent and media management - My Lord, is that legal?
6. Media serving - the forgotten world of DLNA and UPNP/AV
7. Home automation - my home is smart
8. Backup strategy - because RAID is not a backup
9. Migration - moving it all in

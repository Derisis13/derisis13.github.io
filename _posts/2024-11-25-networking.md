---
layout: post
title: "Server Setup Part 2 - Networking"
tag: "home server"
---

This is part three of my server writeup.
I'll take a look at the networking services running on my homelab.
As usual, this is not a tutorial but merely a reminder for myself of what I have (or had) and a source of inspiration for others.


# Internal networking

At home, I have a decent internet connection: 1 GB/s symmetric, nominally.
The reality of this is outside the scope of this essay, but in the end, it's sufficient for my activities.
Most importantly, I'm not behind CGNAT.
The ISP-provided modem/router/AP all-in-one device is acceptable, and I'm satisfied with it in terms of configurability, except for the fact that it runs an OS whose source code I can't access (even though it's based on OpenWrt).
All my networking equipment supports 1 Gbit or less, as I currently don't need more.

In terms of network interfaces for my server, I run a virtual bridge network behind the RockPro64's single gigabit Ethernet port.
Setting it up was a bit tricky (as is the case for every IP change).
Be absolutely sure to set it to the correct static IP, default gateway, and DNS server.
Having a bridge network is mostly beneficial for virtualization, and even though Docker does its own networking, I don’t mind it.
It imposes no drawbacks while providing flexibility.

Docker containers are accessed through port forwards, for which I prefer to use the default port when available, only remapping if there’s a collision.
Many services (e.g., qBittorrent) will expect you to use a specific port and require extra configuration internally if you want to use another.
The OpenMediaVault control panel was also moved from ports 80/443 to allow Nextcloud to use them.
I no longer keep a homepage (eg. Homarr) running to remind me of the assigned ports—you’ll see why later.

Pi-hole is used for DNS, as it allows me to block network traffic according to my taste.
Currently, I have lists loaded against telemetry, ads, and pornography.
If I weren’t so lazy, I would have migrated to its DHCP server (in addition to providing DNS), but the ISP router does the job just fine—except that I can’t back up its config.
Using Unbound in conjunction with Pi-hole would be ideal, as the recursive DNS solution provided by Unbound is more private than forwarding all my queries to 8.8.8.8 or 1.1.1.1.
Unfortunately, I’ve failed to find a working AArch64 image for it when I tried, they kept restarting Unbound perpetually.


# External networking

My homelab has a fully qualified domain name (FQDN) because it’s required for Nextcloud (and its HTTPS certificates).
Since I don’t want to pay for a static IP, I use DuckDNS for dynamic DNS, as they provide free domains.
This domain is then further divided into subdomains by a Caddy reverse proxy on my homelab to forward traffic to my various services.
This is why I no longer need a homepage.
The best thing is that Caddy provides Let’s Encrypt certificates and HTTPS encryption, even if the underlying services don’t support it.
This even works with DuckDNS domains, but a special container with built-in support is required.

I currently don’t have a VPN set up, as my laptop has issues with WireGuard (even though it works fine from my phone) or I’ve misconfigured it.
I haven’t bothered with it further, as my most important services are proxied out and accessible from the internet.
If I ever upgrade or change distros, I’ll give WireGuard another chance.


# Final words

Although I have public IPv6, DuckDNS is yet to support it.
Luckily, I don’t need it for my server just yet.

I’m considering getting a regular domain instead of the one provided by DuckDNS, but so far, the prices I’ve seen are pretty steep.
If I did get one, it’d likely be to host my own email server.
Because I’m somewhat paranoid about sharing my IP/domain (mostly due to fear of DoS attacks), this blog will continue to exist on GitHub Pages.

I’m thinking about installing an x86 machine to take over networking, home automation, and maybe media server duties, but for now, that’s only something I’ve experimented with.

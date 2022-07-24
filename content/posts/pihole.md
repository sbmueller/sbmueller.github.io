+++
title = "Device independent AdBlock with Pi-Hole"
tags = ["hacking"]
date = "2017-02-25T12:36:54+01:00"
+++

Ads are annoying. Not only when trying to read morning news and more than half
of the website is spammed with moving gifs (or flash ads, the worst kind of
ads), but also they are able to spy on you and track you browsing behaviour.

Of course, most people are using browser extensions like [AdblockPlus]
(https://adblockplus.org/) or (my favorite) [uBlockOrigin]
(https://github.com/gorhill/uBlock). But what about your devices that do
not support extensions? For instance, I own a Samsung Smart TV and there is no
way Samsung is letting me install some privacy enhancing apps. This is exactly
where [Pi-Hole](https://pi-hole.net/) comes in.

Pi-Hole is a service meant to run on a Raspberry Pi (we all have at least one,
right?) and serves as a DNS server in your local network. Instead of resolving
all hostnames without thinking, like your router or any other DNS server would
do, Pi-Hole has a Blacklist of malicious domains providing either Ads or
Trackers. These domains are not getting resolved, preventing any device to
access them as long as they are trying to reach by hostname and not by IP. By
entering my local Raspberry Pi as DNS server address in my Samsung TV, I found
out that it was spying on me. A lot. There were several requests every few
seconds, trying to reach log-ingestion-eu.samsungacr.com. ACR in this URL
stands for "Automatic Content Recognition". So my TV was trying to gather data
on what I am watching, when and how long. I'm glad I can prevent this from now
on; shame on you Samsung!

At the same time, I technically don't need a AdBlock plugin on my browser
anymore, saving some memory and CPU (like 0.1%). The better thing is, many
websites that aren't meant to be accessed by AdBlocked browsers still work with
Pi-Hole, since the blocking mechanism works different from browser
extensions.

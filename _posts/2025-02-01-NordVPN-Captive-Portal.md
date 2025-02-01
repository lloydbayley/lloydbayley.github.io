---
title: "NordVPN & Captive Portals"
categories: [software, mac apps]
tags: [software]     # TAG names should always be lowercase
---

### Captive Portals & NordVPN

I've discovered a solution to a problem that was right in front of me the whole time and due to a lack of reading, I unintentionally caused the problem myself.

Lately, I've been having a problem logging into captive portals on my Macbook Air.
For example;

I connected to a wifi AP from a freshly-installed Tasmota install so that I could configure it.
The wifi network connects, however, when the captive portal comes up, it was blank and said it couldn't connect.
Weird, I thought. So I tried connecting to the IP shown as the 'router' (tasmota device - 192.168.4.1) and it wouldn't respond. I checked it with a ping and it timed out.

I've tried all sorts of things and it works on other Macs I have here but this is my everyday laptop, so I need it on here.

After much messing about, I found I had turned on an option in NordVPN that seems innocuous-enough at first glance but I missed the 'description' of the option.

The option in question is: "Stay Invisible On A Local Network"
The description afterwards tells you that you won't be able to access other network devices whilst the VPN is connected. - That's the bit I didn't see.

So, when I was connecting to the 'untrusted' wifi network, NordVPN switched on and therefore denied access to 'local' IPs.

Once I turned this option off, everything works as expected.

I'm posting this here in-case it helps someone else.


Moral of the story:

Remember, a few hours of trial and error can save you several minutes of looking at the README.


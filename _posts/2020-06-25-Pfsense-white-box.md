---
title: "The ASRock J3355b-itx pfSense box"
date: 2020-06-25 10:31:25 -0300
tags: homelab homeserver firewall network
header:
  overlay_image: "/assets/posts/2020-06-25-Pfsense-white-box/header.jpg"
toc: true
toc_label: "Table of Contents"
toc_icon: "list"
---

# Introduction
[pfSense CE](https://www.pfsense.org/) is a free, [open-source](https://github.com/pfsense/pfsense), and very popular **firewall/router** that runs on [FreeBSD](https://www.freebsd.org/) and is developed by [Netgate](https://www.netgate.com/).  Even though Netgate sells [official pfSense appliances](https://www.netgate.com/products/appliances/), it's possible to build your own, custom-made pfSense box for (way) less money (at the expense of way less support from Netgate). 

In this article, I talked about **my ASRock mini-ITX pfSense box project**.  In the first part, I listed all the necessary hardware to build the box, and in the second part, I briefly talked about the software.

[top](#){: .btn .btn--small .btn--light-outline}

# Hardware
This project has five main hardware components, namely the **motherboard** (mobo), the **network interface card** (NIC), **storage**, **case**, and **power supply unit** (PSU).

## Motherboard
Chances are that if you search for ```pfSense white-box```, you'll find someone mentioning the [**ASRock J3355b-itx**](https://www.asrock.com/mb/Intel/J3355B-ITX/).  This is definitely **not a top of the line** mobo but it comes with a **passively cooled [Intel dual-core processor](https://ark.intel.com/content/www/us/en/ark/products/95597/intel-celeron-processor-j3355-2m-cache-up-to-2-5-ghz.html)** (AES-NI enabled), two **SO-DIMM DDR3L** memory slots, a **PCIe 2.0x16** expansion slot, and 2x SATA III ports.  On top of that, this is a ***mini*-ITX mobo**, so we can put it inside a low-profile case.  At the very least, if you're looking for a more recent mobo, my suggestion is to use the ASRock J3355b-itx as reference.

[![ASRock J3355b-itx](/assets/posts/2020-06-25-Pfsense-white-box/asrock-mobo.jpg){: .PostImage}](/assets/posts/2020-06-25-Pfsense-white-box/asrock-mobo.jpg)

## Network interface card
Even though the ASRock mini-itx comes with a built-in NIC, it's a single port Realtek NIC that pfSense will likely not recognize out of the box.  Ideally, you'd buy and install an Intel NIC using the PCI-e expansion slot on the ASRock mobo.  The card choice depends on your needs but at the very least, consider a **Gigabit Intel NIC with two ports**, one for WAN and another for LAN.  (It's possible to use a single port with a VLAN-capable switch but this is a far more complex setup.)  Also, make sure the Intel NIC comes with a **low-profile support** or it won't fit many min-ITX cases.

[![Intel NIC with 2 ports](/assets/posts/2020-06-25-Pfsense-white-box/intel-nic.jpg){: .PostImage}](/assets/posts/2020-06-25-Pfsense-white-box/intel-nic.jpg)

## Storage
pfSense uses very little storage space, so you don't need TBs of storage here.  The things that matter are read/write speed and reliability.  A simple **120GB solid-state drive (SSD)** will be more than enough, for example. 

[![SSD](/assets/posts/2020-06-25-Pfsense-white-box/120gb-ssd.jpg){: .PostImage}](/assets/posts/2020-06-25-Pfsense-white-box/120gb-ssd.jpg)

## Case
You can use *any* mini-ITX case *as long as* it has support for at least **one expansion slot**.  Also, note that some mini-itx cases have an expansion slot parallel to the mobo (it sits above the mobo's i/o plate), instead of perpendicular (next to the mobo's i/o plate).  That will usually be okay but you'll need to buy compatible PCI-e extension cable then.

[![Case](/assets/posts/2020-06-25-Pfsense-white-box/case-antec-front.jpg){: .PostImage}](/assets/posts/2020-06-25-Pfsense-white-box/case-antec-front.jpg)

[![Case](/assets/posts/2020-06-25-Pfsense-white-box/case-antec-back.jpg){: .PostImage}](/assets/posts/2020-06-25-Pfsense-white-box/case-antec-back.jpg)

[![Case](/assets/posts/2020-06-25-Pfsense-white-box/case-kmex-front.jpg){: .PostImage}](/assets/posts/2020-06-25-Pfsense-white-box/case-kmex-front.jpg)

[![Case](/assets/posts/2020-06-25-Pfsense-white-box/case-kmex-back.jpg){: .PostImage}](/assets/posts/2020-06-25-Pfsense-white-box/case-kmex-back.jpg)

## Power supply unit
If you bought an expensive case with a built-in PSU, go ahead and use it.  However, if you bought a cheap case with a PSU or one that even doesn't have a built-in PSU, then buy and use a [**pico PSU**](https://www.amazon.com/s?k=pico+PSU). My experience is that **cheap PSUs come with bad fans**, so avoid them if your firewall will be next to anyone in the house because it will eventually start making a lot of noise.  A pico PSU is the way to go, and because this box uses very little power, buy a low power PSU (anything between 60W-180W should be okay, for reference).

[![PicoPSU](/assets/posts/2020-06-25-Pfsense-white-box/pico-psu.jpg){: .PostImage}](/assets/posts/2020-06-25-Pfsense-white-box/pico-psu.jpg)

[top](#){: .btn .btn--light-outline .btn--small}

# Software
Software-wise, there's not much to say other than

1. Download the ISO from the [official website](https://www.pfsense.org/download/);
2. Check the SHA signature of the downloaded file against the one from the official website;
3. Flash the image onto a USB stick;
4. Connect a monitor and keyboard to your pfSense box;
5. Insert the USB stick into your pfSense box, turn it on, and follow the instructions.

Now, if you want a more in-depth look into installation and initial configuration, check [**the official docummentation**](https://docs.netgate.com/pfsense/en/latest/install/installing-pfsense.html).  Alternatively, there's a very easy to follow video tutorial that [Lawrence Systems](https://www.youtube.com/channel/UCHkYOD-3fZbuGhwsADBd9ZQ) put together. (It's a bit old but still valid.)

{% include video id="9kSZ1oM-4ZM" provider="youtube" %}

[top](#){: .btn .btn--light-outline .btn--small}

# Conclusion
This concludes the basics of a cheap, low-profile, and low-power (yet powerful) pfSense white-box.  I've never had any issues with this hardware and recommend it to anyone interested in diving into firewalls or just looking for something a bit more advanced than what commercial routers can offer.  If you want to try something different than pfSense, take a look at [OPNsense](https://opnsense.org/)--a pfSense fork--for another free and open-source firewall software.

[top](#){: .btn .btn--light-outline .btn--small}

---
title: "Mesh networking: A guide to using free and open-source software with common hardware"
date: 2020-12-07 12:10:00 -0300
tags: mesh adhoc ieee wifi wireless radio network router openwrt batman
header:
  overlay_image: "/assets/posts/2020-11-24-mesh-networking-openwrt-batman/header.jpg"
  overlay_filter: "0.85"
toc: true
toc_label: "Table of Contents"
toc_icon: "list"
---
# Changelog
**October XX, 2021**: Major updates to make the guide compatible with OpenWrt 21.02.
{: .notice--success }
**September 16th, 2021**: Updated the information about OpenWrt 21 in the section [**Bonus content: Moving from OpenWrt 19 to 21**](#bonus-content-moving-from-openwrt-19-to-21).  In brief, DSA support is still very limited and OpenWrt has officially started rolling out version 21 with the [release of OpenWrt 21.02](https://openwrt.org/releases/21.02/notes-21.02.0). I'm currently testing the new version and network configuration on a few devices and once I get everything running as well as it was in version 19, I will update the entire article to reflect the new (and current) configuration.  It is, of course, still possible to download and use [the latest OpenWrt 19 images](https://downloads.openwrt.org/releases/19.07.8/targets/), which should be just fine for a long time still.  However, if you want to make use of OpenWrt 21, then read the aforementioned bonus section for guidance on the syntax changes and updated hardware requirements.
{: .notice--info }
**July 6th, 2021**: Added information about transitioning from OpenWrt 19 (current stable release) to OpenWrt 21 (next stable release) to a new section called [**Bonus content: Moving from OpenWrt 19 to 21**](#bonus-content-moving-from-openwrt-19-to-21).  In brief, the *next* stable release includes changes to the network configuration syntax that are incompatible with this guide.  Once the release version 21 becomes the *current* stable, however, I will update the main guide to reflect those changes.  In the meantime, I added a few references to the OpenWrt forum that should help anyone interested in using version 21 instead of 19.  Thanks to [Steve](https://forum.openwrt.org/u/SteveNewcomb) for testing and sharing his `batman-adv` configuration running on OpenWrt 21.
{: .notice--info }
**Feb 17th, 2021**: Per a reader's suggestion (Joshua), I added a [`vi` cheat table](#vi-cheat-table) that has a summary of the main commands, and in the [Mesh node basic config](#mesh-node-basic-config) section, I included additional instructions on how to copy and paste the configuration files from one mesh node to another using `scp`.  (Alternatively, it's also possible to do so using Luci's backup/restore option.)
{: .notice--info }
**Jan 9th, 2021**, Update #2: Added instructions on how to automatically upgrade all installed packages with a single command.  This information is in [Updating and installing packages](#updating-and-installing-packages).
{: .notice--info }
**Jan 9th, 2021**, Update #1: Added a new section about [hardware-specific configurations](#hardware-specific-configurations) that are sometimes required for enabling the `mesh point` mode of operation.
{: .notice--info }
**Dec 7th, 2020**: Publication of the original guide
{: .notice--info }

[top](#){: .btn .btn--light-outline .btn--small}

# Introduction
In this tutorial, we will learn how to create [**mesh networks**](https://en.wikipedia.org/wiki/Wireless_mesh_network) ([**IEEE 802.11s**](https://en.wikipedia.org/wiki/IEEE_802.11s)) using [**OpenWrt**](https://openwrt.org/) and the [layer-2](https://en.wikipedia.org/wiki/Data_link_layer) implementation of the *Better Approach to Mobile Adhoc Networking*, called [**batman-adv**](https://www.kernel.org/doc/html/v4.15/networking/batman-adv.html).  All the software mentioned here is **free** and **open-source**, as opposed to commercial alternatives ([UniFi Mesh](https://unifi-mesh.ui.com) or [Google's Nest Wifi](https://store.google.com/us/product/nest_wifi)).

This is not meant to be an exhaustive presentation of any of the covered topics. If you have suggestions on how to improve this guide, feel free to [get in touch with me](/contact/). I'm always eager to learn new things and share them. Also, I plan on updating this article every once in a while to best reflect my knowledge about the topics covered here and to add information provided by the readers. Check the [changelog](#changelog) for updates.

[top](#){: .btn .btn--light-outline .btn--small}

# Why am I writing this guide?
Even though the concept of mesh networking has been around for quite some time now, the documentation of its implementation is still scarce/nichey, proprietary, or outdated.  I don't feel qualified to speculate on why this is so but I find it odd because many of the radio devices found in popular wireless routers actually support mesh networking--but the original firmware rarely supports it.

My intention with this tutorial is to help closing the gap between concept and implementation of mesh networking using up-to-date software that anyone can download and install on cheap, commonly available hardware--primarily consumer wireless routers (from old to new, single- or multi-band) but the principles should be extendable to any cellphones, laptops, PCs or servers running **Linux**.  The content is partially based on my own experience and builds upon the work of other, much more talented individuals who shared their knowledge on the Web.  More specifically, the content is notably influenced by the following:

* Brian Innes workshop about using Raspberry Pis to create a mesh network for sharing sensor data wirelessly ([Github repo](https://github.com/binnes/WiFiMeshRaspberryPi))
* Andreas Spiess [LoRa mesh project](https://www.youtube.com/watch?v=TY6m6fS8bxU)
* Maintaners of the [OpenWRT documentation](https://openwrt.org/docs/start) and the [B.A.T.M.A.N. wiki](https://www.open-mesh.org/projects/batman-adv/wiki)
* Multiple users from the OpenWrt forum who shared their opinions over the years. To name a few,  the users [jeff](https://forum.openwrt.org/u/jeff), [mcarni](https://forum.openwrt.org/u/mcarni), [oavaldezi](https://forum.openwrt.org/u/oavaldezi), [slh](https://forum.openwrt.org/u/slh), and many others. Thanks for keeping the posts public.

[top](#){: .btn .btn--light-outline .btn--small}

# Objectives
1. Get familiar with `/etc/config/` files in OpenWrt devices (namely, `wireless`, `network`, `dhcp`, `firewall`) to quickly and permanently configure mesh nodes. 
2. Edit files directly from the terminal using the default text editor `vi`.
3. Configure OpenWrt devices to play one of three possible roles in the network: (a) mesh node, (b) mesh + bridge node, or (c) mesh + gateway node.
4. Install and configure the Kernel module `batman-adv` on an OpenWrt device using the `opkg` package manager.
5. Use `batctl` to test, debug, and monitor connectivity within the mesh.
6. Use two radios to segment mesh (5Ghz) from non-mesh (2.4Ghz) wireless communication.
7. Add encryption to the mesh network with the package `wpad-mesh-openssl`.
8. Use VLANs to create `default`, `iot`, and `guest` networks within the mesh using `batman-adv`.

[top](#){: .btn .btn--light-outline .btn--small}

# Outline
From this point forward, the article is divided into four main parts: 
1. [Concepts and documentation](#concepts-and-documentation): *Optional for advanced users.* Brief introduction to just enough network concepts to allow the implementation of simple mesh networks. When appropriate, a link to the relevant OpenWrt documentation was also provided.  
2. [Hardware](#hardware): *Optional for everyone*. A few notes about the hardware used in the examples and recommendations for those who are planning on buying new/used devices for their mesh project.
3. [Software](#software): *Optional for everyone*. A few notes about the software used in the examples.
4. [Implementation](#implementation): *Required*. Step-by-step procedure to configure mesh nodes, bridges, and gateways.  It goes from flashing OpenWrt to configuring VLANs with `batman-adv`. You probably came here for this part.

[top](#){: .btn .btn--light-outline .btn--small}

# Concepts and documentation

## Main network definitions
* Mesh [node](https://en.wikipedia.org/wiki/Node_(networking)): Any network device that is connected to the mesh network and that helps routing data to (and from) mesh clients.  Here, however, if a mesh node acts as a bridge or gateway, it will always be referred by the latter role, even though by definition, mesh bridges and mesh gateways are also mesh nodes.  
  In addition, even though it's possible to route mesh traffic via cable, in this tutorial, *all mesh nodes are also wireless devices*, meaning that they have access to a radio with [**mesh point** (802.11s)](https://en.wikipedia.org/wiki/IEEE_802.11s) capabilities.
  * [Learn about the OpenWrt wireless config **/etc/config/wireless**](https://openwrt.org/docs/guide-user/network/wifi/basic)

* [Bridge](https://en.wikipedia.org/wiki/Bridging_(networking)): A network device that joins any two or more network interfaces (e.g., LAN Ethernet and wireless) into a single network.  Here, when a device is referred to as a bridge, it means that in addition to being a mesh node, the only other thing it does is bridge interfaces.  But of course, a gateway *device*, such as a router with a built-in modem, or a firewall appliance, may also work as a bridge for multiple interfaces. The distinction in the examples is just used to highlight its main role in the network.  Therefore, a mesh bridge in this tutorial is a mesh node that simply bridges the mesh network with a WiFi access point  for non-mesh clients, for example, or its LAN ports.

* [Gateway](https://en.wikipedia.org/wiki/Gateway_(telecommunications)): A network device that translates traffic from one network (LAN) to another (WAN) and here, acts as both a **firewall** and **DHCP server**.  (If there's more than one DHCP server in the same network, they assign IPs to different ranges, such as `.1-100`, `.101-200`, and so on.)
  * [Learn about the OpenWrt network config **/etc/config/network**](https://openwrt.org/docs/guide-user/base-system/basic-networking)

* [DNS](https://en.wikipedia.org/wiki/Domain_Name_System): In brief, a system for translating domain names (e.g., `cgomesu.com`) into IP addresses (`185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`). DNS filtering systems, such as [PiHole](https://pi-hole.net/), work by catching such requests--usually sent through port `53`--and checking if the domain is blacklisted or not.  In this tutorial, we will always use an external DNS server, such as `1.1.1.1` (Cloudflare) or `8.8.8.8` (Google), but if you have your own DNS resolver, feel free to use it instead when configuring your mesh network but then make sure the mesh network/VLAN has access to its address.

* [DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol): An IP management system that dynamically assigns layer-3 addresses for devices connected to a network. For instance, it might dynamically assign IPs between `192.168.1.0` and `192.168.1.255` (i.e., `192.168.1.0/24`) to any devices connected to LAN. Of note, because this is a network layer protocol, it uses IP addresses, whereas `batman-adv` uses MAC addresses because it works at the data link layer (and therefore, `batman-adv` actually doesn't need DHCP and IPs to discover and manage mesh clients but we're going to use them to make it more intuitive and easier to integrate mesh with non-mesh clients).
  *  [Learn about the OpenWrt DNS and DHCP config **/etc/config/dhcp**](https://openwrt.org/docs/guide-user/base-system/dhcp)

* [Firewall](https://en.wikipedia.org/wiki/Firewall_(computing)): A network system that monitors and controls network traffic, such as specifying rules for incoming WAN traffic (e.g., `deny all`), outgoing LAN traffic (`accept all`), geoblocking and IP filtering systems, intrusion prevention/detection systems ([Suricata](https://suricata-ids.org/)), and so on.  [OpenSense](https://opnsense.org/) and [pfSense](https://www.pfsense.org/) are examples of dedicated firewall software. If a mesh node is acting as a mesh gateway, it's imperative to configure the firewall or your mesh network will likely end up without access to external networks (e.g., WAN) and their services (e.g., DNS servers).
  * [Learn about the OpenWrt firewall config **/etc/config/firewall**](https://openwrt.org/docs/guide-user/firewall/firewall_configuration)

* [VLAN](https://en.wikipedia.org/wiki/Virtual_LAN): A *virtual* LAN that is partitioned and isolated in a network at the layer-2 level.  They are often followed by an integer to differentiate each other (e.g., VLAN 1, VLAN 50) and used to better manage network clients that belong to different groups (e.g., administrators, IoT devices, security cameras, guests).

## Network topologies

{% include video id="zbqrNg4C98U" provider="youtube" %}
{:. text-center}

## Mesh networks

### What are mesh networks?

{% include video id="tYLU755T6_I" provider="youtube" %}
{:. text-center}

### Where can I learn more about mesh networking?
* Wikipedia articles about [mesh networking](https://en.wikipedia.org/wiki/Mesh_networking) and [wireless mesh networks](https://en.wikipedia.org/wiki/Wireless_mesh_network)
* [Peer-reviewed papers or books](https://scholar.google.com/scholar?q=mesh+networking)

### Routing protocols
There are [dozens of algorithms](https://en.wikipedia.org/wiki/Wireless_mesh_network#Protocols) for routing packets in a mesh network.  A few notable ones are the Optimized Link State Routing (OLSR) and the Hybrid Wireless Mesh Protocol (HWMP). 

In this tutorial, however, we will cover only one of them, called [*Better Approach to Mobile Adhoc Networking*](https://en.wikipedia.org/wiki/B.A.T.M.A.N.) (**B.A.T.M.A.N.**), because [it has long been incorporated into the Linux Kernel](https://www.kernel.org/doc/html/latest/networking/batman-adv.html) and is thus easily enabled on Linux devices.  It is also a [fairly well-documented](https://www.open-mesh.org/projects/batman-adv/wiki) algorithm that [has been continuously improved](https://www.open-mesh.org/projects/open-mesh/activity) over the years.  Another noteworthy feature of `batman-adv` is its lack of reliance on layer-3 protocols for managing mesh clients because it works at the layer-2 and its ability to create VLANs.  Think of it as if it were a big, smart, virtual switch, in which its VLANs are port-based segmentations.  If you want an interface to use a particular mesh VLAN, just "plug it" into the approriate port of the `batX` switch (e.g., bridge `if-guest` and `bat0.2` to give the guest network access to the `bat0` VLAN 2).

#### batman-adv
As mentioned before, B.A.T.M.A.N. has gone through multiple changes over the years, which means that there are actually *multiple versions of the algorithm*. I've had a good experience with [**B.A.T.M.A.N. IV**](https://www.open-mesh.org/projects/batman-adv/wiki/BATMAN_IV) and therefore, the examples here make use of it.  However, you are free to try whatever version you want and even run them in parallel to each other, by assigning a different `batX` interface to each version of the algorithm (versions are chosen with `option routing_algo` in the `/etc/config/network` config file for each enabled `batX` interface).

Config-wise, there's very little to do because the default settings should work very well in most environments.  One exception is when you have multiple gateways in the network to provide high availability, for example, and you might want to let each mesh node know about them and their speeds to better route the mesh traffic.  This requires setting `option gw_mode` to `server` or `client`, for example.  Many other tweaks that are not covered here are [described in their wiki](https://www.open-mesh.org/projects/batman-adv/wiki/Doc-overview#Protocol-Documentation).

#### batctl
Another very cool feature of B.A.T.M.A.N. is the ability to test, debug, monitor, and set settings with the package [`batctl`](https://downloads.open-mesh.org/batman/manpages/batctl.8.html).  A few noteworthy options:

* Ping mesh node/client with its MAC address `f0:f0:00:00:00:00`
```
batctl p f0:f0:00:00:00:00
```
* [`tcpdump`](https://linux.die.net/man/8/tcpdump) for all mesh traffic in the `bat0` interface
```
batctl td bat0
```
* Prints useful stats for all mesh traffic, such as sent and received bytes
```
batctl s
```
* Shows the neighboring mesh nodes
```
batctl n
```
* Displays the gateway servers (`option gw_mode 'server'`) in the mesh network
```
batctl gwl
```

It goes without saying that if you want to dive deep into `batman-adv`, you should take a good look at `batctl`, too.

[top](#){: .btn .btn--light-outline .btn--small}

# Hardware
Unless otherwise specified, all mesh nodes used in the various implementations had the following hardware:

* **Device**: [TP-Link TL-WDR4300](https://www.tp-link.com/us/home-networking/wifi-router/tl-wr1043nd/) v1.0 - v1.7
  * **SoC**: Atheros AR9344
  * **WLAN Hardware**: Dual-band (Atheros AR9344, Atheros AR9580)
  * **CPU**: `560 Mhz`
  * **Flash memory**: `8 MB`
  * **RAM**: `128 MB`

[![TL-WDR4300 front](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/tplink-tl-wdr4300-front.jpg){:.PostImage}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/tplink-tl-wdr4300-front.jpg)

[![TL-WDR4300 back](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/tplink-tl-wdr4300-back.jpg){:.PostImage}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/tplink-tl-wdr4300-back.jpg)

This is a low-end, Atheros-based *dual-band* router that satisfies the [minimum hardware requirements imposed by OpenWrt 21](https://openwrt.org/releases/21.02/notes-21.02.0#increased_minimum_hardware_requirements8_mb_flash_64_mb_ram)--namely, at least `8MB` of flash memory and `64MB` of RAM.  However, the general ideas presented here should apply to **any wireless device** that meets the following criteria:

1. Compatible with the latest OpenWRT version. Refer to their [**Hardware List**](https://openwrt.org/toh/start);
2. Has access to a radio that supports the **mesh point** (**802.11s**) mode of operation. If you already have OpenWrt installed on a wireless device, you can type `iw list` and search for `mesh point` under **Supported interface modes**, or simply check if the following command outputs `* mesh point` below the name of a detected radio (e.g., `phy0`, `phy1`):

	```
iw list | grep -ix "^wiphy.*\|^.*mesh point$"
```

	If it does, then the associated radio can be configured as a mesh point.

Now, if you're looking for devices to buy and experiment on, my suggestion is to look for high-end dual-band wireless routers to allow a better segmentation of the wireless networks.  If you can afford spending more for a mesh node, look for tri-band devices.  Netgear and Linksys have solid options that are compatible with OpenWrt. For example, the Linksys WRT1900AC (v1/v2) dual-band wireless router would make for a good mesh node:

[![Linksys WRT1900AC](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/linksys-wrt1900ac.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/linksys-wrt1900ac.jpg) 

For single-board computer (SBC) fans like me, you can run OpenWrt with most of them and then use a combination of on-board wireless and USB adapter to create a powerful mesh node. [ClearFog boards](https://shop.solid-run.com/product-category/embedded-computers/marvell-family/clearfog-base-pro/) with one or two mini PCIe wireless cards would make very good candidates for such a project, for example:

[![ClearFog Pro](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/clearfog-pro.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/clearfog-pro.jpg) 

Of course, you can install OpenWrt on bare metal x86-64 machines (e.g., standard PC or server running Intel/AMD), which will give you lots of options to put together an impressive mesh device. However, if you just want your work/home laptops/PCs to *be part of the mesh* (i.e., become a mesh node), there are better alternatives than installing OpenWrt as its OS.  For example, you can run OpenWrt with a [virtual machine](https://openwrt.org/docs/guide-user/virtualization/start) or as a [docker container](https://github.com/openwrt/docker).  Naturally, it's also possible to configure `batman-adv` on Linux distributions other than OpenWrt, such as Arch, Debian, and Ubuntu.  See [Getting started with `batman-adv` on any Linux device](#getting-started-with-batman-adv-on-any-linux-device).

As mentioned before, even if the existing/on-board radio of your SBC/laptop/PC/server does not support the mesh point mode of operation, you can always buy a compatible PCIe card or USB adapter to turn your device into a mesh node and then use the other radio for another purpose.  For example, many [Alfa Network](https://www.alfa.com.tw/) adapters can operate in mesh point mode, like the cheap AWUS036NH: 

[![Alfa AWUS036NH](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/AWUS036NH.jpg){:.PostImage}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/AWUS036NH.jpg) 

All that said, most home users will be just fine with a cheapo, used, old, and single-band router.  For a brand reference, TP-Link has good and affordable devices that can be used in a mesh networking project without issues.  If you're new to this, start from here (small, simple) and think about efficiency over power.  You don't need to drive a Lamborghini to get a snack at the grocery store. For additional resources, skip to the section called [Useful hardware resources](#useful-hardware-resources) down below.

## Hardware-specific configurations
Every once in a while, I run into hardware that is capable of operating in `mesh point` mode but the default OpenWrt firmware uses a module for the wireless adapter that is loaded with incompatible parameters.  Here is a list of a few of the known ones and their solution.

### ath9k modules
If your device uses the `ath9k` module, there's a chance that you'll need to enable the `nohwcrypt` parameter of the module to use the mesh *with encryption*.  First, however, try without changing the default module parameters.  After rulling out possible typos in the network and wireless configuration files, try the following:
1. Edit the `/etc/modules.d/ath9k` file and add `nohwcrypt=1` to it.  If there's something in the file, use a whitespace to separate parameters.
2. Save the file, and **reboot** your device. 
3. Once the device comes back, check if `nohwcrypt` is now enabled by typing 
   ```
   cat /sys/module/ath9k/parameters/nohwcrypt
   ```
   If `nohwcrypt` is enabled, the output will be `1`; otherwise, it will be `0`.
4. Check your mesh configuration once again and add encryption to your wireless mesh stanza.

* Known affected devices:

  | brand | model | version | OpenWrt release |
  |:---:|:---:|:---:|:---:|
  | TP-Link | WR-1043-ND | 1.8 | 19.07 |

### ath10k modules
I've noticed that radio devices that use the `ath10k` module and more specifically, the ones using `ath10k-firmware-qca988x-ct`, are not able to operate in `mesh point` mode by default.  If you check the syslog, you'll notice that there will be a few messages stating that the `ath10k` module must be loaded with `rawmode=1` to allow mesh.  However, I've tried that before without much success.  Instead, my current recommendation to get `mesh point` working with the **QCA988x** is the following (**Internet connection required** to download packages via `opkg`):
1. Remove the **Candela Tech** (`*-ct`) modules as follows:
   ```
   opkg remove ath10k-firmware-qca988x-ct kmod-ath10k-ct
   ```
2. Install the non-ct modules:
   ```
   opkg update && opkg install ath10k-firmware-qca988x kmod-ath10k
   ```
3. Reboot your device and then check the status of your mesh network.

* Known affected devices:

  | brand | model | version | OpenWrt release |
  |:---:|:---:|:---:|:---:|
  | TP-Link | Archer C7 | 2.0 | 19.07 |
  | TP-Link | Archer C7 US | 2.0 | 19.07 |
  | TP-Link | Archer C7 | 4.0 | 19.07 |
  | TP-Link | Archer C7 | 5.0 | 19.07 |

## Useful hardware resources
These are a few resources that I've used in the past that you might find useful when looking for mesh compatible devices:

- **OpenWrt**:
  - [Buyers' Guide](https://openwrt.org/toh/buyerguide)
  - [Extended Table of Hardware](https://openwrt.org/toh/views/toh_extended_all)
- **Linux Wireless wiki**:
  - The wiki has a list of [existing Linux wireless *drivers*](https://wireless.wiki.kernel.org/en/users/drivers) that mention whether it supports `mesh point` or not.  To search for specific *devices*, my recommendation is to (a) open the page of a driver that supports mesh (e.g., [ath10k](https://wireless.wiki.kernel.org/en/users/drivers/ath10k)), then (b) look for [supported devices](https://wireless.wiki.kernel.org/en/users/drivers/ath10k#supported_devices) under it, and finally, (c) go to OpenWrt's [Extended Table of Hardware](https://openwrt.org/toh/views/toh_extended_all) and in **WLAN Hardware**, enter a supported device found on the Linux Wireless wiki (e.g., `QCA9880`). This will [create a filter](https://openwrt.org/toh/views/toh_extended_all?dataflt%5BWLAN+Hardware*%7E%5D=QCA9880) to show only devices that contain the given hardware and should provide you a starting point for further research.  


[top](#){: .btn .btn--light-outline .btn--small}

# Software
Unless otherwise specified, all mesh nodes were running the following software:

[![OpenWrt default SSH welcome](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/openwrt-ssh-welcome.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/openwrt-ssh-welcome.jpg) 

* **Operating System**:
	* **Firmware**: OpenWrt `21.02.0`, `r16279-5cc0535800`
	* **Linux kernel**: `5.4.143`

* **Packages mentioned in the tutorial**:
	* [`batctl-full`](https://openwrt.org/packages/pkgdata/batctl-default): 2021.1-1
	* [`kmod-batman-adv`](https://openwrt.org/packages/pkgdata/kmod-batman-adv): 5.4.143+2021.1-4
	* [`wpad-mesh-wolfssl`](https://openwrt.org/packages/pkgdata/wpad-mesh-wolfssl): 2020-06-08-5a8b3662-35

To find out the version of all installed packages, type 

```
opkg list-installed
```

or if you prefer to filter the output, use grep.  For example, the following will show the version of all installed packages containing `bat` (e.g., `batctl`, `kmod-batman-adv`):

```
opkg list-installed | grep bat
```

Huge differences in firmware, kernel, or package versions *might* make the implementation of a mesh network a little bit different than the way it was explained here.  Of note, devices running the `batman-adv` **version 2019.0-2 and older** are certainly incompatible with the instructions found in this tutorial, the reason being that the module was modified after then to better integrate with the [network interface daemon](https://openwrt.org/docs/techref/netifd).  Fortunately, the implementation using old modules is just a simple as with the latest one. [Check what the B.A.T.M.A.N. wiki has to say about it](https://www.open-mesh.org/projects/batman-adv/wiki/Batman-adv-openwrt-config#Batman-adv-20190-2-and-older).  However, it's worth mentioning that with old batman modules, changes to `/etc/config/network` will likely require a reboot instead of simply reloading `/etc/init.d/network`.

Also, I've noticed that when installing `kmod-batman-adv`, the package manager will install a minimal version of `batctl`, called `batctl-tiny`, that lacks some of the options mentioned here (e.g., `batctl n` and `batnctl o`).  However, if you install `batctl` first and then `kmod-batman-adv`, the package manager will preserve `batctl-default`, which has most of the `batctl` features.  In this tutorial, however, we will use the `batctl-full` package that contains all features referred to in the [`batctl` manual](https://downloads.open-mesh.org/batman/manpages/batctl.8.html).

Finally, the installation of `wpad-mesh-wolfssl` will conflict with the already installed `wpad-basic-wolfssl` package (or any other `wpad-basic*` package, for that matter).  This means **you have to remove the latter before installing the former**.  To remove the `wpad-basic-wolfssl` or any other conflicting `wpad-basic` package, simply type

```
opkg remove wpad-basic*
```

## VI text editor
The default text editor in a standard OpenWrt image is [**vi**](https://en.wikipedia.org/wiki/Vi), which is an old, screen oriented editor that most modern users will find counterintuitive to use.  Fortunately, once you get the hang of it, `vi` becomes very easy to use and it becomes a very convenient way of editing config files.  Here's all that you need to know about using `vi` in a terminal:

You can open a file by adding the filename as an argument to `vi`, as follows

```
vi /etc/config/network
```

and if the file does not exist, `vi` will create one with that name.  

By default, `vi` will start in **command mode**.  Such a mode let's you navigate the file with the arrow keys and use the *delete button* to delete characters.  (Also, in command mode, you can type `dd` to delete entire lines, which is very useful if you need to delete lots of things quickly.)  

However, if you need to type characters and have more flexibility to edit the file, you need to tell `vi` to enter **insert mode**.  To enter insert mode, type (no need to hit return/enter afterwards)

```
i
```

and at the bottom of the screen, you will see that it now shows a `I` to indicate that `vi` is in insert mode.  You can now type freely and even paste multiple things at once in insert mode. 

When you're done, press the button **Esc** to go back into command mode.  Notice that at the bottom of the screen, now there's a `-` where the `I` was, which tells you you're in command mode once again.

In command mode, you can then **write changes to the file** by typing (followed by return/enter)

```
:w
```

Now you've saved the file. To quit, type

```
:q
```

Alternatively, you can *write and quit* by simply typing `:wq`.

`vi` has other commands as well but honestly, that's pretty much all that you need to know about `vi` in order to use in the examples covered here.  Give it a try!  

### VI cheat table

| mode | key/command | action | 
|:---:|:---:|:---:|
| command | `i` key | Enter *insert* mode |
| insert | `Esc` key | Return to *command* mode |
| command | `dd` (or hold `d` key) | Erase entire row |
| command | `:w` + Enter/Return | Write to file |
| command | `:q` + Enter/Return | Quit to terminal |
| command | `:q!` + Enter/Return | Quit without saving changes |
| command | `:wq` + Enter/Return | Write to file and quit |

### Alternatives to VI
Now, if you still don't like to use `vi`, you can always transfer files from your laptop/PC to OpenWrt via sftp, for example, or utilities like [`scp`](https://en.wikipedia.org/wiki/Secure_copy_protocol).

[top](#){: .btn .btn--light-outline .btn--small}

# Implementation
In this section, we will see how to configure **four mesh nodes** in **three different network topologies**. More specifically: 

* **Gateway-Bridge**: A mesh network in which one node plays the role of a mesh gateway and another, of a bridge, while the remaining are just mesh nodes.  This is a very typical scenario for a home or small office, for example.

[![Topology - Gateway-Bridge](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-gateway-bridge.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-gateway-bridge.jpg)

* **Bridge-Bridge**: Two nodes play the role of a bridge, therefore making the mesh network transparent to the external (non-mesh) networks.

[![Topology - Bridge-Bridge](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-bridge-bridge.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-bridge-bridge.jpg)

* **Gateway-Gateway**: Two nodes play the role of a gateway to provide high-availability to mesh clients/nodes.

[![Topology - Gateway-Gateway](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-gateway-gateway.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-gateway-gateway.jpg)

In all such cases, we will use the **5Ghz** radio exclusively for *mesh* wireless traffic, while the more widely compatible **2.4Ghz** radio, as well as Ethernet connections, will be left available for *non-mesh* wireless traffic:

[![Segmentation](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/segmentation.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/segmentation.jpg)

Segmentation of mesh vs. non-mesh traffic **is not a requirement** but an option that greatly improves performance. If your mesh devices do not support dual-band, simply assign the same radio for both mesh and non-mesh wireless interfaces.
{:. .notice--info }

First, however, we will start with the aspects that are common to all topologies, such as planning the mesh network, and the installation and basic configuration of OpenWrt mesh nodes.  Then, we will move to the specifics of each of the aforementioned mesh network topologies.  Finally, we end the section with a slightly more complex scenario to illustrate how to create **mesh VLANs** with `batman-adv` and a very brief introduction to using `batman-adv` on other Linux distros.

[![Topology - Mesh VLANs](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-mesh-vlans.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-mesh-vlans.jpg)

Even though the examples show static nodes, **none of the mesh nodes need to be static**. The mesh network and its components can be partially or totally mobile. For example, if some of your nodes are mobile units (e.g., vehicles, drones, robots, cellphones, laptops), they can leave and join the mesh, recreate the mesh elsewhere, join a completely different mesh, and so on.  The routing algorithm (`batman-adv`) will automatically (and  seamlessly) take care of changes to the network topology. (But of course, if there's a single gateway and it does not reach any node, the network is bound to stop working as intended without proper configuration to handle such scenarios.)

[![Topology - Moving nodes](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-moving-nodes.gif){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-moving-nodes.gif) 

## Planning
Just like any other type of network, deploying a mesh network--especially over large areas, with dozens of nodes--requires a fair deal of planning; Otherwise, you are bound to experience, for instance, bottlenecks, uneven access point signal quality, and unstable WAN connectivity across the mesh.  Also, features like **high availability** go well beyond the configuration and topology of a mesh network (e.g., power source, whre your WAN connections are coming from, and the hardware you are using all play important roles when it comes to high availability).  Mesh networks are very, very easy to scale but planning is key.  

*Eli the Computer Guy* has an old video about mesh networks that goes into things like high availability and bottlenecks.  If that matters to you, take a look. The relevant content **starts at `03:30`** and **ends at `17:30`**, approximately.

{% include video id="T7fJwAyALss" provider="youtube" %}
{:. text-center}

The examples in this tutorial are simple *by design*--they were created to illustrate different scenarios in a way that makes it easy to understand what is going on. The idea is to use the examples as templates for more complex implementations.

## OpenWrt installation and initial configuration
Now that you have the hardware, the first thing to do is to install OpenWrt.  Flashing a default OpenWrt image onto a **compatible device** is a very **easy and safe procedure** because it's been tested multiple times.  (For extra safety precautions, you might want to search the web for `your-device` + `openwrt` to see if there's any indexed forum post or comment regarding installation issues and bugs, for example.)  

If you're **new to all this**, the folks at OpenWrt were kind enough to provide a plethora of instructions on [how to install and uninstall OpenWrt](https://openwrt.org/docs/guide-user/installation/start) and even put together an [**installation checklist**](https://openwrt.org/docs/guide-user/installation/generic.flashing#installation_checklist).  At the very least, do the following:

1. Look for your device's **model and version** in the [**Table of Hardware**](https://openwrt.org/toh/start) and open its **Device Page** (e.g., [TP-Link TL-WDR4300](https://openwrt.org/toh/tp-link/tl-wdr4300_v1));
2. Double check that the **model and version** match your device's **model and version** in the **Supported Versions** table;
3. In the **Installation** table, you will find a column called *Firmware OpenWrt Install URL* and another one called *Firmware OpenWrt Upgrade URL*. If your device is **still running the original firmware**, then download the binary from the *Firmware OpenWrt **Install** URL* column; otherwise, download the binary from the *Firmware OpenWrt **Upgrade** URL* column.  Both files should have a `.bin` extension;
4. Regardless of the binary file downloaded, [**verify its checksum**](https://openwrt.org/docs/guide-quick-start/verify_firmware_checksum) afterwards;
5. Disconnect your laptop/PC from any access point or switch, and connect your laptop/PC directly to the device's Ethernet port. 
6. Open your device's web UI, go to its Settings and/or find the **Firmware Upgrade** option. Then, select the downloaded OpenWrt binary, and let it do its thing. Once it's done, the device will reboot with OpenWrt installed. 
7. You should now be able to reach your new OpenWrt device at `192.168.1.1` if connected to a LAN port. (Remember that all wireless interfaces are disabled by default, so you can only reach it via cable.)

If you followed these steps and successfully flashed the default OpenWrt firmware onto your device, then go ahead and skip to the [Initial configuration](#initial-configuration) section.  The remaining part of this section is meant for advanced users who want to customize their image files.
{:.notice--success}

If you're an **experienced user**, you can use [OpenWrt's Image Builder](https://openwrt.org/docs/guide-user/additional-software/imagebuilder) to create a customized image that contains all the necessary packages and configuration files by default.  This can save you a lot of time by letting you skip either partially or completely the remaining configuration instructions, depending on the level of specification of the `make image` build command.  

To build a custom image file, first [install the dependencies](https://openwrt.org/docs/guide-user/additional-software/imagebuilder#prerequisites) for your Linux distribution.  Afterwards, follow these steps:

1. Find the target for your device in the device's OpenWrt page.  For instance, for the [TP-Link TL-WDR4300 v1](https://openwrt.org/toh/tp-link/tl-wdr4300_v1), the target is *ath79/generic*;
2. Navigate to the root of the available targets for the latest version of the OpenWrt 21.02 release (e.g., [`21.02.0`](https://downloads.openwrt.org/releases/21.02.0/targets/));
3. Navigate to the root of your device's target (e.g., for the TL-WDR4300, that would be [*ath79*](https://downloads.openwrt.org/releases/21.02.0/targets/ath79/) > [*generic*](https://downloads.openwrt.org/releases/21.02.0/targets/ath79/generic/));
4. Go to the **Supplementary Files** table at the bottom;
5. Download the image builder `.tar.xz` file (e.g., [openwrt-imagebuilder-21.02.0-ath79-generic.Linux-x86_64.tar.xz](https://downloads.openwrt.org/releases/21.02.0/targets/ath79/generic/openwrt-imagebuilder-21.02.0-ath79-generic.Linux-x86_64.tar.xz)) and check its hash afterwards:

    ```
    sha256sum openwrt-imagebuilder-*.tar.xz
    ```
    ```
    6354c0380a8cdb2c6a7f43449a7f6b3d04c4148478752a90f2af575ee182d2bb  openwrt-imagebuilder-21.02.0-ath79-generic.Linux-x86_64.tar.xz
    ```

6. If everything looks good, extract the image builder: 
    
    ```
    tar -xvf openwrt-imagebuilder-*.tar.xz
    ```

7. Enter the image builder directory to start using `make` and then search for your device's `PROFILE` name (e.g., `tplink_tl-wdr4300-v1`), as follows:
    
    ```
    cd openwrt-imagebuilder-*
    make info
    ```
    ```
    tplink_tl-wdr4300-v1:
      TP-Link TL-WDR4300 v1
      Packages: kmod-usb2 kmod-usb-ledtrig-usbport
      hasImageMetadata: 1
      SupportedDevices: tplink,tl-wdr4300-v1 tl-wdr4300
    ```

    For long lists, you might want to filter the output via `grep`.  For example, to show only entries that contain `wdr4300`, run `make info | grep -i wdr4300`.
    {:.notice--info}

8. Build customized images for your device's profile. (See below for a table with a list of specific packages to add and to remove if you want to enable batman mesh support by default.)  For example, to build images for the **TL-WDR4300 (v1)** *without* pppoe and IPv6 support and *with* a minimal LuCI and `batman-adv` mesh support, run the following `make image` command:
    
    ```
    make image PROFILE=tplink_tl-wdr4300-v1 \
      PACKAGES="uhttpd uhttpd-mod-ubus libiwinfo-lua luci-base luci-app-firewall luci-mod-admin-full luci-theme-bootstrap \
      -ppp -ppp-mod-pppoe \
      -ip6tables -odhcp6c -kmod-ipv6 -kmod-ip6tables -odhcpd-ipv6only \
      -wpad-basic-wolfssl wpad-mesh-wolfssl \
      batctl-full kmod-batman-adv" \
      CONFIG_IPV6=n
    ```

    Notice that the prefix `-` is meant to inform that the package should be *removed* from the default image of the chosen `PROFILE`. In addition, the inclusion of `CONFIG_IPV6=n` is optional and only used here to completely disable the IPv6 configuration in the example.  Other [build configuration variables](https://openwrt.org/docs/guide-user/additional-software/saving_space#modifying_build_configuration_variables) are also optional and can be modified by adding them to the `make image` command.  For more detailed information, run `make help`.
    {:.notice--info}

    This will prompt your system to start downloading the required packages and then start building the firmware.  **Be patient** because this operation can take several minutes.

9. Once the builder is done **without any errors**, navigate to the subdirectory `./bin/targets/<target>` that contains the built image files, in which `<target>` is the device's target.  For the TL-WDR4300, for instance, the target is `ath79/generic`, which means the built files are at `./bin/targets/ath79/generic` and you can navigate to it from the image builder root directory as follows:
    
    ```
    cd ./bin/targets/ath79/generic
    ```

    Of note, the `*.manifest` file contains a list of installed packages, which is useful if you need to double check which packages a given image contains by default.  The `profiles.json` file contains even more detailed information about the built image but it is in json format.  If you have `jq` installed, however, you can parse it via `cat profiles.json | jq .` to get a more readable version.
    {:.notice--info} 

10. Personally, I like to copy all generated files to a location outside the image builder.  To create a new location for the built images in your user's Downloads directory, do as follows (suggested structure is `openwrt-custom-images/<release>/<device>`):
    
    ```
    mkdir ~/Downloads/openwrt-custom-images \
      ~/Downloads/openwrt-custom-images/21.02 \
      ~/Downloads/openwrt-custom-images/21.02/tl-wdr4300_v1
    ```

    Then copy all generated files to the new directory outside the image builder:

    ```
    cp ./* ~/Downloads/openwrt-custom-images/21.02/tl-wdr4300_v1/
    ```

    Now you can safely go back to the root of the image builder directory and run `make clean` to delete all generated files.  This is good practice if building multiple images with different features.

11. From this part forward, the procedure is the same as outlined before for the **default installation**. 

Lastly, to save additional firmware space and RAM, [follow the OpenWrt recommendation](https://openwrt.org/docs/guide-user/additional-software/saving_space) and at the end of the package list, add the following to enable `batman-adv` and include support for mesh encryption (e.g., use `sae` to authenticate mesh nodes):

| action | package |
|:---:|:---:|
| remove mesh encryption conflict | `-wpad-basic-wolfssl` |
| add mesh encryption | `wpad-mesh-wolfssl` |
| add the full batctl | `batctl-full` |
| add batman-adv | `kmod-batman-adv` |


### Initial configuration
As mentioned before, we **will not use the web UI** in this tutorial, even if the OpenWrt image you're using has LuCI installed by default.  Instead, we will access our device and configure it using only **SSH**.  So, open a terminal and `ssh` into your OpenWrt device, as follows

```
ssh root@192.168.1.1
```

in which `192.168.1.1` is your OpenWrt device's IP address (that's usually the case after a fresh install but if it's different, use the proper IP then). Because this is the first time using the system, you'll need to set a password for the `root` user.  You can do that by typing

```
passwd
```

and following the instructions.  At this point, it's good practice to label this device (e.g., `node01`) and take note of its **MAC address**.  To find out the latter, type

```
ip link
```

and keep a record of the device's name and its MAC address--if there are multiple different addresses, take note of all of them and their interface.

(*Optional*. [Configure key-based authentication](https://openwrt.org/docs/guide-user/security/dropbear.public-key.auth) and [disable password login](https://openwrt.org/docs/guide-user/base-system/dropbear). Reboot and check that `ssh` access methods are correctly configured.)  

From this point forward, we will start editing files using `vi`.  If you've not read the [section about how to use `vi`](#vi-text-editor) yet, this is a good time to do so.
{: .notice--warning }

### Default config for the hardware
Regardless of the hardware, **before doing anything related to the mesh network**, always take your time and **study the default configuration** found in `/etc/config/`.  For reference, I usually go over the following:

* How many Ethernet ports?
* Are they labeled either LAN or WAN or there's both?
* In `/etc/config/network`, how is the router handling multiple Ethernet ports? If there's both LAN and WAN, how is the router separating LAN from WAN?
* If there is both LAN and WAN, how is the firewall handling them in `/etc/config/firewall`? (Probably two zones, LAN and WAN, with LAN->WAN accept all but WAN->LAN deny all?)
* In `/etc/config/dhcp`, how is the device handling IP addresses?  (Is there a DHCP server for LAN?)

And finally, look at the wireless settings (`/etc/config/wireless`):

* How many radio devices and their names? (e.g., `radio0`)
* Configuration-wise, what is the device using by default vs. what is it capable of? (`iw list`)
* Is the radio enabled or disabled? (Keep/add `option disabled 1` to disable it before configuration; to re-enable, simply comment this line out or set the value to `0`.)
* Are there pre-configured wireless access points being broadcast?  If yes, which `option network` is it using by default? (Likely `lan` or whatever the LAN interface is being called in `/etc/config/network`.)

For example, many wireless routers have LAN and WAN ports which are handled by a `switch` configuration with VLANs enabled to separate LAN from WAN.  Take note of it;  understand what is going on in the config files;  play with them;  then, continue.  Also, take this opportunity to go over the **Device Page** to check if there's any warnings or special configuration notes.

This understanding is instrumental to the way the device will be configured to play different roles in the mesh network and a good grasp of the device's default settings will greatly reward you later on.

### Updating and installing packages
(*Only experienced users*: If you used a default image, this is a good opportunity to remove unnecessary packages. See the OpenWrt FAQ for a reference of [safe to remove packages](https://openwrt.org/faq/which_packages_can_i_safely_remove_to_save_space), for example. If this is your first time playing with mesh, leave any unmentioned pkg alone until you get everything working as intended.)

In order to update and install packages, you need to give your device **temporary access to the Internet**.  More often than not, if you have an existing network with access to the Internet on-site, then just connect the device to a router/switch via cable.  If that doesn't work, go ahead and configure your device to act like a [**dumb access point**](https://openwrt.org/docs/guide-user/network/wifi/dumbap) first.  You can check that the device has access to the Internet by `ping`ing `google.com` or `8.8.8.8`, as follows

```
ping google.com
```

If it all looks good, it's time to **update the package list**, as follows

```
opkg update
```

*Optional*. Upgrade all installed packages. Type `opkg list-upgradable` to find which packages can be upgraded and then `opkg upgrade PKG`, in which `PKG` is the package name.  If `opkg list-upgradable` run into memory issues, try commenting out a few lines in `/etc/opkg/distfeeds.conf` and try again. Alternatively, it's possible to use the following command to automatically upgrade all packages at once, per the [opkg openwrt wiki examples](https://openwrt.org/docs/guide-user/additional-software/opkg#examples):
```
opkg list-upgradable | cut -f 1 -d ' ' | xargs opkg upgrade
```
**Be careful with mass upgrades though**, especially if you're running a device with limited memory.  You might end up even bricking your device.

Now, let's install the mesh-related packages and remove conflicting packages.  First, remove `wpad-basic-wolfssl` with

```
opkg remove wpad-basic-wolfssl
```

then install `batctl-full`, `batman-adv`, and `wpad-mesh-wolfssl` with

```
opkg install batctl kmod-batman-adv wpad-mesh-wolfssl
```

It is up to you whether to install `wpad-mesh-wolfssl` or `wpad-mesh-openssl`. For a detailed description of the main differences, take a look at the [wolfSSL documentation](https://www.wolfssl.com/docs/wolfssl-openssl/).  In brief, wolfSSL was built for embedded systems--such as most consumer routers--and it is lighter and more frequently patched than OpenSSL. OpenSSL is much older and more general purpose.
{:.notice--info}

Make sure there are no error messages and if there are, troubleshoot them before proceeding. 

Remove the connection that gave your device temporary access to the Internet.  Then, **reboot** (type `reboot` in the terminal) and restart the SSH session with your laptop/PC still connected to the device via cable.

Depending on your hardware, you might run into issues while trying to make one or multiple radios operate in `mesh point` mode, owing to loaded modules and their default parameter values.  Keep a close look at your device's syslog file (run `logread` to output it to the terminal) for kernel related errors.  In addition, take a look at the section [Hardware-specific configurations](#hardware-specific-configurations) for any comments related to the module used by your OpenWrt device.
{: .notice--warning }

## Mesh node basic config
It is time to configure the basics of our mesh network and nodes.  To do so, we will edit multiple files in `/etc/config/` but first, let's find out the capabilities of the detected radios in our wireless device, as follows

```
iw list
```

which will output something like this

```
Wiphy phy1
        wiphy index: 1
        max # scan SSIDs: 4
        max scan IEs length: 2261 bytes
        max # sched scan SSIDs: 0
        max # match sets: 0
        Retry short limit: 7
        Retry long limit: 4
        Coverage class: 0 (up to 0m)
        Device supports AP-side u-APSD.
        Device supports T-DLS.
        Available Antennas: TX 0x7 RX 0x7
        Configured Antennas: TX 0x7 RX 0x7
        Supported interface modes:
                 * IBSS
                 * managed
                 * AP
                 * AP/VLAN
                 * monitor
                 * mesh point
                 * P2P-client
                 * P2P-GO
                 * outside context of a BSS
        Band 2:
                Capabilities: 0x11ef
                        RX LDPC
                        HT20/HT40
                        SM Power Save disabled
                        RX HT20 SGI
                        RX HT40 SGI
                        TX STBC
                        RX STBC 1-stream
                        Max AMSDU length: 3839 bytes
                        DSSS/CCK HT40
                Maximum RX AMPDU length 65535 bytes (exponent: 0x003)
                Minimum RX AMPDU time spacing: 8 usec (0x06)
                HT TX/RX MCS rate indexes supported: 0-23
                Frequencies:
                        * 5180 MHz [36] (17.0 dBm)
                        * 5200 MHz [40] (17.0 dBm)
                        * 5220 MHz [44] (17.0 dBm)
                        * 5240 MHz [48] (17.0 dBm)
                        * 5260 MHz [52] (21.0 dBm) (radar detection)
                        * 5280 MHz [56] (21.0 dBm) (radar detection)
                        * 5300 MHz [60] (21.0 dBm) (radar detection)
                        * 5320 MHz [64] (21.0 dBm) (radar detection)
                        * 5500 MHz [100] (21.0 dBm) (radar detection)
                        * 5520 MHz [104] (21.0 dBm) (radar detection)
                        * 5540 MHz [108] (21.0 dBm) (radar detection)
                        * 5560 MHz [112] (21.0 dBm) (radar detection)
                        * 5580 MHz [116] (21.0 dBm) (radar detection)
                        * 5600 MHz [120] (21.0 dBm) (radar detection)
                        * 5620 MHz [124] (21.0 dBm) (radar detection)
                        * 5640 MHz [128] (21.0 dBm) (radar detection)
                        * 5660 MHz [132] (21.0 dBm) (radar detection)
                        * 5680 MHz [136] (21.0 dBm) (radar detection)
                        * 5700 MHz [140] (21.0 dBm) (radar detection)
                        * 5745 MHz [149] (21.0 dBm)
                        * 5765 MHz [153] (21.0 dBm)
                        * 5785 MHz [157] (21.0 dBm)
                        * 5805 MHz [161] (21.0 dBm)
                        * 5825 MHz [165] (21.0 dBm)
        valid interface combinations:
                 * #{ managed } <= 2048, #{ AP, mesh point } <= 8, #{ P2P-client, P2P-GO } <= 1, #{ IBSS } <= 1,
                   total <= 2048, #channels <= 1, STA/AP BI must match, radar detect widths: { 20 MHz (no HT), 20 MHz, 40 MHz }

        HT Capability overrides:
                 * MCS: ff ff ff ff ff ff ff ff ff ff
                 * maximum A-MSDU length
                 * supported channel width
                 * short GI for 40 MHz
                 * max A-MPDU length exponent
                 * min MPDU start spacing
        max # scan plans: 1
        max scan plan interval: -1
        max scan plan iterations: 0
        Supported extended features:
                * [ RRM ]: RRM
                * [ CQM_RSSI_LIST ]: multiple CQM_RSSI_THOLD records
                * [ CONTROL_PORT_OVER_NL80211 ]: control port over nl80211
                * [ TXQS ]: FQ-CoDel-enabled intermediate TXQs
                * [ AIRTIME_FAIRNESS ]: airtime fairness scheduling
                * [ SCAN_RANDOM_SN ]: use random sequence numbers in scans
                * [ SCAN_MIN_PREQ_CONTENT ]: use probe request with only rate IEs in scans
                * [ CAN_REPLACE_PTK0 ]: can safely replace PTK 0 when rekeying
                * [ CONTROL_PORT_NO_PREAUTH ]: disable pre-auth over nl80211 control port support
                * [ DEL_IBSS_STA ]: deletion of IBSS station support
                * [ MULTICAST_REGISTRATIONS ]: mgmt frame registration for multicast
                * [ SCAN_FREQ_KHZ ]: scan on kHz frequency support
                * [ CONTROL_PORT_OVER_NL80211_TX_STATUS ]: tx status for nl80211 control port support
Wiphy phy0
        wiphy index: 0
        max # scan SSIDs: 4
        max scan IEs length: 2257 bytes
        max # sched scan SSIDs: 0
        max # match sets: 0
        Retry short limit: 7
        Retry long limit: 4
        Coverage class: 0 (up to 0m)
        Device supports AP-side u-APSD.
        Device supports T-DLS.
        Available Antennas: TX 0x3 RX 0x3
        Configured Antennas: TX 0x3 RX 0x3
        Supported interface modes:
                 * IBSS
                 * managed
                 * AP
                 * AP/VLAN
                 * monitor
                 * mesh point
                 * P2P-client
                 * P2P-GO
                 * outside context of a BSS
        Band 1:
                Capabilities: 0x11ef
                        RX LDPC
                        HT20/HT40
                        SM Power Save disabled
                        RX HT20 SGI
                        RX HT40 SGI
                        TX STBC
                        RX STBC 1-stream
                        Max AMSDU length: 3839 bytes
                        DSSS/CCK HT40
                Maximum RX AMPDU length 65535 bytes (exponent: 0x003)
                Minimum RX AMPDU time spacing: 8 usec (0x06)
                HT TX/RX MCS rate indexes supported: 0-15
                Frequencies:
                        * 2412 MHz [1] (20.0 dBm)
                        * 2417 MHz [2] (20.0 dBm)
                        * 2422 MHz [3] (20.0 dBm)
                        * 2427 MHz [4] (20.0 dBm)
                        * 2432 MHz [5] (20.0 dBm)
                        * 2437 MHz [6] (20.0 dBm)
                        * 2442 MHz [7] (20.0 dBm)
                        * 2447 MHz [8] (20.0 dBm)
                        * 2452 MHz [9] (20.0 dBm)
                        * 2457 MHz [10] (20.0 dBm)
                        * 2462 MHz [11] (20.0 dBm)
                        * 2467 MHz [12] (20.0 dBm)
                        * 2472 MHz [13] (20.0 dBm)
                        * 2484 MHz [14] (disabled)
        valid interface combinations:
                 * #{ managed } <= 2048, #{ AP, mesh point } <= 8, #{ P2P-client, P2P-GO } <= 1, #{ IBSS } <= 1,
                   total <= 2048, #channels <= 1, STA/AP BI must match, radar detect widths: { 20 MHz (no HT), 20 MHz, 40 MHz }

        HT Capability overrides:
                 * MCS: ff ff ff ff ff ff ff ff ff ff
                 * maximum A-MSDU length
                 * supported channel width
                 * short GI for 40 MHz
                 * max A-MPDU length exponent
                 * min MPDU start spacing
        max # scan plans: 1
        max scan plan interval: -1
        max scan plan iterations: 0
        Supported extended features:
                * [ RRM ]: RRM
                * [ CQM_RSSI_LIST ]: multiple CQM_RSSI_THOLD records
                * [ CONTROL_PORT_OVER_NL80211 ]: control port over nl80211
                * [ TXQS ]: FQ-CoDel-enabled intermediate TXQs
                * [ AIRTIME_FAIRNESS ]: airtime fairness scheduling
                * [ SCAN_RANDOM_SN ]: use random sequence numbers in scans
                * [ SCAN_MIN_PREQ_CONTENT ]: use probe request with only rate IEs in scans
                * [ CAN_REPLACE_PTK0 ]: can safely replace PTK 0 when rekeying
                * [ CONTROL_PORT_NO_PREAUTH ]: disable pre-auth over nl80211 control port support
                * [ DEL_IBSS_STA ]: deletion of IBSS station support
                * [ MULTICAST_REGISTRATIONS ]: mgmt frame registration for multicast
                * [ SCAN_FREQ_KHZ ]: scan on kHz frequency support
                * [ CONTROL_PORT_OVER_NL80211_TX_STATUS ]: tx status for nl80211 control port support
```

Here, we are particularly interested in learning the following:

- How many radios there are (`phy0`, `phy1`);
- The supported **modes of operation** of each radio, and more specifically, that the device is indeed able to operate in `mesh point` mode, as shown under `Supported interface modes:`;
- The **total number of bands**. In this example, each radio can use only one band but one uses `2.4GHz` frequencies (`phy0`, `Band 1`), while the other uses `5GHz` frequencies (`phy1`, `Band 2`);
- For each band, the **acceptable channels**, as shown under `Frequencies:`.

With such information, we can now configure our radio devices in [`/etc/config/wireless`](https://openwrt.org/docs/guide-user/network/wifi/basic), as follows

```
vi /etc/config/wireless
```

and then edit each `config wifi-device` stanza accordingly.  In the TL-WDR4300, there's two default `config wifi-device` stanzas--namely, one for the 2.4GHz radio (called `radio0`) and another for the 5GHz radio (`radio1`).  After [changing and adding a few additional options](https://openwrt.org/docs/guide-user/network/wifi/basic), mine usually look like the following:

```
config wifi-device 'radio0'
        option type 'mac80211'
        option channel '1'
        #option txpower '20'  ##uncomment and edit to override default transmission power in dBm
        option hwmode '11g'
        option path 'platform/ahb/18100000.wmac'
        #option htmode 'HT20'  ##uncomment and edit to override default high throughput mode
        option country 'BR'  ##must match your country code
        option disabled '1'  ##change to 0 to enable it

config wifi-device 'radio1'
        option type 'mac80211'
        option channel '153'  ##all nodes must use the same channel
        #option txpower '21'  ##uncomment and edit to override default transmission power in dBm
        option hwmode '11a'
        option path 'pci0000:00/0000:00:00.0'
        #option htmode 'HT20'  ##uncomment and edit to override default high throughput mode
        option country 'BR'  ##must match your country code
        option disabled '0'  ##change to 1 to disable it
```

The comments in this and other config files are just for educational purpose. Feel free to remove them in your device's config files.
{: .notice--info }

In this guide, `radio1` (5GHz) will be used for the *mesh* traffic under the channel `153`, which means all other mesh nodes must use the **same channel**.  However, `radio0` (2.4GHz) will at times be used to create standard wireless access points (WAPs; 802.11b/g/n) for *non-mesh* clients, which means that none of the other nodes need to use the same channel with this radio.  In fact, it is strongly advised that 2.4GHz WAPs in close proximity should use **different channels**--namely, channels `1` (`2401–2423MHz` frequency range), `6` (`2426–2448MHz`), or `11` (`2451–2473MHz`) because those are non-overlapping channels and therefore, do not interfere with each other.  Because only mesh *bridges* will make use of `radio0`, the configuration indicates that it should be *disabled* by default.

The segmentation between mesh and non-mesh wireless communication adopted in this guide is best summarized by the following illustration:

[![Segmentation](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/segmentation.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/segmentation.jpg)

In addition, for `HT20/HT40` devices, stick to `HT20` if you are deploying the mesh in a crowded area, such as an apartment building; otherwise, the interference might make higher high-throughput (HT) actually less performant.  Finally, remember to edit the **country code** before enabling the radio and [follow country regulations](https://en.wikipedia.org/wiki/List_of_WLAN_channels#5_GHz_(802.11a/h/j/n/ac/ax)) when overriding the default transmission power (`option txpower`).  More often than not, you should actually *decrease* the `txpower` rather than increase it.  (For related material, see OpenWrt's articles on [Exceeding transmit power limits](https://openwrt.org/docs/guide-user/network/wifi/transmit.power.limits) and [Other transmit power issues](https://openwrt.org/faq/other_transmit_power_issues).)

Comment out or delete any `config wifi-iface` automatically generated after a fresh install by adding a `#` at the beginning of each line or typing `dd`, as follows

```
#config wifi-iface 'default_radio0'
#        option device 'radio0'
#        option network 'lan'
#        option mode 'ap'
#        option ssid 'OpenWrt'
#        option encryption 'none'
```

Then, at the end of the file, let's add a `wifi-iface` for the wireless mesh, called `wmesh`, as follows

```
config wifi-iface 'wmesh'
        option device 'radio1'  ##must match the name of a wifi-device
        option network 'mesh'  ##mesh stanza in /etc/config/network
        option mode 'mesh'  ##use 802.11s mode
        option mesh_id 'MeshCloud'  ##mesh "ssid"
        option encryption 'sae'  ##https://openwrt.org/docs/guide-user/network/wifi/basic#encryption_modes
        option key 'MeshPassword123'  ##password in plain text
        option mesh_fwding '0'  ##let batman-adv handle routing
        option mesh_ttl '1'  ##time to live in the mesh
        option mcast_rate '24000'  ##routes with a lower throughput rate won't be visible
        option disabled '0'  ##change to 1 to disable it
```

Because all mesh nodes must operate on the same channel, use the same authentication, etc., multiple config options are often dictated by the **"lowest common denominator"** across all mesh nodes--that is, the best possible configuration that will work with **all nodes**, not just the ones with the best hardware and software available.  For example, not all devices will necessarily be able to use SAE because it's very new and therefore, won't be able to connect to mesh networks that use it. Instead, you might want to set encryption to something like `psk2+aes`, which should be good enough for most devices out there. So, keep that in mind when configuring your mesh nodes.

**Save the file** and exit it.  

Now we need to configure [`/etc/config/network`](https://openwrt.org/docs/guide-user/base-system/basic-networking) to allow `wmesh` to use `batman-adv`.  To do so, edit the `network` file, as follows

```
vi /etc/config/network
```

and let's add an `interface` called `bat0` at the bottom of the file, as follows:

```
config interface 'bat0'
        option proto 'batadv'
        option routing_algo 'BATMAN_IV'
        option aggregated_ogms '1'
        option ap_isolation '0'
        option bonding '0'
        option bridge_loop_avoidance '1'
        option distributed_arp_table '1'
        option fragmentation '1'
        option gw_mode 'off'
        #option gw_sel_class '20'
        #option gw_bandwidth '10000/2000'
        option hop_penalty '30'
        option isolation_mark '0x00000000/0x00000000'
        option log_level '0'
        option multicast_mode '1'
        option multicast_fanout '16'
        option network_coding '0'
        option orig_interval '1000'
```

The `bat0` stanza has options with default values to facilitate fine-tuning later on.  The specifics about each option is derived from the [official `batctl` manual](https://downloads.open-mesh.org/batman/manpages/batctl.8.html).  For more details, refer to the [Protocol Documentation](https://www.open-mesh.org/projects/batman-adv/wiki#Protocol-Documentation) and more specifically, the [Tweaking](https://www.open-mesh.org/projects/batman-adv/wiki/Tweaking) section. 

In very nichey cases of highly mobile nodes, it is recommended to disable `aggregated_ogms` and lower `orig_interval`.
{:.notice--info}

Then, at the bottom of the `/etc/config/network` file, let's add an actual network interface to transport `batman-adv` packets, which in our case will be the network used by `wmesh` in the `/etc/config/wireless` config file, namely `mesh`, as follows:

```
config interface 'mesh'
        option proto 'batadv_hardif'
        option master 'bat0'
        option mtu '1536'
```

The [maximum transmission unit (MTU) size](https://en.wikipedia.org/wiki/Maximum_transmission_unit) should be anything between `1500` (usual size for Ethernet connections) and `2304` (usual size for WLAN connections).  However, because `batman-adv` adds its own header to packets traveling through the wireless mesh network, it is suggested to set a minimum of `1528` instead.  For a more detailed discussion, see [Fragmentation](https://www.open-mesh.org/projects/batman-adv/wiki/Fragmentation-technical) in the official batman-adv wiki.

**Save the file** and exit. 

Next, let's **reboot** the device (type `reboot` in the terminal) and once it comes back online, `ssh` into it once again because we want to check that our `batman-adv` interfaces are up.  To do so, type

```
ip link | grep bat0
```
and if the config is right, you should now see `bat0` and `wlan1` in the output. Similarly, we can use `batctl` to show us all active mesh interfaces, as follows

```
batctl if
```

If it all looks good, exit the `ssh` session, disconnect your laptop/PC from the wireless device (but keep it running nearby), and **go ahead and configure at least one other node**.  This can be done manually just like you've just configured the current node.  However, if your other mesh nodes are identical to the one you have already configured--that is, it is the same brand, model, and it is running the same OpenWrt version--then you can simply **copy the modified files** and then paste them on the `/etc/config/` directory of the new device.  To copy all such files from the configured device to your laptop/PC current directory, you can use `scp`, as follows:

```
scp -r root@192.168.1.1:/etc/config ./
```

which should create a `config` dir on your laptop/PC that has all the config files from the already configured device.  Then, once connected to another default OpenWrt device, it's just a matter of doing the reverse operation, as follows:

```
scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
  -r ./config/* root@192.168.1.1:/etc/config/
```

Because we are starting SSH sessions with different machines that have the same IP address (`192.168.1.1`), we can include `-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null` to disable checking the `known_hosts` file and redirect discovery of the new key to `/dev/null` instead of your user's `known_hosts`.  Alternatively, you can manually edit or delete your user's `known_hosts` file, which is usually found at `~/.ssh/known_hosts`.
{: .notice--warning }

Afterwards, `ssh` into one of the configured mesh nodes and type 

```
batctl n
```

which will show a table with the interfaces (`if-mesh`), MAC address of the neighboring mesh nodes, and when each of them was last seen.  Copy the MAC address (e.g., `f0:f0:00:00:00:01`) from each neighboring mesh node and ping them through the mesh (using `batctl p`) to see if they are all replying, as follows (press Ctrl+C to stop)

```
batctl p f0:f0:00:00:00:01
```

which should output something like the following if everything is working fine

```
PING f0:f0:00:00:00:01 (f0:f0:00:00:00:01) 20(48) bytes of data
20 bytes from f0:f0:00:00:00:01 icmp_seq=1 ttl=50 time=3.01 ms
20 bytes from f0:f0:00:00:00:01 icmp_seq=2 ttl=50 time=1.71 ms
20 bytes from f0:f0:00:00:00:01 icmp_seq=3 ttl=50 time=1.10 ms
--- f0:f0:00:00:00:01 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
rtt min/avg/max/mdev = 1.103/1.942/3.008/0.794 ms
```

Pat yourself on the back because you have successfully configured multiple mesh nodes! 

**Go ahead and configure all your mesh nodes the same way as before** and only then move on to bridges, gateways, and VLAN configs, as described next.

## Troubleshooting mesh issues
These are a few tips in case you run into issues when configuring gateways and bridges. 

To test node to node connectivity, connect to a mesh node and use 

```
batctl p MAC
```

in which `MAC` is another node's MAC address.  If the node does not reply, there's an issue with `batman-adv` or its configuration.  Try rebooting both nodes before doing anything else.

A more powerful tool to see what is going on in the mesh network is the `tcpdump` utility for `batman-adv`.  To use it, connect to a mesh node and type

```
batctl td batX
```

in which `batX` is a `batman-adv` interface (usually `bat0` but if you have more than one, then `bat1`, etc.).  This is quite useful when configuring VLANs because it will show the VLAN ID of each client as well.  In addition, it is possible to specify the VLAN ID in the `td` argument to constraint the output to one particular VLAN (e.g., `batctl td bat0.1`).  Depending on the scale of your mesh network, you might need to filter the output because things can get wild with `tcpdump` really fast.

For more details, see the [`batctl` manual](https://downloads.open-mesh.org/batman/manpages/batctl.8.html) or type `batctl -h` for cli usage information.  Keep in mind that while many options can be set via `batctl`, those changes are ephemeral--that is, they won't survive a reboot.  To make permanent changes, you need to add/edit the respective option in the `/etc/config/network` file and `batX` stanza.

It is worth mentioning that if you've been following my suggestion to name and take note of each device's MAC address, you can now create a file called `bat-hosts` in `/etc/` that contains pairs of `MAC address` and `name`, as follows:

```
f0:f0:00:00:00:00 node01
f0:f1:00:00:00:00 node02
f0:f2:00:00:00:00 node03
f0:f3:00:00:00:00 node04
```

This makes it much easier to identify the mesh nodes when issuing a command like `batctl n` and other debug tables.  As far as I'm aware, however, you have to create and update such file in each node because such information will just be available to nodes that have a `bat-hosts` file.

Finally, as mentioned before, keep an eye on your device's syslog for errors.  Module related issues are often associated with logged kernel errors (see the section [Hardware-specific configurations](#hardware-specific-configurations)) and wpa_supplicant has [multiple mesh-specific error codes](https://www.toomanyatoms.com/computer/disconnection_codes.html) to help you debug connectivity issues. The syslog can be inspected via `logread`, as follows:

```
logread
```

For usage information, type `logread -h`.

## Configuring common mesh networks
Here, we will see how to turn one or two of our configured mesh nodes into either a mesh **bridge** or a mesh **gateway**.  To avoid repetition, the configuration of bridges and gateways is described in more detail in the [first example](#gateway-bridge), and only a few small differences and observations are highlighted afterwards.  In addition, only IPv4 addresses and configurations were used but nothing prohibits the use of IPv6 in a mesh network.  

### Gateway-Bridge
This first example applies to the following topology:

[![Topology - Gateway-Bridge](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-gateway-bridge.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-gateway-bridge.jpg)

More specifically, the mesh has access to the WAN (**Network A**) via a *gateway device* and has a single, private network defined in the `192.168.10.0/24` IP range, which is used by both the **mesh network** devices and the **Network B**, non-mesh devices. The latter is enabled by a *bridge device* that works as an access point for non-mesh clients.

First, let's configure our **mesh gateway**.  

#### Mesh gateway configuration

Get one of the [pre-configured mesh nodes](#mesh-node-basic-config) that has at the very least two Ethernet ports, a LAN port and a WAN port.  (This, of course, is not required for a gateway device because [there are multiple ways to connect to WAN](https://openwrt.org/docs/guide-user/network/wan/internet.connection) but having separate physical ports makes the explanation much simpler to follow.  If that is not your case, just adapt to whatever interfaces you have configured that play the role of default `lan` and `wan`.)  

If you've configured this node as a [dumb access point](https://openwrt.org/docs/guide-user/network/wifi/dumbap) to temporarily give it access to the Internet while updating and installing packages, undo the configuration before proceeding because we will use both the `firewall` and `dhcp` config files in the gateway configuration.  
{: .notice--warning }

Connect your laptop/PC to the mesh node via cable using the LAN port--this way, the mesh node's IP address should still be `192.168.1.1`.  Then, `ssh` into the mesh node and let's take a look at the `/etc/config/network`, as follows

```
vi /etc/config/network
```

At the beginning of the file, there should a bunch of `config interface` for `loopback`, `lan`, and `wan`, for example, as well as a default `config device` for the `lan` bridge, called `br-lan`, and.  At the end, of course, there should be the mesh interfaces we previously created for the mesh node, namely `bat0` and `mesh`.  There are at least two options at this point: 

1. Create an entirely new local network for `bat0`, called `default`, at the expense of additional `dhcp` and `firewall` configuration; 
2. Or use the original `lan` network by simply bridging `bat0` at the `config device` `br-lan` stanza, as follows:
    ```
    config device                             
            option name 'br-lan'
            option type 'bridge'
            list ports 'eth0.1'  ##edit according to your device
            list ports 'bat0'
    ```

While the latter option is much easier than the former, we will choose the first here (i.e., create a new local network from the ground up) because it makes this tutorial compatible with multiple devices (switched or switchless) and it allows us to keep the original `lan` (`192.168.1.0/24`) as a management/debugging network.  (Later on, we will see how to bridge the original `lan` with any `bat0` VLAN, for example, so the original `lan` becomes accessible to the mesh as well.  For now, keep it simple.) 

At the bottom of the `/etc/config/network` file, let's add the following two stanzas:

```
config device                             
        option name 'br-default'
        option type 'bridge'
        list ports 'bat0'
                                      
config interface 'default'
        option device 'br-default'
        option proto 'static'
        option ipaddr '192.168.10.1'  ##static address on the new 192.168.10.0/24 network pool
        option netmask '255.255.255.0'
        list dns '1.1.1.1'  ##comment out to enable cloudflare dns
        list dns '8.8.8.8'  ##comment out to disable google dns
```

The first stanza (`config device`) creates a **bridge** (layer 2) for the `default` network, while the second stanza (`config interface 'default'`) creates the proper `default` **network** (layer 3) at the `192.168.10.0/24` pool, then sets a static IP address for this device at `192.168.10.1` and broadcasts to any client that they should use the external DNS servers `1.1.1.1` or `8.8.8.8`.  

**Save the file** and exit it. 

Next, let's edit the [`/etc/config/dhcp`](https://openwrt.org/docs/guide-user/base-system/dhcp) config to run a DHCP server on the new interface, as follows

```
vi /etc/config/dhcp
```

and at the end of the file, add the following

```
config dhcp 'default'
        option interface 'default'
        option start '50'  ##start leasing at addr 192.168.10.50
        option limit '100'  ##max leases, so for 100, leased addr goes from .50 to .149
        option leasetime '6h'
        option ra 'server'
```

**Save the file** and exit it. 

Finally, let's edit the [`/etc/config/firewall`](https://openwrt.org/docs/guide-user/firewall/firewall_configuration) config.  Many things that can be done at the firewall level and for this reason, it's often the most overwhelming part of the configuration.  Fortunately, in our case, all that we need to do here is simply **copy** the default `lan` config for the new `default`.  That is, anything that has `lan` we will 

1. copy the related config;
2. paste it immediately below the equivalent `lan` config;
3. and then change `lan` for `default` in the new config. 

Start by editing the `firewall` config file with `vi`, as follows

```
vi /etc/config/firewall
```

then the first set of configs we will add (immediately below the equivalent `lan` config) is the **zone** settings, namely 

```
config zone
        option name     default
        list network    'default'
        option input    ACCEPT
        option output   ACCEPT
        option forward  ACCEPT
```

the second set of configs will be for the **forwarding** settings, namely

```
config forwarding
        option src   default
        option dest  wan
```

and **that is it!**  

*Optional*. At the end of the `firewall` config file, there's a bunch of examples that you could use as template for more avdanced usage of this device's firewall.  Feel free to play around with them **once you get everything up and running**.

**Save the file** and exit it. 

*Optional*. Because we're not going to use IPv6, I suggest (a) disabling `odhcpd` altogether (run `/etc/init.d/odhcpd stop && /etc/init.d/odhcpd disable`) and (b) comment out any related IPv6 configuration in the files we just edited as well (e.g., remove configuration and references to `wan6`).

**Reboot** the device and connect the **WAN** cable to the device's **WAN Ethernet port**.

Once the device comes back online, `ssh` into it. Then, let's check the new configuration.  First, type

```
ip a
```

and as before, there should be `bat0` and `wlan1` interfaces, but now, your gateway device should have the static IP `192.168.10.1` in the new `192.168.10.0/24` network under `br-default`.

Similarly, because we preserved the original `lan` configuration, the device will continue to have the static IP `192.168.1.1` in the `192.168.1.0/24` network under `br-lan`.  This means it should always be reachable at its original IP address with an Ethernet cable directly connected to one of its LAN Ethernet ports.

If you **don't see the static IP on the new network**, then review the files we have just configured because there's likely a misconfiguration.  Don't expect to get things working until you fix this issue.

#### Mesh bridge configuration
The configuration of a mesh bridge is much simpler than of a mesh gateway because contrary to the gateway config, our mesh bridge doesn't require the use of a DHCP server and firewall.  In fact, both services will be disabled in a mesh bridge and instead, the ony thing we will do is join interfaces to make them look like a single one to any connected device.

As before, get one of the other [pre-configured mesh nodes](#mesh-node-basic-config) and to start things off, we will configure it as a [dumb access point](https://openwrt.org/docs/guide-user/network/wifi/dumbap).  Follow the instructions in the OpenWrt documentation, except for the following when configuring the original `lan` interface:

- Add `bat0` to a new `list ports` entry in the bridge stanza used by the `lan` interface, namely `br-lan`;
- Set a static IP for the device on the `192.168.10.0/24` network, such as `192.168.10.10`, pointing to our configured gateway at `192.168.10.1`;
- After all is done, rename every `lan` entry for `default` to make it consistent with the gateway configuration.  This, of course, is optional.

Once done, the configuration of the original `lan` interface and its `br-lan` bridge, which are now called `default` and `br-default`, respectively, should look something like this:

```
config device
        option name 'br-default'
        option type 'bridge'
        list ports 'eth0.1'
        list ports 'bat0'

config interface 'default'
        option device 'br-default'
        option proto 'static'
        option ipaddr '192.168.10.10'
        option netmask '255.255.255.0'
        option gateway '192.168.10.1'
        option dns '192.168.10.1'
``` 

After applying this configuration, it will let any ***non-mesh* clients** to join the mesh **via Ethernet cable**--that is, by connecting a cable to one of the **LAN ports** of the mesh bridge device.  As long as the gateway is reachable, everything should work like a standard network, you could use the device's own switch or connect the device to a switch and manage things there, and so on.

**Save the file** and exit it.

Similarly, you can create a **wireless access point** (WAP) for *non-mesh* clients, and the instructions in the [**dumb access point** documentation](https://openwrt.org/docs/guide-user/network/wifi/dumbap) will work just fine because it uses a network that is bridged with our mesh--namely, the default `lan`.  To avoid confusion, make sure to use **a different SSID** for the WAP(s) than the `mesh_id` used for the mesh.  In addition, use **a different radio** for the WAP(s) and set them to operate on **different channels**.  If that is not possible, that is probably okay for most home users but keep in mind that node hoping will start affecting performance quite noticeably. 

(*Optional*.) To illustrate, let's create a simple 2.4GHz WAP for your home devices that will make use of the `default` network.  This can be done by editing the `/etc/config/wireless` file as follows:

- In the 2.4GHz radio stanza (`radio0`), set `option disabled` to `'0'` to **enable** it:
   
   ```
   config wifi-device 'radio0'
           option type 'mac80211'
           option channel '1'
           #option txpower '20'
           option hwmode '11g'
           option path 'platform/ahb/18100000.wmac'
           #option htmode 'HT20'
           option country 'BR'
           option disabled '0'
   ```

- At the bottom of the file, create a new `config wifi-interface 'whome'` stanza that configures an access point (`option mode 'ap'`) that will make use of the `default` network. It should look similar to the following one when done:

   ```
   config wifi-iface 'whome'
           option device 'radio0'
           option network 'default'
           option mode 'ap'
           option ssid 'HomeWAP'  ##edit it
           option encryption 'psk2+aes'  ##https://openwrt.org/docs/guide-user/network/wifi/basic#encryption_modes
           option key 'MyStrongPassword123'  ##edit it
           option disabled '0'
   ```

- **Save the file** and exit.

Finally, in the terminal, make sure to disable `dnsmasq`, `odhcpd`, and the `firewall`, as follows

```
/etc/init.d/dnsmasq stop && /etc/init.d/dnsmasq disable 
/etc/init.d/odhcpd stop && /etc/init.d/odhcpd disable
/etc/init.d/firewall stop && /etc/init.d/firewall disable
```

**Reboot** your device and on your laptop/PC, **disable networking** altogether to force it to get a new IP from the bridge when it comes back online--alternatively, just disconnect the Ethernet cable.

Once the bridge is back online--wait at least a minute or two to give it enough time to connect to the mesh first--**re-enable networking** on your laptop/PC (or reconnect the Ethernet cable) and it should receive an IP addr from our mesh gateway in the `192.168.10.0/24` network (on a Linux distro, type `ip a` or `ip addr` or `ifconfig`), the bridge node should now be reachable at `192.168.10.10`, and you should be able to access the Internet from your laptop/PC through the mesh (try `ping google.com`, for example).  

If something doesn't work, review the config files mentioned here and then go over the ones for the gateway, reboot all mesh nodes (gateway first, then nodes, then bridge) and test again.

### Bridge-Bridge
This second example applies to the following topology:

[![Topology - Bridge-Bridge](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-bridge-bridge.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-bridge-bridge.jpg)

Contrary to the first example, there's no mesh gateway device and as such, this topology could be used to extend an already existing private network (Networks A and B) over the wireless mesh (all defined in the `192.168.10.0/24` IP range).  However, to make matters simple, we will assume that **the existing network has a gateway/firewall** in either Network A or B that can be found at the IP addr `192.168.10.1`, and **there's a DHCP server being advertised on the network**.  (If your existing Networks A and B are not defined in the `192.168.10.0/24` IP range, just edit your previous config files accordingly and the mesh network will follow your existing network instead.)

Config-wise, the mesh bridges in this topology are configured exactly [as in the first example](#mesh-bridge-configuration), except for the following differences in the configuration of the `/etc/config/network` config file:

- **Each mesh bridge** should have a different static IP address in the `default` interface, as indicated by `option ipaddr`.  For example, the first mesh bridge will have `option ipaddr '192.168.10.10'`, while the second mesh bridge will have `option ipaddr '192.168.10.11'`;

- The `option gateway '192.168.10.1'` in the `default` stanza must match an existing gateway on either Network A or B, and similarly, `option dns '192.168.10.1'` must point to a valid DNS resolver or forwarder;

- As mentioned before, if your existing Networks A and B are not defined in the `192.168.10.0/24` IP range, then just edit the config file accordingly.

### Gateway-Gateway
The third and final example applies to the following topology:

[![Topology - Gateway-Gateway](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-gateway-gateway.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-gateway-gateway.jpg)

Specifically, there's only one private network (mesh, defined in the `192.168.10.0/24` IP range) and notably, **two** mesh gateways.  This provides "high availability" of the Internet connection to mesh nodes and surprisingly enough, the configuration of each mesh gateway is [just like in the first example](#mesh-gateway-configuration), with the following exceptions

- Like in the [bridge-bridge example](#bridge-bridge), we must assign different static IP addresses to **each** mesh gateway.  This is done by editing the `/etc/config/network` config file, and in the `default` interface configuration, add a different IP addr next to the `option ipaddr` option.  For example, the first mesh gateway will have `option ipaddr '192.168.10.1'`, while the second mesh gateway will have `option ipaddr '192.168.10.2'`.

- Because we will now run **two** DHCP servers on **the same network**, we need to find a way of avoiding conflicts when assigning an IP address to new clients.  The easiest way of doing that is by assigning **different intervals** to each DHCP server running on the same network.  In OpenWrt, this is done by editing the `/etc/config/dhcp` config file, and in the `default` DHCP configuration, we add a different starting point next to the `option start` option.  For example, while the DHCP server running on the first gateway will have `option start '50'`, the DHCP server running on the second gateway will have `option start '150'` instead.  This way, the first DHCP server leases addresses from `192.168.10.50` to `.149`, whereas the second leases addresses from `192.168.10.150` to `.249`.

- In the `bat0` interface config of the `/etc/config/network` config file, we can now enable the `option gw_mode 'server'` and specify the WAN connection speed with `option gw_bandwidth '10000/2000'` (i.e., 10000kbps download and 2000kbps upload), as follows:
   
   ```
   config interface 'bat0'
           option proto 'batadv'
           option routing_algo 'BATMAN_IV'
           option aggregated_ogms '1'
           option ap_isolation '0'
           option bonding '0'
           option bridge_loop_avoidance '1'
           option distributed_arp_table '1'
           option fragmentation '1'
           option gw_mode 'server'
           #option gw_sel_class '20'
           option gw_bandwidth '10000/2000'  ##download/upload in kbps
           option hop_penalty '30'
           option isolation_mark '0x00000000/0x00000000'
           option log_level '0'
           option multicast_mode '1'
           option multicast_fanout '16'
           option network_coding '0'
           option orig_interval '1000'
   ```

   Similarly, now in each other **mesh node** (non-gateway devices), we set the `option gw_mode` to `'client'` instead of `'off'` and enable selection options, as follows:

   ```
      config interface 'bat0'
           option proto 'batadv'
           option routing_algo 'BATMAN_IV'
           option aggregated_ogms '1'
           option ap_isolation '0'
           option bonding '0'
           option bridge_loop_avoidance '1'
           option distributed_arp_table '1'
           option fragmentation '1'
           option gw_mode 'client'
           option gw_sel_class '20'  ##set to 1 for fast connection policy (BATMAN_IV)
           #option gw_bandwidth '10000/2000'
           option hop_penalty '30'
           option isolation_mark '0x00000000/0x00000000'
           option log_level '0'
           option multicast_mode '1'
           option multicast_fanout '16'
           option network_coding '0'
           option orig_interval '1000'
   ```

   This way, we can make each mesh node aware of the two gateways on the network (and their speeds) to better route mesh traffic.

To learn more about how `batman-adv` handles multiple gateways, read the official [Gateway documentation](https://www.open-mesh.org/projects/batman-adv/wiki/Gateways).

## Mesh VLANs
You don't need to configure VLANs in order to use `batman-adv` but it is one of its best features.  In brief, this is a way of using **our already configured** wireless mesh network to route traffic **to/from multiple and all networks** in a secure, isolated way (as far as VLANs go).  No need for additional hardware--the combination of OpenWrt and `batman-adv` turns even cheap wireless hardware into powerful virtual switches.  It's just a matter of tagging the additional (and virtual) networks instead of using the untagged `bat0` (or similarly, in a port-based analogy, "plugging" standard interfaces into different ports of our `bat0` switch).  This is a fairly advanced topic but surprisingly easy to incorporate to our existing `batman-adv` configuration.

Consider, for example, the following network

[![Topology - Mesh VLANs](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-mesh-vlans.jpg){:.PostImage .PostImage--large}](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/topo-mesh-vlans.jpg)

There's a single gateway device that provides WAN access to the mesh and Networks B, C, and D, which are all private networks defined in different IP ranges. In addition, all the Networks B, C, and D traffic should go via **any** mesh node in the mesh network while keeping them **isolated from each other**.  To make it easier to remember and distinguish each private network, let's call 

* Network **B** by `iot` network (`192.168.20.0/24`);
* Network **C** by `guest` network (`192.168.50.0/24`);
* and Network **D** by `default` network (`192.168.10.0/24`).

To implement such a mesh network with VLANs, we're going to follow very similar steps to [the first example of a gateway-bridge mesh network](#gateway-bridge), except for the following: 

* We will have two additional bridges in the network--that is, one for each mesh VLAN, for a total of three bridges. This is not a necessity but a matter of convenience to keep the example simple. The same bridge device can definitely bridge more than one mesh VLAN;
* In the gateway device, we will create VLAN IDs for the `iot` (**2**), `guest` (**5**), and `default` (**1**) networks, each with a separate set of DHCP server and firewall rules;
* In each bridge device, we will join the default `lan` with the **VLAN ID** of the mesh VLAN (`bat0.1`, `bat0.2`, `bat0.5`), instead of `bat0`.

Surprisingly enough, we don't need to do a thing about the **mesh nodes** that are not **gateways** or **bridges**--that is, the [mesh node basic config](#mesh-node-basic-config) is both necessary and sufficient for simple mesh nodes, even when using VLANs.  The only exception is if one of your mesh nodes is, for example, a laptop and you want it to use a particular mesh VLAN instead of the untagged `bat0`.  In our case, however, the pre-configured mesh nodes are ready to route traffic of any VLAN that belongs to `bat0`.

As before, let's start with the **gateway** configuration.

### Mesh gateway with VLAN configuration
First, configure the gateway **the same way** [as in the gateway-bridge example](#mesh-gateway-configuration).

Now, instead of `lan_bat0`, we're going to change it to `default` in the config files, then do the same for `iot` and `guest`.  So, if you're ready, `ssh` back into it and let's start by editing the `/etc/config/network` config file, as follows

```
vi /etc/config/network
```

and at the end, comment out the `lan_bat0` config, as follows

```
#config interface 'lan_bat0'
#        option type 'bridge'		#uncomment if adding other interfaces to ifname
#        option ifname 'bat0'
#        option proto 'static'
#        option ipaddr '192.168.10.1'	#static addr for this gateway on the 192.168.10.0/24 net
#        option netmask '255.255.255.0'
#        list dns '1.1.1.1'			#cloudflare dns server
#        list dns '8.8.8.8'			#google dns server
```

then below it, let's add a new interface for `default`, as follows

```
config interface 'lan_bat0_1'
#        option type 'bridge'	#uncomment if adding other interfaces to ifname
        option ifname 'bat0.1'
        option proto 'static'
        option ipaddr '192.168.10.1'
        option netmask '255.255.255.0'
        list dns '1.1.1.1'
#        list dns '8.8.8.8'	#make it use cloudflare
```

then another one for `iot`

```
config interface 'lan_bat0_2'
#        option type 'bridge'	#uncomment if adding other interfaces to ifname
        option ifname 'bat0.2'
        option proto 'static'
        option ipaddr '192.168.20.1'
        option netmask '255.255.255.0'
#        list dns '1.1.1.1'	#make it use google
        list dns '8.8.8.8'
```

and another one for `guest`

```
config interface 'lan_bat0_5'
#        option type 'bridge'	#uncomment if adding other interfaces to ifname
        option ifname 'bat0.5'
        option proto 'static'
        option ipaddr '192.168.50.1'
        option netmask '255.255.255.0'
#        list dns '1.1.1.1'	#make it use google
        list dns '8.8.8.8'
```

**Save the file** and exit it.

Now, let's edit the `/etc/config/dhcp` config file, as follows

```
vi /etc/config/dhcp
```

and once again, comment out all `lan_bat0` config, as follows

```
#config dhcp 'lan_bat0'
#        option interface 'lan_bat0'
#        option start 50		#start leasing at addr 192.168.10.50
#        option limit 100		#max leases, so for 100, leased addr goes from .50 to .149
#        option leasetime '3h'
#        option ra 'server'
```

then add a DHCP server config for the `default` interface below it

```
config dhcp 'lan_bat0_1'
        option interface 'lan_bat0_1'
        option start 50
        option limit 100
        option leasetime '12h'
        option ra 'server'
```

and like before, we will add another one for the `iot` interface

```
config dhcp 'lan_bat0_2'
        option interface 'lan_bat0_2'
        option start 50
        option limit 100
        option leasetime '6h'
        option ra 'server'
```

and another one for the `guest` interface

```
config dhcp 'lan_bat0_5'
        option interface 'lan_bat0_5'
        option start 50
        option limit 100
        option leasetime '1h'
        option ra 'server'
```

**Save the file** and exit it.

Finally, let's edit the `/etc/config/firewall` config file, as follows

```
vi /etc/config/firewall
```

and once again, comment out the `lan_bat0` configs, as follows

```
#config zone
#        option name     lan_bat0
#        list network    'lan_bat0'
#        option input    ACCEPT
#        option output   ACCEPT
#        option forward  ACCEPT
```
```
#config forwarding
#        option src   lan_bat0
#        option dest  wan
```

and below each one of them, add one for the `default` interface

```
config zone
        option name     lan_bat0_1
        list network    'lan_bat0_1'
        option input    ACCEPT
        option output   ACCEPT
        option forward  ACCEPT
```
```
config forwarding
        option src   lan_bat0_1
        option dest  wan
```

then another one for the `iot` interface

```
config zone
        option name     lan_bat0_2
        list network    'lan_bat0_2'
        option input    ACCEPT
        option output   ACCEPT
        option forward  ACCEPT
```
```
config forwarding
        option src   lan_bat0_2
        option dest  wan
```

and another one for the `guest` interface

```
config zone
        option name     lan_bat0_5
        list network    'lan_bat0_5'
        option input    ACCEPT
        option output   ACCEPT
        option forward  ACCEPT
```
```
config forwarding
        option src   lan_bat0_5
        option dest  wan
```

**Save the file** and exit.

**Reboot** the device.

Once the gateway device is back online--by the way, it should still be at `192.168.1.1` because the gateway's default `lan` is intact, so even if we fuck something up, we should be able to find the gateway via a direct cable connection--`ssh` into it once again and type

```
ip a
```

which now should show the new interfaces we created (e.g, `bat0.1@bat0`) and the static IP addr of your device in each one of them (`192.168.10.1`).  (As mentioned before, if the `option type 'bridge'` was enabled in the `/etc/config/network` config stanza, then there will be an additional interface with the `br-` prefix attached to it and the static IP addr of your device will be associated with it.)

If everything looks good, we're done with the gateway configuration!  We're now ready to tell our bridges which VLAN ID to join with their standard interfaces.

You don't need to use **interface names** such as `lan_bat0_1`; they can be whatever you find intuitive.  However, whatever you choose, **keep them short**--that is, less than 14 characters long--or you'll start experiencing config issues.
{: .notice--danger }

### Mesh bridge with VLAN configuration
Here, we'll also configure the bridges **the same way** as in the gateway-bridge example. However, each bridge device will bridge **a different VLAN ID**--namely, either `bat0.1` or `bat0.2` or `bat0.5`--with its default `lan`, instead of bridging `bat0` with its default `lan`.

Let's start with the Network B (**IoT**) bridge. 

Configure one of the mesh nodes [as in the gateway-bridge example](#mesh-bridge-configuration), except that in the default `lan` interface stanza of the `/etc/config/network` file, let's do the following:

* In `option ifname`, change `bat0` for `bat0.2`;
* In `option ipaddr`, change `192.168.10.` for `192.168.20.`;
* In both `option gateway` and `option dns`, change `192.168.10.1` for `192.168.20.1`;
* Then, the default `lan` stanza should look something like this
  ```
config interface lan
        option type 'bridge'
        option ifname 'eth0.1 eth1 bat0.2'	#ethX might be different for your device
        option proto 'static'
        option ipaddr '192.168.20.10'
        option netmask '255.255.255.0'
        option gateway '192.168.20.1'
        option dns '192.168.20.1'
```

**Save the file** and exit it.

**Reboot** your device. 

Once it comes back on, your laptop/PC will receive an IP addr from our mesh gateway in the `192.168.20.0/24` network, the bridge node should be reachable at `192.168.20.10`, and you should be able to access the Internet via the **IoT** network (try `ping google.com`, for example). 

**If something doesn’t work**, review the config files from your gateway and then from the bridge, then reboot the gateway and the bridge, and test again.

If this config is working, **repeat the same steps** in the config of the other two bridges, with the following exceptions

* In the Network C bridge (**Guest**), use `bat0.5` instead of `bat0.2`, and similarly, use the `192.168.50.` IP addr instead of `192.168.20.`;

* In the Network D bridge (**Default**), use `bat0.1` instead of `bat0.2`, and similarly, use the `192.168.10.` IP addr instead of `192.168.20.`;

*Optional*. When configuring a **Guest** WAP, for example, you can add `option isolate 1` to the relevant stanza in the `/etc/config/wireless` config file to deny client-to-client connectivity without the need of re-enabling the firewall in the bridge device.  If that's not enough, re-enable the firewall and configure it according to your needs--at the bottom of the `/etc/config/firewall` file, there are examples you can use as template.

## Getting started with batman-adv on any Linux device
OpenWrt makes using `batman-adv` a nearly trivial thing but you certainly don't need OpenWrt to implement a mesh network or even to use `batman-adv` in your mesh.  As mentioned before, `batman-adv` has long been added to the Linux Kernel and therefore, you should be able to configure it on pretty much *any* device running Linux.  

Even though the specifics of configuring network interfaces and managing connections might be different across Linux distributions, the initial steps always consist of the following:

1. Installing (in popular distros, this is *not needed*) and loading (*always* needed) the `batman-adv` Kernel module.
  `lsmod` will show a list of active modules, so we can `grep` it to check if the `batman-adv` module has already been loaded, as follows
  ```
lsmod | grep batman
  ```
  then if it isn't loaded, we add the `batman-adv` kmod to `/etc/modules` and load it with `modprobe`, as follows
  ```
# append batman-adv to /etc/modules
echo 'batman-adv' | sudo tee -a /etc/modules > /dev/null
# load the batman-adv module
sudo modprobe batman-adv
# check that the batman-adv module is now loaded
lsmod | grep batman
  ```
  Afterwards, you can check the [**sysfs**](https://en.wikipedia.org/wiki/Sysfs) of each network device in `/sys/class/net/` and there should be a `batman_adv` folder.  When the `batman-adv` module gets configured to use a particular network device, the files `batman_adv/iface_status` and `batman_adv/mesh_iface` will change their contents to reflect that. In addition, once enabled, `bat0` will show up as a new network device in `/sys/class/net/` and its options (e.g., `gw_mode`) can be modified by `echo`ing new values to their corresponding file in `/sys/class/net/bat0/mesh/`  (`echo 'client' > /sys/class/net/bat0/mesh/gw_mode`).
2. Installing the `batctl` package. On apt-based distros like Debian, you should be able to install it with the following
  ```
sudo apt install batctl
  ```
3. Using a combination of `iw` and `ip` to configure the network interfaces, as illustrated in the [B.A.T.M.A.N. quick start guide](https://www.open-mesh.org/projects/batman-adv/wiki/Quick-start-guide).  In our case, however, the wireless mode of operation (as in the specification of `type` in the `iw` interface creation command) is `mesh` (or `mp`), instead of `adhoc` (or `ibss`).
4. Using something like [wpa_supplicant](https://en.wikipedia.org/wiki/Wpa_supplicant) to manage connections.

If you know of a program that has a GUI and is able to handle such configurations on popular Linux distros, let me know about it. As far as I know, there's currently nothing like that and it would be so very useful.

[top](#){: .btn .btn--light-outline .btn--small}

# Bonus content: Physical computing
If your device has unused **general purpose I/O** pins, it's possible to do all sorts of things with them.  Check the [GPIO documentation](https://openwrt.org/docs/techref/hardware/port.gpio) for examples of how to install new LEDs and buttons, for instance.  ([Your device's OpenWrt page can be very useful as well](https://openwrt.org/toh/tp-link/tl-wr1043nd#gpios).)

Also, if you want to change the functionality of a few of the existing LEDs on your wireless device, check the [LED configuration documentation](https://openwrt.org/docs/guide-user/base-system/led_configuration).  Now that you have new mesh interfaces, you can use the LEDs to blink depending on the status of neighboring nodes, mesh gateways, or WAN connectivity through the mesh, to mention a few examples. (As mentioned before, [your device's OpenWrt page can be very useful here](https://openwrt.org/toh/tp-link/tl-wr1043nd#leds).)

[top](#){: .btn .btn--light-outline .btn--small}

# Bonus content: Moving from OpenWrt 19 to 21
If you just found this guide, you can safely ignore the content in this section because the entire article **has been updated** to make it compatible with OpenWrt 21.02, which is now the **current stable release**.  However, if you're currently running OpenWrt 19.07 and want to upgrade to 21.02, then read on.
{:.notice--warning}

When this guide was first written, [OpenWrt 19.07](https://openwrt.org/releases/19.07/start) was the current stable release version.  However, as of September 4th, OpenWrt 19.07 transitioned to old stable and [**OpenWrt 21.02**](https://openwrt.org/releases/21.02/start) is now the current stable release.  For one, this means that most device pages (e.g., [TP-Link Archer C7 AC1750](https://openwrt.org/toh/tp-link/archer_c7)) have been updated to link to the OpenWrt 21.02 firmware binaries. 

Of course, it is still possible to download and use the latest version of the OpenWrt 19.07 binaries (`19.07.8`) by looking for your device's target at [releases/19.07.8/targets](https://downloads.openwrt.org/releases/19.07.8/targets/).  However, it is generally a good idea to run the latest release version for multiple reasons, **security being the main one**.  Nonetheless, OpenWrt 21.02 introduces **new hardware requirements** and **changes to the network syntax** that you should not overlook before making the transition.  More specifically:

  - OpenWrt 21.02 introduces initial support for the [**Distributed Switch Architecture** (**DSA**)](https://www.kernel.org/doc/html/latest/networking/dsa/dsa.html).  Currently, however, this only applies to [a very limited number of devices](https://openwrt.org/releases/21.02/notes-21.02.0#initial_dsa_support).  If you have one of such devices, then make sure to read rmilecki's [mini tutorial for DSA network configuration](https://forum.openwrt.org/t/mini-tutorial-for-dsa-network-config/96998) because the syntax is a little bit different than the one used in this guide.

  - The [hardware requirements to run OpenWrt 21.02](https://openwrt.org/releases/21.02/notes-21.02.0#increased_minimum_hardware_requirements8_mb_flash_64_mb_ram) has increased to `8 MB` of flash memory and `64 MB` of RAM.  In the first version of this guide, I used the **TP-Link TL-WR1043ND (v1.8)** as an example of mesh node hardware, which has `8MB` of flash memory and `32MB` of RAM.  At first, I tried to use OpenWrt 21.02 with it but the system became **too unstable**, even after making several changes to multiple firmware images (e.g., removing LuCI altogether and adding `zram` support).  This is what prompted me to change the device in the examples to the **TP-Link TL-WD4300**, which is also a *low-end* router but it has `128MB` of RAM instead and importantly, it is a *dual-band* router that allows better segmentation of mesh vs non-mesh wireless traffic. 

  - There is a small but important [change in the configuration **syntax**](https://openwrt.org/releases/21.02/notes-21.02.0#new_network_configuration_syntax_and_boardjson_change) in `/etc/config/network`, namely:
    1. The option `ifname` is now called `device` in all `config interface` stanzas;
    2. The option `ifname` is now called `ports` in all `config device` stanzas of type `bridge`.

    Fortunately, it seems that the **old syntax** (as in the first version of this guide) **is still supported** but if you are using LuCI, you will run into compatibility issues and will be prompted to update.  To update it, take a closer look at the examples in the current version of the guide, which are now compatible with the network syntax introduced by OpenWrt 21.02.

  - There many other changes in OpenWrt 21.02 but from my experience so far, none of them are as relevant as the ones mentioned before.  For other highlights and additional information, please check the [official OpenWrt 21.02.0 release notes](https://openwrt.org/releases/21.02/notes-21.02.0).

Upgrading the firmware is as easy as it has always been: (a) go to the device's OpenWrt page, (b) download the new `*-sysupgrade.bin` binary, and then (c) flash it onto your device via LuCI.  If you're only using the terminal, first SSH into your device and make sure it has enough free memory by typing:

```
free
```

which should output something like this:

```
              total        used        free      shared  buff/cache   available
Mem:          27064       16168        6004         304        4892        8368
Swap:         13308         768       12540
```

and if the amount of `free` in the `Mem:` row is higher than the size of the binary, then copy the new binary to the root of the `/tmp/` directory via `scp` (or any other method) and run `sysupgrade` to upgrade your firmware to the latest release, as follows:

```
sysupgrade -v -n /tmp/*-sysupgrade.bin
```

Importantly, owing to changes in the network syntax, I strongly recommend to **discard all configuration files when making the transition**.  When upgrading via LuCI, make sure to *de*select the option to preserve configuration, and similarly, when upgrading via `sysupgrade`, add the `-n` argument command, as mentioned before. This, of course, means you will lose connection to the device if you are running the upgrade via a wireless connection, so make sure to use a cable for this particular operation.  

If the configuration files in `/etc/config/` have been extensively edited, make sure to make a backup of them before running the upgrade.
{:.notice--warning}

In addition, remember that the various packages supporting the use of `batman-adv` do not come with pre-built (default) images, which means that you won't be able to connect to your mesh node after an upgrade if you are relying on the mesh network to reach it.  If you do not want to reinstall all such packages (or cannot physically reach the nodes), check the updated section about [OpenWrt installation and initial configuration](#openwrt-installation-and-initial-configuration), which now features instructions on how to build customized images with pre-installed mesh packages.  Building your own images also means you can create default versions for all `/etc/config/` files (see `FILES=""` usage in the `make image` command) but **use caution with such feature** to avoid (soft) bricking your device.  At the very least, use only configurations you have already tested and that will work independently of any other node.

Overall, I like the clearer distinction between layers 2 and 3 introduced by the new network syntax in OpenWrt 21.02.  Once you get the hang of it, the configuration looks more organized and intuitive than before, and therefore, I think it is a step forward in the right direction.  Lastly, I would like to thank [SteveNewcomb](https://forum.openwrt.org/u/stevenewcomb) for testing--and letting me know about--`batman-adv` under the OpenWrt 21.02 release candidates.


[top](#){: .btn .btn--light-outline .btn--small}

# Final remarks

![Futurama Hubert Farnsworth](/assets/posts/2020-11-24-mesh-networking-openwrt-batman/futurama.jpg){:.PostImage}

Good news, everyone! You've reached the end of this tutorial, which means it's time to start planning your own mesh networking project.  I love to hear about different takes on the projects I post on my blog, so don't hesitate to [contact me](/contact/) if you just want to share or bounce a few ideas.  Different perspectives give an opportunity to learn, grow, and innovate.

## Other similar mesh solutions
If you find this guide overwhelming but you're still curious about mesh networking, take a look at the following alternatives (in alphabetical order):

* [Commotion Wireless](https://www.commotionwireless.net)
* [LibreMesh](https://libremesh.org)

They have pre-configured images that will work "out of the box" with compatible devices.  You might find instructive to start playing around with their software first and once comfortable, build your own configuration from a default (or customized from the source) OpenWrt image.

[top](#){: .btn .btn--light-outline .btn--small}

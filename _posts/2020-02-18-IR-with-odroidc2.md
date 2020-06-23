---
title: "Using the Odroid C2 IR receiver with LibreElec OS"
date: 2020-02-18
tags: odroid sbc
header:
  overlay_image: "https://1.bp.blogspot.com/-8FZvCPFMbgQ/XrQpSpxWgWI/AAAAAAAABaw/B_23PiUqm3YiWwv8tZ4bxtRL_zjygHGrQCK4BGAsYHg/odroidc2.jpg"
---

In case you didn't know, the [Odroid C2](https://www.hardkernel.com/shop/odroid-c2/) comes with an onboard infrared (IR) receiver.  Until a few days ago, I thought that such a receiver was only compatible with their own IR remote controller but it turns out you can use it with *any IR controller*.  We can do that with a package called [**lirc**](http://www.lirc.org/html/), which stands for **linux infrared remote control**.

[![Odroid C2 board](https://1.bp.blogspot.com/-8FZvCPFMbgQ/XrQpSpxWgWI/AAAAAAAABaw/B_23PiUqm3YiWwv8tZ4bxtRL_zjygHGrQCK4BGAsYHg/odroidc2.jpg){:.PostImage}](https://1.bp.blogspot.com/-8FZvCPFMbgQ/XrQpSpxWgWI/AAAAAAAABaw/B_23PiUqm3YiWwv8tZ4bxtRL_zjygHGrQCK4BGAsYHg/odroidc2.jpg)

This brief tutorial is for the **LibreElec OS** but one can use lirc with **any Linux distro** and the configuration won't be completely different than the one shown here.  (For the sake of completeness, I've tested with **LibreElec Official OS 9.0.2** running **Kodi 18.2**) Here's a step by step procedure to get the IR working with lirc:

1. SSH into your Odroid C2
```
ssh root@IP
```
2. Get a list of all available keys that you can map to your IR remote
```
irrecord --list-namespace
```
3. Kill running lircd, if there's any
```
ps aux | grep lircd
kill PID
```
4. Turn off all other IR compatible devices before moving forward
5. Go to ```/storage```, record IR custom keys and follow the instructions that will show yup on your screen:
```
cd /storage/
irrecord
```
6. If succesful, irrecord will genetare a .conf file on /storage/ with the name you provided at the beginnig. Copy the .conf file to ```/storage/.conf/lircd.conf```, as follows:
```
cp *.conf /storage/.conf/lircd.conf
```
7. Reboot your system
```
reboot now
```
8. Test your IR remote! If some key is missing, you can go back to ```irrecord``` and edit or record new keys.

That's it.  This is a short tutorial I wrote mostly to remind myself about this feature but hopefully, this tutorial is going to help someone else out there, too.

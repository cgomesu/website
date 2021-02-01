---
title: "Flashing Tasmota firmware onto Sonoff devices over-the-air: \"How-to\" using just a terminal and web-browser"
date: 2021-02-01 10:20:00 -0300
tags: iot tasmota sonoff wireless automation firmware
header:
  overlay_image: "/assets/posts/2021-01-30-ota-tasmota-sonoff/header.jpg"
  overlay_filter: "0.8"
toc: true
toc_label: "Table of Contents"
toc_icon: "list"
---

# Changelog
**Feb 1st, 2021**: Publication of the original article
{: .notice .notice--info }

[top](#){: .btn .btn--light-outline .btn--small}


# Introduction
**[Sonoff](https://sonoff.tech/)** devices are very popular home-automation devices developed by a Chinese company called [ITEAD](https://www.itead.cc). By default, they are controlled by a closed-source application developed by ITEAD--called [EWeLink](https://www.itead.cc/wiki/EWeLink_Introduction)--that can be installed onto iOS and Android cellphones, for example, making use of cloud services.  However, this makes it hard to integrate with existing home-automation servers, such as [Home Assistant](https://www.home-assistant.io/) and [OpenHAB](https://www.openhab.org/), or to simply control the devices locally--that is, without access to the Internet.  

Fortunately, there are alternatives that require flashing a different firmware onto Sonoff devices. The **[Tasmota](https://github.com/arendst/tasmota/) firmware**, for example, is a well-known alternative that provides easy integration with existing home-automation servers and let users control devices via multiple methods, such as webUI, HTTP requests, and MQTT, all of which can be accessed either locally or remotely or both.  On top of that, it is **free and open-source**.  Traditionally, flashing a Tasmota firmware onto a Sonoff device involves finding a **serial connection**, soldering a few cables/pins, and connecting the device to a **serial-to-USB** adapter.  However, more often than not, this takes time, knowledge about electronics, and soldering very small components.

ITEAD is aware that many users do not user their app or even their firmware for Sonoff devices.  Instead of forcing the use of their own software, recently, they have taken the much smarter path of *making it easier* for users to control Sonoff devices independently of their software via the release of a **[DIY mode](https://github.com/itead/Sonoff_Devices_DIY_Tools)** in the latest firmware versions.  In DIY mode, it is possible to use the device's **RESTful API** to monitor and control a variety of attributes, such as toggle a relay ON/OFF, checking the wireless signal quality, and more importantly for any Tasmota enthusiast, **flashing custom firmware over-the-air (OTA)**.

In this tutorial, I will describe how to flash the `tasmota-lite.bin` binary onto **Sonoff Mini** relays and any other **Sonoff device that can operate in DIY mode**.  This will be done OTA (wirelessly) using only `curl` to send `POST` requests and then either the BusyBox HTTP Daemon (`busybox httpd`) or a common webserver application (e.g., Apache, Nginx) to create a simple webserver to serve the Tasmota binary to the local network. There is no need to install and run any other application, executable, or whatever.  Software-wise, we just need a terminal and web-browser.

**DISCLAIMER**.The procedure described in this tutorial is **one-way**.  That is, once flashed the Tasmota firmware, it is **not possible to go back** to the original ITEAD firmware.
{:.notice .notice--warning}

**ATTENTION**. From this part and on, the tutorial describes procedures that involve working with **mains power**.  If you have not taken the necessary time to learn how to work with it safely, **stop right now** and ask for someone knowledgeable to assist and teach you.  **Do not take this warning lightly**.  Mains power can kill you or set your house on fire or both (or worse).  **Be safe**.
{:.notice .notice--danger}

[top](#){: .btn .btn--light-outline .btn--small}


# Requirements
Only follow this tutorial if your Sonoff device satisfies all of the following criteria:

* **Device**
   * [Sonoff Basic R3](https://www.itead.cc/sonoff-basicr3-wifi-diy-smart-switch.html)
   * [Sonoff RF R3](https://www.itead.cc/sonoff-rfr3.html)
   * [Sonoff Mini](https://www.itead.cc/sonoff-mini.html)

* **ITEAD firmware**
   * Version `3.5` or higher
     
     If running version `3.3` or `3.4`, you can try the [protocol v`1.4` documentation](https://github.com/itead/Sonoff_Devices_DIY_Tools/blob/master/SONOFF%20DIY%20MODE%20Protocol%20Doc%20v1.4.md#diy-mode-description) instead. The procedure does not require soldering but you need to open the device to connect the OTA jumper manually.  The interaction with the RESTful API is the same as described here, so come back to follow the procedure for flashing Tasmota OTA with `curl`.
     {:.notice .notice--info}

     Alternatively, if running an outdated version, install the EWeLink app, create a bogus acount, update the firmware to latest, uninstall the app, come back and follow this guide.
     {:.notice .notice--info}

That said, it's possible that this tutorial is partially or completely applicable to other Sonoff devices that can operate in **DIY mode**.  The ones listed here are the ones that [ITEAD listed as supported](https://github.com/itead/Sonoff_Devices_DIY_Tools).

## Additional hardware requirements
You **won't need** to do any soldering and won't even need to open the device.  However, we will need to **power the device using mains (110-220v AC) power**.  For the **Sonoff Mini**, for example, you need to wire it as follows:

[![Sonoff Mini Wiring](/assets/posts/2021-01-30-ota-tasmota-sonoff/sonoff-mini-wiring.jpg){:.PostImage}](/assets/posts/2021-01-30-ota-tasmota-sonoff/sonoff-mini-wiring.jpg)

Please note that color conventions, outlet format, etc., are not always the same accross countries.  Check (and double check) the ones in your country and property **before wiring the Sonoff device to mains**.  

For other Sonoff devices, check their manual.  At this point, you just need to provide power to the device itself--there is no need to connect it to whatever the relay is going to control, for example, or any switches.

Therefore, the only additional hardware requirements are the following:

* **Stripped mains power cable** with live (**L**) and neutral (**N**) wires;
* A wifi capable **laptop or PC**.

## Additional software requirements
I wrote this tutorial for **GNU/Linux** users.  That is, unless otherwise specified, the instructions assume that you are running a Linux distribution on your PC/laptop that will be used to interact with (and serve files to) the Sonoff device.  If running iOS, you might be able to adapt the procedure more easily than if you were running Windows or other OS.

Therefore, the only additional software requirement is the following:

* **GNU/Linux distro** installed on the host machine, preferrably **apt**-based distros, such as Debian or Ubuntu.

[top](#){: .btn .btn--light-outline .btn--small}


# Installation
This section describes how to flash the Tasmota firmware onto a Sonoff device OTA.  In brief, the procedure consists of (a) putting the Sonoff device in DIY mode, (b) configuring it to access your existing wireless network, (c) using a set of GNU/Linux utilities to interact with the device's RESTful API, (d) creating a simple webserver to serve the Tasmota firmware locally, and finally, (e) flashing the Tasmota firmware OTA.  Each of those steps is explained in more detail next.

## Preparing the Sonoff device
1. **Turn ON** your Sonoff device by connecting it to the mains power;

2. Enable the **DIY mode** by pressing its button for at **least 5 seconds**;

3. Once the DIY mode is enabled, the device will create a wireless access point (WAP) with the following credentials:
   ```
   SSID: ITEAD-X
   Password: 12345678
   ```
   Using your laptop/PC, find the SSID and enter the credentials to **join the ITEAD WAP**;

4. The Sonoff device will assign an IP to your laptop/PC in the `10.10.7.0/24` network, which you can check with `ip a`.  If it does, then **open a web-browser and type the following IP**:
   ```
   10.10.7.1:80
   ```
   If your laptop/PC was assigned to a different IP pool than `10.10.7.0/24`, then simply try the first address of whichever pool it was assigned to (e.g., if `10.10.1.0/24`, then `10.10.1.1`);

5. Follow the inscructions on the screen to **let the Sonoff device join your local network** via an existing WAP. **Save and let it reboot**.  

   In the meantime, tell your laptop/PC to **join the same local network** you configured in the Sonoff device webUI;

6. **Go to your DHCP server** and find out which IP address it assigned to the Sonoff device on the local network.  From now on, I will refer to its local IP address as `IP_SONOFF`.  Change it too its actual IP addr before running any command.

## Interacting with the RESTful API
1. Now, we will start using `curl` to send `POST` requests and pipe the output to `jq` to parse the `json` output.  Later on, we will use `wget` to download the Tasmota binary from its latest release.  To make sure all utlities are installed on your distro and you are running their latest version, **open a terminal** and run the following command:
   ```
   sudo apt update && sudo apt install curl jq wget -y
   ```

   If not using an `apt` based distro, simply adapt the code to use your package manager instead.
   {:.notice .notice--info}

2. Let's check that the Sonoff device's API is working and **get information** about it, as follows:
   ```
   curl -v -H "Content-Type: application/json" -d "{"data": {}}" IP_SONOFF:8081/zeroconf/info | jq '.'
   ```
   which should output something like this:
   ```json
   {
    "seq": 1,
    "error": 0,
    "data": {
        "switch": "off",
        "startup": "off",
        "pulse": "off",
        "pulseWidth": 2000,
        "ssid": "SSID_WAP",
        "otaUnlock": false,
        "fwVersion": "3.5.0",
        "deviceid": "ID_DEVICE",
        "bssid": "BSSID_WAP",
        "signalStrength": -48
        }
   }
   ```

3. Of note, check that `otaUnlock` is `false`, which means that currently, it is not possible to flash a custom firmware OTA.  To enable it, we need to **set `otaUnlock` to `true`**, as follows:
   ```
   curl -v -H "Content-Type: application/json" -d "{"data": {}}" IP_SONOFF:8081/zeroconf/ota_unlock | jq '.'
   ```
   and we can now verify that OTA is unlocked by getting the device's info once again, as follows:
   ```
   curl -v -H "Content-Type: application/json" -d "{"data": {}}" IP_SONOFF:8081/zeroconf/info | jq '.'
   ```
   which should indicate that `otaUnlock` is now `true`:
      ```json
   {
    "seq": 3,
    "error": 0,
    "data": {
        "switch": "off",
        "startup": "off",
        "pulse": "off",
        "pulseWidth": 2000,
        "ssid": "SSID_WAP",
        "otaUnlock": true,
        "fwVersion": "3.5.0",
        "deviceid": "ID_DEVICE",
        "bssid": "BSSID_WAP",
        "signalStrength": -48
        }
   }
   ```
   If you are not seeing this, review your steps until now.  You can still reset the device powering it OFF and back ON, and the device should come back in DIY mode once again (test with the first `curl` command, for example).

4. **Download the latest `tasmota-lite.bin` binary** from the Tasmota Github repository to your user's `Downloads/tasmota` directory, as follows:
   ```
   mkdir /home/${USER}/Downloads/tasmota
   wget -P /home/${USER}/Downloads/tasmota $(curl -s https://api.github.com/repos/arendst/Tasmota/releases/latest | grep '\"browser_download_url.*tasmota-lite.bin\"' | cut -d '"' -f 4)
   ```
   This should write a `tasmota-lite.bin` file onto your user's `Downloads/tasmota/` directory. If it does not, please [let me know about it](/contact) and in the meantime, try downloading the file manually from the [Tasmota Github repo](https://github.com/arendst/Tasmota/releases).

5. Check the `tasmota-lite.bin` SHA256 signature and save it to file `tasmota-lite-sha256.txt`, as follows:
    ```
    sha256sum "/home/${USER}/Downloads/tasmota/tasmota-lite.bin" > "/home/${USER}/Downloads/tasmota/tasmota-lite-sha256.txt"
    ```
    **Take note of the signature** and paste it whenever I refer to `BIN_SHA256`.  You can find the signature with the following command (or just open the text file and copy the first string):
    ```
    cat "/home/${USER}/Downloads/tasmota/tasmota-lite-sha256sum.txt" | cut -d ' ' -f 1
    ```
    This signature is used to check the firmware integrity after the Sonoff device is done downloading it from a webserver. This is done to prevent the device from flashing a corrupted firmware, for example, because a corrupted file will likely yield a different SHA256 signature.

## Webserver configuration
The webserver has a few peculiar requirements (e.g., needs to accept the `Ranges` header, run in `http` instead of `https`) that does not allow us to point to Tasmota's OTA website, Github releases, or any other official source of the Tasmota firmware binary. Fortunately, we can run a webserver on the local network that satisfies all requirements by the ITEAD firmware and in my experience, the easiest way to do that is to either use the **BusyBox HTTP Daemon** or run an **Apache** webserver--or lighttp or Nginx, for instance, but **do not** try Python's `http.server` or PHP because they do not accept partial content.  

BusyBox and Apache are the easiest applications to get up and running because they often come pre-installed in several Linux distros, such as Debian, which means that all you need to do is run a simple command or enable a systemd service that controls the webserver application.  Here, I will focus on the former (`busybox`) rather than the latter (`apache2`) because in my opinion, BusyBox makes this step rather trivial.

If you have never heard of [BusyBox](https://www.busybox.net/) before, it is a sort of Swiss Army knife for Linux, Android, and many other POSIX environments.  It encapsulates multiple Unix utilities into a single and small executable, and for this reason, it is often used in resource limited applications (e.g., routers/modems, other embedded systems, cellphones).  Here, we will take advantage of one of its utilities, namely the **[HTTP Daemon](https://openwrt.org/docs/guide-user/services/webserver/http.httpd)**, to create a minimal webserver to serve the `tasmota-lite.bin` firmware to the Sonoff device.

1. First, make sure you have `busybox` **installed**, as follows:
    ```
    sudo apt update && sudo apt install busybox -y
    ```

    If not using an `apt` based distro, simply adapt the code to use your package manager instead.
    {:.notice .notice--info}


2. Then, **create a webserver** with your user's `Downloads/tasmota` directory as root and using port `2020`:
   ```
   busybox httpd -p 2020 -h "/home/${USER}/Downloads/tasmota/"
   ```

   If you are running a **firewall** on your laptop/PC, make sure to allow incoming TCP/UDP to port `2020` as well.  Otherwise, other devices on your local network won't be able to access the webserver you just created.
   {:.notice .notice--warning}

3. Find the local **IP address of your laptop/PC** with `ip a` (e.g., `192.168.1.100`).  The address should be reachable by the Sonoff device (e.g., it is on the same subnet).  From this point forward, I will refer to the host IP address by `IP_HOST`.

4. *Optional.* Using another wifi capable device, test that the `tasmota-lite-bin` file is **available to the local network** by typing the following on a web-browser:
   ```
   http://IP_HOST:2020/tasmota-lite.bin
   ```

   If correctly configured, the device should be able to download the binary.  Otherwise, review your steps.

If you would like to use one of the most common webservers, such as Apache, Lighttpd, and Nginx, install any one of them using your system's package manager and then *copy the files in your user's* `Downloads/tasmota` *dir to the webserver's root*, which by default is usually on `/var/www/html`.
{:.notice .notice--info}

## Flashing the Tasmota firmware
1. Flash the `tasmota-lite.bin` binary onto the Sonoff device via a `curl` `POST`: (Change `IP_HOST`, `BIN_SHA256`, and `IP_SONOFF` **before running the command**.  Double check everything to make sure it is correct as well.)
   ```
   curl -v -H "Content-Type: application/json" -d '{"data": {"downloadUrl": "http://IP_HOST:2020/tasmota-lite.bin", "sha256sum": "BIN_SHA256"}}' IP_SONOFF:8081/zeroconf/ota_flash | jq '.'
   ```
   You should get an HTTP response fairly quickly.  The following codes indicate that there was an **error** and you should review your steps until now:

   * **403**: The operation failed and the OTA function was not unlocked.
   * **408**: The operation failed and the pre-download firmware timed out.
   * **413**: The operation failed and the request body size is too large.  Make sure the tasmota firmware is the right size for your device.  You should try the `tasmota-lite.bin` before anything else.
   * **424**: The operation failed and the firmware could not be downloaded. Check that your webserver and firmware file are both reachable by other devices on the same local network; check for typos in the URL.
   * **471**: The operation failed and the firmware integrity check failed.

2. **Wait 2 minutes** or so.

3. Once the device is done flashing the new firmware, **open a web-browser** and try reaching the Tasmota webUI at the following URL:
   ```
   IP_SONOFF:80
   ```
   If corretcly installed, you will be greeted by the following Tasmota webUI:

   [![Sonoff webUI](/assets/posts/2021-01-30-ota-tasmota-sonoff/sonoff-webui.jpg){:.PostImage}](/assets/posts/2021-01-30-ota-tasmota-sonoff/sonoff-webui.jpg)

   and then, see the next section for the basic settings; Otherwise, review your steps and try reflashing the firmware.

[top](#){: .btn .btn--light-outline .btn--small}


# Basic Tasmota configuration
Before wiring your device to anything else, you should first **configure** and **test** it.  Configuration-wise, there is a lot of possibilities with a Tasmota firmware.  If you've never used Tasmota before, check Robbert's ([The Hook Up](https://www.youtube.com/channel/UC2gyzKcHbYfqoXA5xbyGXtQ)) introduction video:

{% include video id="08_GBROKQH0" provider="youtube" %}
{:. text-center}

At the very least, you should **update the firmware Template** to use the one appropriate for your device.  Templates are device-specific definitions of how their GPIO pins are assigned.  

1. **Copy the template** for your Sonoff device:

* [Sonoff Basic R3](https://www.itead.cc/sonoff-basicr3-wifi-diy-smart-switch.html):
   ```json
   {"NAME":"Sonoff Basic","GPIO":[17,255,255,255,255,0,0,0,21,56,255,0,0],"FLAG":0,"BASE":1}
   ```

* [Sonoff RF R3](https://www.itead.cc/sonoff-rfr3.html)
   ```json
   {"NAME":"Sonoff RF","GPIO":[17,255,255,255,255,0,0,0,21,56,255,0,0],"FLAG":0,"BASE":2}
   ```

* [Sonoff Mini](https://www.itead.cc/sonoff-mini.html)
  ```json
  {"NAME":"Sonoff Mini","GPIO":[17,0,0,0,9,0,0,0,21,56,0,0,255],"FLAG":0,"BASE":1}
  ```

2. **Open a web-browser** and navigate to the Tasmota webUI.

3. On the webUI, go to **Configuration** > **Configure other** and then **paste the tempalte** into the *Template field*, check the *Activate* box and hit **Save**.  The device will then reboot.

4. Once the device is back up, check that its name is now the same as in the `NAME` property value.  For the Sonoff mini template, for example, it should be `Sonoff Mini`.  You can further configure your template at **Configuration** > **Configure Template** to assign new components, if at all possible.  (The Mini does have an exposed GPIO available that was previously used by the ITEAD firmware for flashing mode, which is not going to be used anymore.)

## Fixing the timezone
If you installed a pre-compilled firmware, there's a chance your device is using the incorrect timezone.  To check the current timezone, go to the webUI main page and then **Console**. Now, type the following:
```
timezone
```
and if that is incorrect, to change it, enter the same `timezone` command with a value equal to your region's [standardized time zone](https://upload.wikimedia.org/wikipedia/commons/8/88/World_Time_Zones_Map.png) timezone.  For America/Sao_Paulo, for example, that would be `-3`, which can be set in your Tasmota device as follows
```
timezone -3
```

[top](#){: .btn .btn--light-outline .btn--small}


# Final remarks
Tasmota is a **featureful firmware** and it is worth taking a look at the **[official documentation](https://tasmota.github.io/docs/)** to learn about the possibilities.  If you run into issues, go to their [Github repository](https://github.com/arendst/tasmota/), search their open and closed issues, and if you do not find an answer to your problem, open a new one.  

Come back to this website every once in a while to check for changes in the [changelog](#changelog).  I try to keep all my guides up-to-date as much as possible because I actually use them myself.

[top](#){: .btn .btn--light-outline .btn--small}


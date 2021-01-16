---
title: "Tasmota webcam server for the ESP32-cam"
date: 2021-01-15 9:00:00 -0300
tags: iot esp32 tasmota mqtt cam webcam surveillance wifi wireless network
header:
  overlay_image: "/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/header.jpg"
  overlay_filter: "0.85"
toc: true
toc_label: "Table of Contents"
toc_icon: "list"
---

# Changelog
**Jan 16th, 2021**: Publication of the original article
{: .notice .notice--info }

[top](#){: .btn .btn--light-outline .btn--small}


# Introduction
The ESP32 is a cheap and low-power microcontroller developed by [Espressif](https://www.espressif.com).  In addition to its low-cost, the ESP32 is known for its tiny and robust design, the versatility of its applications, and for having onboard Wi-Fi and Bluetooth.  It is sold world-wide (e.g., [Amazon](https://www.amazon.com/s?k=esp32), [Aliexpress](https://aliexpress.com/wholesale?SearchText=esp32), [Mercado Livre](https://lista.mercadolivre.com.br/esp32)) in a variety of boards (e.g., NodeMCU, TTGO, Lolin32).  

In this tutorial, I will talk about one type of ESP32 board that has an **integrated camera module**, called the **ESP32-cam**, which can be found for [less than US$10](https://www.amazon.com/s?k=esp32+cam&s=price-asc-rank&ref=sr_st_price-asc-rank).  The goal is to build a cheap alternative to commercial wireless cameras using an open-source firmware that can be easily controlled via HTTP or MQTT and integrated to an existing camera surveillance server (e.g., [MotionEye](https://github.com/ccrisan/motioneye/), [Shinobi](https://shinobi.video/), [ZoneMinder](https://www.zoneminder.com/), [iSpy](https://www.ispyconnect.com/)) or multi-purpose automation server (e.g., [HomeAssistant](https://www.home-assistant.io/), [OpenHAB](https://www.openhab.org/), [NodeRed](https://nodered.org/)) by capturing its live stream from a simple MJPEG URL.  All that can be accomplished with **[Tasmota](https://tasmota.github.io/)** and its (beta) **[webcam server firmware for the ESP32-cam](https://github.com/arendst/Tasmota/tree/firmware/firmware/tasmota32)**.

---

If you're new to **ESP32** boards, check Bill's ([DroneBot Workshop](https://www.youtube.com/channel/UCzml9bXoEM0itbcE96CB03w)) review video:

{% include video id="xPlN_Tk3VLQ" provider="youtube" %}
{:. text-center}

---

For a comparison of a few different **ESP32-cam** boards, check [Andreas Spiess'](https://www.youtube.com/channel/UCu7_D0o48KbfhpEohoP7YSQ) video:

{% include video id="5IhhyJjjCxo" provider="youtube" %}
{:. text-center}

---

[top](#){: .btn .btn--light-outline .btn--small}


# Overview
This tutorial was organized as follows.  First, I presented the motivation behind the use of Tasmota32 webcam server over one of the most common firmwares for the ESP32-cam, the Espressif CameraWebServer Arduino sketch.  This is followed by a list of the main hardware components involved into flashing a firmware onto the ESP32-cam.  Most of the tutorial focused on the installation and configuration of the Tasmota32 webcam server using a GNU/Linux OS.

[top](#){: .btn .btn--light-outline .btn--small}


# Why Tasmota?
Tasmota was created and it is still maintanted by [Theo Arends](https://github.com/arendst). It started as hacky alternative to the [Sonoff](https://sonoff.tech/) commercial firmware and moved onto an independent, [free and open-source project](https://github.com/arendst/Tasmota) that provides multiple firmwares for ESP8266-based devices.  The firmwares come with a simple webUI that let's you control and configure the board main modules as well as integration with a MQTT server and more. Even though Tasmota [support for the ESP32 is still in beta development](https://tasmota.github.io/docs/ESP32/), my experience with it has been very positive.  

One of the main webcam firmwares for the **ESP32-cam** is the one provided by Espressif themselves, the [CameraWebServer](https://github.com/espressif/arduino-esp32/tree/master/libraries/ESP32/examples/Camera/CameraWebServer) Arduino sketch.  This one has features that the Tasmota32 webcam firmware does not offer, such as face recognition and motion detection.  However, my experience with the **video streaming** has been negative.  Specifically, the streaming runs smoothly when the video resolution is low (640x480) but it strugles quite a bit when running at medium to high resolutions--that is, the number of frames per second decreases noticeably.  I've also noticed that the board runs very hot when running the CameraWebServer Arduino sketch, even when the most CPU intensive tasks (motion detection and face reconition) are disabled. 

On the other hand, the **[Tasmota32 webcam server](https://github.com/arendst/Tasmota/tree/firmware/firmware/tasmota32)** seems to perform much better in the areas the CameraWebServer Arduino sketch strugles with.  More specifically, the streaming is smoother and the board does not seem to get as hot.  I've not had a chance to investigate why this happens and to measure the actual difference in frames per second and temperature, so don't take my opinion too seriously.  Also, I cannot tell if this happens for all ESP32-cam boards because I've only tested with the **AI-Thinker** module.  Overall, however, my experience with the Tasmota32 firmware has been better than with the Espressif firmware in the area that I think is the most relevant one for a camera module, namely video streaming performance.  On top of that, the Tasmota firmware offers a multitude of methods to interact with the ESP32-cam remotely, while the Espressif sketch is very limited in that regard.

---

If you've never heard of Tasmota before, check Robbert's ([The Hook Up](https://www.youtube.com/channel/UC2gyzKcHbYfqoXA5xbyGXtQ)) introduction video:

{% include video id="08_GBROKQH0" provider="youtube" %}
{:. text-center}

---

[top](#){: .btn .btn--light-outline .btn--small}


# Hardware
To make a single wireless camera based on the ESP32-cam board, you'll need at least the following items:

* Board: 
  * 01x [ESP32-CAM, AI-Thinker board](https://www.amazon.com/s?k=esp32cam+ai-thinker)

  [![ESP32cam](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam.jpg){:.PostImage}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam.jpg)

* USB to TTL adapter:
  * 01x [FTDI FT232RL USB to TTL/serial module with 5v/3v3 voltage jumper](https://www.amazon.com/s?k=ftdi+ft232RL+usb+to+ttl)

  [![FTDI FT232RL](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/ftdi-usb-ttl.jpg){:.PostImage}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/ftdi-usb-ttl.jpg)

* Cables:
  * 05x [Female-Female dupont/jumper wires](https://www.amazon.com/s?k=female+dupont+wires)

  [![Female dupont wires](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/female-dupont.jpg){:.PostImage}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/female-dupont.jpg)

  * 01x USB cable compatible with your USB to TTL adapter: Check the USB type and use a short cable to decrease resistance as much as possible because initially, we will be powering the ESP32-cam via your computer's USB port.

  [![USB cable](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/usb-cable.jpg){:.PostImage}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/usb-cable.jpg)

*Optional.* If you wish to add a case to your ESP32-cam project, there are many [3D printed options to choose from](https://duckduckgo.com/?q=esp32-cam+case).  (Notice that the use of an external antenna requires (de)soldering very small components.  If you plan on installing the camera on an area with good wireless coverage, don't bother with an external antenna.  Otherwise, plan accordingly.)  In addition, you might want to add a [USB to DIP adapter](https://www.amazon.com/s?k=USB+to+DIP) to your shopping list in order to power your ESP32-cam without the USB to TTL adapter.  The USB to DIP adapter is highly dependent on the casing choice and most 3D printed enclosure projects for the ESP32-cam include assembly instructions.  **Casing and assembly are not covered in this tutorial**, owing to the plethora of alternatives.

[top](#){: .btn .btn--light-outline .btn--small}


# Installation
This guide assumes you're running a **Linux** distribution, and more specifically, an **apt-based distro**, such as Debian or Ubuntu.  If you're running a different distro, simply change the apt code to reflect your system's package manager.  For non-Linux users, check [Tasmota's Getting Started](https://tasmota.github.io/docs/Getting-Started/) but use the binaries mentioned here and come back for the post-flashing configuration of the webcam server.

## Required packages and user permissions
Before we can flash the Tasmota32 webcam server onto the ESP32-cam, we will need to install a few packages and configure the permissions of our Linux user.

1. Open a terminal and install the required packages:
   ```
   sudo apt update && sudo apt install wget python3 python3-pip
   ```

2. Install `esptool.py` via `pip3`:
   ```
   pip3 install esptool
   ```

3. Find out if `esptool.py` can be found in your user's `$PATH`. (Alternatively, when required to run `esptool.py`, instead of `esptool.py OPTIONS`, run as `python3 -m esptool OPTIONS`. If you choose to do this, skip this and the next step.)
   ```
   whereis esptool.py
   ```

4. If `esptool.py` was not found, it means your user's `.local/bin` is not in your `$PATH`.  Add it as follows:
   ```
   echo "export PATH="$HOME/.local/bin:$PATH"" | tee -a "$HOME/.bashrc" > /dev/null
   ```

5. Connect your ESP32-cam to the USB to TTL/serial adapter in flash mode:
   
   [![ESP32cam flash mode](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-wiring-flash-mode.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-wiring-flash-mode.jpg)

   **Attention.** Make sure your USB to TTL adapter has **VCC in 5V mode** and in the ESP32, the VCC cable is connected to the 5V pin.  Double check the wiring before moving on.
   {: .notice .notice--warning}

6. Connect the adapter to a USB port on your computer and check the new device in `/dev/`:
   ```
   ls -l /dev/ttyUSB*
   ```

7. Add your `$USER` to the same group as `/dev/ttyUSB*` (it's usually `dialout` but if different, change in the command below) and `tty`:
   ```
   sudo usermod -a -G dialout ${USER} && sudo usermod -a -G tty ${USER}
   ```

8. Log off and back on.  (If you continue to run into permission issues, try rebooting instead.  You can check your user's permissions with `id ${USER}`.)

## Flashing Tasmota32 webcam server
We are now ready to flash the Tasmota firmware.  For reference, the official information is available at [https://tasmota.github.io/docs/ESP32](https://tasmota.github.io/docs/ESP32).

1. Create a `Tasmota32` dir in `/opt`:
   ```
   cd /opt && sudo mkdir Tasmota32
   ```

2. Download the `tasmota32-webcam.bin` binary and the needed ESP32 Tasmota binaries from the official Github repo via `wget`:
   ```
   sudo wget -P Tasmota32/ https://github.com/arendst/Tasmota/raw/firmware/firmware/tasmota32/tasmota32-webcam.bin https://github.com/arendst/Tasmota/raw/firmware/firmware/tasmota32/ESP32_needed_files/boot_app0.bin https://github.com/arendst/Tasmota/raw/firmware/firmware/tasmota32/ESP32_needed_files/bootloader_dout_40m.bin https://github.com/arendst/Tasmota/raw/firmware/firmware/tasmota32/ESP32_needed_files/partitions.bin 
   ```

3. Change ownership to your user instead of `root`:
   ```
   sudo chown -R ${USER} Tasmota32/
   ```

4. Make sure your ESP32-cam is still connected to your computer in **flash mode** (GPIO0-GND jumper).

5. Erase the current firmware (or whatever data) from your ESP32-cam. Change `--port /dev/ttyUSB` to the port your device is connected to, which you can find via `ls -l /dev/ttyUSB*`. For example, if your device is `/dev/ttyUSB0`, then use `--port /dev/ttyUSB0` in the options of the `esptool.py` utility.
   ```
   esptool.py --port /dev/ttyUSB erase_flash
   ```

6. **Wait until `esptool.py` is done**. Then, press the **reset button on the ESP32-cam**.  Now, check that `/dev/ttyUSB` is available again.

7. Flash the `tasmota32-webcam.bin` webcam server binary and the required Tasmota binaries to the ESP32-cam. (As before, change `--port` before running the command.)
   ```
   esptool.py --chip esp32 --port /dev/ttyUSB --baud 921600 --before default_reset --after hard_reset write_flash -z --flash_mode dout --flash_freq 40m --flash_size detect 0x1000 /opt/Tasmota32/bootloader_dout_40m.bin 0x8000 /opt/Tasmota32/partitions.bin 0xe000 /opt/Tasmota32/boot_app0.bin 0x10000 /opt/Tasmota32/tasmota32-webcam.bin
   ```

8. **Wait until `esptool.py` is done**. Then, **remove the flash mode (GPIO0-GND) jumper** from the ESP32-cam.
   
   [![ESP32cam nonflash mode](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-wiring-nonflash-mode.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-wiring-nonflash-mode.jpg)

9.  Now **press the reset button** on your ESP32-cam.

[top](#){: .btn .btn--light-outline .btn--small}


# Configuration
By default, the Tasmota firmware will create a wireless access point for your ESP32-cam. 

1. Use a wifi-capable device (e.g., laptop) and connect to it. The ESP32-cam will give your device an IP address, which you can check via `ip a`. Usually, the device's IP address is in the `192.168.4.0/24` pool, which means the ESP32-cam webUI is at `192.168.4.1:80`; Otherwise, the webUI will be at the first addr in whichever pool your device connected to after joining the wireless access point created by the Tasmota firmware.

2. Open a web-browser of your choice and navigate to the ESP32-cam webUI. You should be prompted to change the wifi settings to allow your ESP32-cam to connect to your local wifi network.  Change the settings, save it, and wait for the ESP32-cam to reboot.

3. Navigate to the **DHCP server** of your local network and find the IP address assigned to your ESP32-cam.  At this point, it's a good idea to assign a static address to it as well.  (If you set a static address, then reboot the ESP32-cam before moving on.)

4. Navigate to the ESP32-cam webUI on your local network.

## Updating the template
Tasmota templates are device-specific definitions of how their GPIO pins are assigned. As mentioned before, there are multiple ESP32-cam boards out there with different definitions.  In my case, I'm using the **AI-Thinker cam** module and therefore, I should configure the Tasmota32 webcam server to use the [AITHINKER CAM template](https://tasmota.github.io/docs/ESP32/#aithinker-cam) instead of the default one.  (If your ESP32-cam is different, then check [https://tasmota.github.io/docs/ESP32/](https://tasmota.github.io/docs/ESP32/) for the appropriate template and use that one instead of the AITHINKER CAM.)

1. Copy the **AITHINKER CAM template**:
   ```
   {"NAME":"AITHINKER CAM","GPIO":[4992,1,1,1,1,5088,1,1,1,1,1,1,1,1,5089,5090,0,5091,5184,5152,0,5120,5024,5056,0,0,0,0,4928,1,5094,5095,5092,0,0,5093],"FLAG":0,"BASE":1}
   ```

2. From the ESP32-cam webUI, go to **Configuration > Configure > Configure other**.

3. Paste the template under **Other parameters > Template**; **Check Activate**; Save it and wait for the reboot.

4. The device should now be named 'AITHINKER CAM' (or whaterver NAME was in the template).

5. The MJPEG stream should be accessible at `http://DEVICE_IP:81/stream` or `http://DEVICE_IP:81/cam.mjpeg`.

6. A single snapshot can be obtained at `http://DEVICE_IP:80/snapshot.jpg`.

## Auto-enabling the webcam server at boot
If your board is like mine, the stream does not initialize on its own at boot--it requires a request to get webUI to initialize the stream.  This will happen whenever you try to visit the device's webUI.  However, if you want to automatically initialize the webserver and video stream at boot, we can do so using Tasmota's **[rules](https://tasmota.github.io/docs/Rules/)**.  More specifically, we will add `Rule1` that tells the ESP32-cam to start the stream once Tasmota is fully initialized (i.e., after wifi and MQTT are connected, if configured).

1. Copy the following rule:
   ```
   Rule1 ON System#Boot DO WcInit ENDON
   ```

2. Go to the ESP32-cam webUI and then **Console**.

3. Paste the rule in the **enter command** box and press enter.

4. To enable `Rule1`, enter the following command:
   ```
   Rule1 1
   ```

5. Restart the ESP32-cam with the following command:
   ```
   Restart 1
   ```

6. Once it comes back on, check the console if `RULE 1` was executed.  It should show something similar to the following if the rule is working as expected:
   ```
   ... RUL: SYSTEM#BOOT performs "WcInit"
   ... SRC: Rule
   ... CMD: Group 0, Index 1, Command "WCINIT", Data ""
   ... CAM: Stream init
   ... CAM: User template
   ... CAM: PSRAM found
   ... CAM: Initialized
   ... RSL: stat/tasmota_***/RESULT = {"WCInit":"Done"}
   ```

7. The MJPEG stream should now be accessible at `http://DEVICE_IP:81/stream` or `http://DEVICE_IP:81/cam.mjpeg` without ever accessing the webUI's main page.

By the way, **rules** are a great way to program your Tasmota device indepedently of any automation server. Make sure to read about [how to add or modify rules](https://tasmota.github.io/docs/Rules/) and [the list of available rule commands](https://tasmota.github.io/docs/Commands/#rules).

## Webcam server additional configurations
A full list of commands for ESP32 devices can be found at [the official docs page](https://tasmota.github.io/docs/Commands/#esp32).  However, by the time I finished writing this, many of the commands that are specific to the Tasmota32 webcam server binary were gone... I'm not sure what happened there.  For this reason, I've decided to post here all the additional commands (`wc`) that I'm aware of (in alphabetical order):

| Command | Definition | Values |
|:---:|:---:|:---:|
| `WcBrightness` | Image brightness | `-2`, `-1`, `0`, `1`, `2` |
| `WcContrast` | Image contrast | `-2`, `-1`, `0`, `1`, `2` |
| `WCFlip` | Flips the image vertically | `1`, `0` |
| `WcInit` | Initializes the webcam server | |
| `WCMirror` | Flips the image horizontally | `1`, `0` |
| `WcResolution` | Image resolution | `0`: `FRAMESIZE 96x96` |
|  |  | `1`: `FRAMESIZE 160x120` |
|  |  | `2`: `FRAMESIZE 176x144` |
|  |  | `3`: `FRAMESIZE 240x176` |
|  |  | `4`: `FRAMESIZE 240x240` |
|  |  | `5`: `FRAMESIZE 320x240` |
|  |  | `6`: `FRAMESIZE 400x256` |
|  |  | `7`: `FRAMESIZE 480x320` |
|  |  | `8`: `FRAMESIZE 640x480` |
|  |  | `9`: `FRAMESIZE 800x600` |
|  |  | `10`: `FRAMESIZE 1024x768` |
| `WcSaturation` | Image saturation | `-2`, `-1`, `0`, `1`, `2` |
| `WcStream` | Controls the video streaming | `0`: stop, `1`: start |


For example, to set the stream resolution to 800x600, go to the **Console** and enter the following command :
```
WcResolution 9
```

Alternatively, it's possible to send commands via HTTP.  The previous example via web-browser: `http://DEVICE_IP/cm?cmnd=WcResolution%209`.  If using a terminal, you can send via `curl`, as follows:
```
curl http://DEVICE_IP/cm?cmnd=WcResolution%209
```
which should reply with a `json` parsable by utilities such as `jq`.

Finally, the **flash LED** is controlled by **GPIO4** and the **red LED** is controlled by **GPIO33**. Their state can be changed programatically as well.

## Fixing the timezone
If you installed a pre-compilled firmware, there's a chance your device is using the incorrect timezone.  To check the current timezone, go to **Console** and type
```
timezone
```
and to change it, enter the command with a value equal to your region's [standardized time zone](https://upload.wikimedia.org/wikipedia/commons/8/88/World_Time_Zones_Map.png).  For America/Sao_Paulo, for example, that would be `-3`, which can be set in your Tasmota device as follows
```
timezone -3
```

[top](#){: .btn .btn--light-outline .btn--small}


# Basic usage
You can now capture the live stream of your ESP32-cam at either `http://DEVICE_IP:81/stream` or `http://DEVICE_IP:81/cam.mjpeg`, and a single snapshot at `http://DEVICE_IP:80/snapshot.jpg`.  Such URLs can be easily fed into most camera surveillance servers, such as [MotionEye](https://github.com/ccrisan/motioneye/), [Shinobi](https://shinobi.video/), [ZoneMinder](https://www.zoneminder.com/), or [iSpy](https://www.ispyconnect.com/).  As mentioned before, the Tasmota32 webcam server can be configure to connect to a **[MQTT server](https://mqtt.org/)** (see **Configuration** > **Configure MQTT**) and then integrated with most home automation servers, such as [HomeAssistant](https://www.home-assistant.io/), [OpenHAB](https://www.openhab.org/), or one based on [NodeRed](https://nodered.org/)'s flow programming.

## Standalone wiring
If you bought a USB to DIP adapter, you can now power your ESP32-cam independent of your USB to TTL/serial adapter using a cheap **5V (at least 400mA) USB power supply**, such as an old cellphone charger, as follows:

[![ESP32cam standalone mode](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-wiring-standalone-mode.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-wiring-standalone-mode.jpg)

[top](#){: .btn .btn--light-outline .btn--small}


# Conclusion
This concludes the tutorial on how to install and configure the Tasmota32 webcam server onto the ESP32-cam.  As usual, if you spot an error or want to share an idea, feel free to [get in touch with me](/contact).  I try to keep my articles updated as much as possible to reflect my current understanding about the topic.  All such updates are noted in the [changelog](#changelog).

[top](#){: .btn .btn--light-outline .btn--small}


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
**September 1st, 2021**: Fixed a few typos (e.g., `ESP_HOME` instead of `ESP_PORT`) and updated the AITHINKER CAM template in [Updating the template](#updating-the-template).  Also, added minor notes to help troubleshooting issues when flashing the latest firmware.
{: .notice--success }

**August 12th, 2021**, Update #3: Made minor changes to a few commands to improve readability.
{: .notice--info }

**August 12th, 2021**, Update #2: Per a user suggestion (Tobias), the [Flashing Tasmota32 webcam server](#flashing-tasmota32-webcam-server) section has been updated. Specifically, the baud rate in the `esptool-py` utility (`-b`) has been omitted to use the default value (`115200`), which seems to work fine with the ESP32-cam module and most adapters.  However, if you run into issues, try the previous value when flashing the Tasmota32-webcam binaries (`-b 921600`).
{: .notice--info }

**August 12th, 2021**, Update #1: There has been changes to the location of the binary files because they moved from the [Tasmota](https://github.com/arendst/Tasmota) repository to the new [Tasmota-firmware](https://github.com/arendst/Tasmota-firmware) repository, which currently has a single branch (`main`).  The location of the necessary binaries to flash the Tasmota32-webcam firmware via the `esptool.py` utility was changed accordingly in the [Flashing Tasmota32 webcam server](#flashing-tasmota32-webcam-server) section. (It seems that changes are still being made to the organization of such files, so if the URLs do not work, check the new repo directly.)
{: .notice--warning }

**July 16th, 2021**: Updated the `WcResolution` command in the [Webcam server additional configurations](#webcam-server-additional-configurations) section to reflect the latest support (firmware `9.5.0`) for higher resolutions (`11`, `12`, `13`).  Thanks to Eric for the heads up!
{: .notice--info }

**April 6th, 2021**, Update #2: Created a bonus content section at the end called [**Firmware customization**](#bonus-content-firmware-customization). The new section describes how to create a customized Tasmota firmware to use any supported I2C or other peripherals that are not available in the pre-compiled binary. The *BME280* sensor--a cheap and very reliable ambient temperature, humidity, and pressure sensor--was used as an example but the same procedure applies for displays and other I2C sensors that you might wish to use with your ESP32-cam board. This provides a very easy way to turn a simple webcam server into a weather station, smoke detector, relay controller, and more.
{: .notice--info }

**April 6th, 2021**, Update #1: Added a pinout diagram for the ESP32-cam AI-Thinker board to the [Hardware](#hardware) section.
{: .notice--info }

**Jan 26th, 2021**: Added an alternative source for the Tasmota32 binaries to the [Flashing Tasmota32 webcam server](#flashing-tasmota32-webcam-server) section.  I few individuals reported issues flashing the latest (`firmware` branch) binaries, so I added a reference to the more stable (`release-firmware` branch) binaries instead.  A list of currently active branches can be found in the official Github repo's [active branches](https://github.com/arendst/Tasmota/branches/active) website.
{: .notice--info }

**Jan 16th, 2021**: Publication of the original article
{: .notice--info }

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

  [![ESP32cam pinout](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-pinout.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-pinout.jpg)

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
   sudo apt update
   sudo apt install wget python3 python3-pip
   ```

2. Install `esptool.py` via `pip3`:

   ```
   pip3 install esptool
   ```

3. Find out if `esptool.py` can be found in your user's `$PATH`.

   ```
   whereis esptool.py
   ```

   Alternatively, when required to run `esptool.py`, instead of `esptool.py OPTIONS`, run as `python3 -m esptool OPTIONS`. If you choose to do this, skip the next step.
   {: .notice }

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
   sudo usermod -aG dialout,tty ${USER}
   ```

8. Log off and back on.  (If you continue to run into permission issues, try rebooting instead.  You can check your user's permissions with `id ${USER}`.)

## Flashing Tasmota32 webcam server
We are now ready to flash the Tasmota firmware.  For reference, the official information is available at [https://tasmota.github.io/docs/ESP32](https://tasmota.github.io/docs/ESP32).

1. Create a `tasmota32` dir in `/opt`:

   ```
   cd /opt
   sudo mkdir tasmota32
   ```

2. Change ownership of the new directory to the current user instead of `root`:

   ```
   sudo chown ${USER}:${USER} tasmota32/
   ```

3. Download the `tasmota32-webcam.bin` binary and the needed ESP32 Tasmota binaries from the official Github repo via `wget`.  (*The following was updated on August 12th, 2021.*) The binaries are now available in a different repository than [before](https://github.com/arendst/Tasmota), namely [arendst/Tasmota-firmware](https://github.com/arendst/Tasmota-firmware), and currently, the new repository has a single branch (`main`). There are two versions of the `tasmota32-wecam.bin`, one from the `release` and another from the `development` portions of the Tasmota32 project. My advice is to try the release first, then development if you have any issues.
   
   To download the **stable release** binaries, use the following command:

   ```
   wget -P /opt/tasmota32/ \
     https://ota.tasmota.com/tasmota32/release/tasmota32-webcam.bin \
     https://github.com/arendst/Tasmota-firmware/raw/main/static/esp32/boot_app0.bin \
     https://github.com/arendst/Tasmota-firmware/raw/main/static/esp32/bootloader_dout_40m.bin \
     https://github.com/arendst/Tasmota-firmware/raw/main/static/esp32/partitions.bin
   ```

   **Alternatively**, to download the **development** binaries, use the following command:

   ```
   wget -P /opt/tasmota32/ \
     https://ota.tasmota.com/tasmota32/tasmota32-webcam.bin \
     https://github.com/arendst/Tasmota-firmware/raw/main/static/esp32/boot_app0.bin \
     https://github.com/arendst/Tasmota-firmware/raw/main/static/esp32/bootloader_dout_40m.bin \
     https://github.com/arendst/Tasmota-firmware/raw/main/static/esp32/partitions.bin
   ```

4. Make sure your ESP32-cam is connected to your computer in [flash mode](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-wiring-flash-mode.jpg) (GPIO0-GND jumper).  Now find the USB port your device is using in `/dev/` and set it to the environmental variable `ESP_PORT`, as follows:
   
   **Attention.** While convenient, the following command assumes there is a single USB to serial adapter connected to your computer.  If this is not the case, manually set `ESP_PORT` to whichever port your USB adapter is currently using. You can find the port via `ls /dev/ttyUSB*` and testing one by one until you find the one used by the adapter. Alternatively, simply disconnect all other USB to serial adapters for this procedure and continue.
   {: .notice--warning }

   ```
   ESP_PORT=$(ls /dev/ttyUSB*)
   ```

   Please notice that this only works if you continue to use the **same shell** in which `ESP_PORT` was defined.  If you log off or even close the current terminal, you will have to redefine `ESP_PORT` to keep using it.
   {: .notice }

   You can check that `ESP_PORT` was correctly defined by `echo`ing it, as follows:

   ```
   echo $ESP_PORT
   ```

   which should output something like this:

   ```
   /dev/ttyUSB0
   ```

5. Erase the current firmware (or whatever data) from your ESP32-cam. 
   
   **Attention.** The following procedure will **wipe all the data** on the ESP32-cam. 
   {: .notice--warning }

   ```
   esptool.py --port $ESP_PORT erase_flash
   ```

   **Wait** until `esptool.py` is done. Then, press the **reset button on the ESP32-cam**.  Now, check that `$ESP_PORT` is available again.
   {: .notice--danger }

6. Flash the `tasmota32-webcam.bin` webcam server binary and the required Tasmota binaries to the ESP32-cam.

   ```
   esptool.py --chip esp32 \
     --port $ESP_PORT \
     --before default_reset \
     --after hard_reset \
     write_flash -z \
     --flash_mode dout \
     --flash_freq 40m \
     --flash_size detect \
     0x1000 /opt/tasmota32/bootloader_dout_40m.bin \
     0x8000 /opt/tasmota32/partitions.bin \
     0xe000 /opt/tasmota32/boot_app0.bin \
     0x10000 /opt/tasmota32/tasmota32-webcam.bin
   ```

   **Wait** until `esptool.py` is completely done before moving on. Flashing a firmware can take a few minutes to complete.  If you experience issues while flashing, try a different baud rate (`-b`) than the default `115200`, such as `-b 921600`. The [Tasmota FAQ](https://tasmota.github.io/docs/FAQ/#flashing) can help with this and other issues.
   {: .notice--danger }

   If `esptool.py` hangs at `Connecting...`, then press the **Restart** button (`RST`) on your ESP-cam module.
   {: .notice }

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

1. Copy the **AITHINKER CAM template** (*Updated in September 1st, 2021*):

   ```json
   {"NAME":"AITHINKER CAM","GPIO":[4992,1,672,1,416,5088,1,1,1,6720,736,704,1,1,5089,5090,0,5091,5184,5152,0,5120,5024,5056,0,0,0,0,4928,576,5094,5095,5092,0,0,5093],"FLAG":0,"BASE":2}
   ```

2. From the ESP32-cam webUI, go to **Configuration > Configure > Configure other**.

3. Paste the template under **Other parameters > Template**; **Check Activate**; Save it and wait for the reboot.
   
   If you lose connection to the ESP-cam afterwards, it is very likely that the AITHINKER CAM template has changed since the last time this article was updated.  In this case, put the ESP-cam in **flash mode** and flash the Tasmota32-webcam firmware once again.  Then, when updating the **Template**, use the one from [https://tasmota.github.io/docs/ESP32/#aithinker-cam](https://tasmota.github.io/docs/ESP32/#aithinker-cam) instead of the one mentioned before.
   {:.notice}

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
|  |  | `11`: `FRAMESIZE 1280x720` |
|  |  | `12`: `FRAMESIZE 1280x1024` |
|  |  | `13`: `FRAMESIZE 1600x1200` |
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


# Bonus content: Firmware customization
Even though many of the GPIO pins in the ESP32-cam board are used for the built-in camera module, the board certainly has more than enough pins to interface with additional peripherals.  In other words, while you can use your ESP32-cam as a simple webcam server, it is possible--and as we will see, very easy--to turn it into something more than that, such as a weather station, smoke detector, relay controller, and so on, owning to the multitude of peripherals that are currently supported by the Tasmota firmware.  For an up-to-date list, see the [**official Supported Peripherals table**](https://tasmota.github.io/docs/Supported-Peripherals/#supported-peripherals).

However, due to space limitations, support for some peripherals are not included in pre-compilled binaries.  In the official docs, for example, it says that support for the [**BME280 sensor module**](https://tasmota.github.io/docs/BME280/) is only available in the `tasmota-sensors.bin` pre-compiled binary.  Fortunately, it is now very easy to customize the `tasmota32-webcam.bin` to support the BME280 and any other supported peripherals.

[![BME280](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/bme280.jpg){:.PostImage}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/bme280.jpg)

In the following sections, I described how to customize the Tasmota32 firmware to support a BME280 sensor module.  This was accomplished with [**Docker**](https://www.docker.com/) and the container [**TasmoCompiler**](https://github.com/benzino77/tasmocompiler).  At the end, I showed how to update the firmware over-the-air and how to configure the template to interface with peripherals connected to GPIO pins.

There are [many different ways of customizing a Tasmota firmware](https://tasmota.github.io/docs/Compile-your-build/).  TasmoCompiler is just one of them that does not use an IDE and has a user-friendly GUI.
{: .notice .notice--info }

## Installing Docker and running TasmoCompiler
**Docker** is a ver well-known, documented, and used virtualization platform. To install Docker, follow the official documentation:

* [**Get Docker** and **install it** on your OS](https://docs.docker.com/get-docker/)

Once you have Docker up and running, it is time to pull and run the **TasmoCompiler** container. TasmoCompiler was developed by user [benzino77](https://github.com/benzino77) to do only one thing, namely compile a Tasmota firmware with customized settings via a simple (web) GUI.  To run it in a Docker container, **open a terminal** and pull the image, as follows:

```
docker pull benzino77/tasmocompiler
```

[![tasmocompiler pull](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-pull.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-pull.jpg)

If you run into permission issues, either append `sudo` to any docker command or create and add your current user to a `docker` group, as follows: `sudo groupadd docker && sudo usermod -aG docker $USER`. Other post-install configurations for Linux users can be found at the official docs: [Optional post-install steps](https://docs.docker.com/engine/install/linux-postinstall/).
{: .notice .notice--info }

Then, run the `tasmocompiler` container, as follows:

```
docker run \
  --rm \
  --name tasmocompiler \
  -p 3000:3000 \
  -e DEBUG=server,git,compile \
  benzino77/tasmocompiler
```

[![tasmocompiler run](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-run.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-run.jpg)

Of course, if you use [Portainer](https://www.portainer.io/) or other application for managing your docker containers, you can also pull and run `tasmocompiler` via the application instead of a terminal.  In this case, translate the commands to your application.  This also applies to users who are not running Docker on Linux.
{: .notice .notice--info }

This will create a container named `tasmocompiler` that has a web GUI exposed on port `3000` of the local machine.  To access it, go to the following address using any web-browser: 

* [**http://localhost:3000**](http://localhost:3000)

## Customizing the tasmota32-webcam firmware
Now that the `tasmocompiler` container is running, we can compile a new customized Tasmota firmware for the ESP32-cam in just a few simple steps:

1. Open any web-browser and navigate to [**http://localhost:3000**](http://localhost:3000);

2. In **Tasmota source code**, select *Refresh Source* (this can take a few minutes, depending on your connection) and afterwards, *Next*; 
   
   [![tasmocompiler step 01](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step01.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step01.jpg)

3. In **WiFi configuration**, add your wifi credentials and hit *Next*;
   
   [![tasmocompiler step 02](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step02.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step02.jpg)

4. In **Select Features**, select **ESP32 webcam** as your board.  For this example, we are adding the **BME280 sensor module** and therefore, in feature, we add the *Temp/Hum sensors* feature to support the BME280 sensor. If you are attaching another device, check the appropriate feature to support it here (e.g., check *Displays (I2C/SPI)* to support an OLED display module).  When done, hit *Next*;
   
   [![tasmocompiler step 03](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step03.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step03.jpg)

5. It is not necessary to edit any parameter in **Custom Parameters**, so hit *Next*;

6. Finally, in **Select Version and Compile**, choose a Tasmota version (*development* is usually fine but if you run into issues later on, try the latest stable) and base language for the interface.  Then, select **Compile** and wait until it is done (this can take a few minutes);

   [![tasmocompiler step 05](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step05.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step05.jpg)

   [![tasmocompiler step 06](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step06.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step06.jpg)

7. Once it is done compiling, check that the firmware was successfully compiled and if all looks good, **download the firmware** (and optionally, any of the other files);

   [![tasmocompiler step 07](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step07.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/tasmocompiler-step07.jpg)

If you run into issues, check the [issues tab](https://github.com/benzino77/tasmocompiler/issues) of the TasmoCompiler repository for open and closed issues similar to the one you are experiencing. If you do not find anything similar, open a new issue there to warn the developer know about it.

## Updating the firmware
If you have already flashed a pre-compiled Tasmota binary onto the ESP32-cam, then it is possible to update the firmware over-the-air (OTA).  To update the firmware OTA, do the following:

1. Open a web-browser and go to the IP address of your Tasmota ESP32-cam;

2. Then navigate to **Firmware Upgrade** > **Upgrade by file upload** > Browse and select the `firmware.bin` file you compiled with TasmoCompiler.  Afterwards, select **Start Upgrade** and wait until it is done;

3. The device will reboot automatically and once it is back on, it should connect to the wireless network configured with TasmoCompiler.

Now, if you have not flashed any pre-compiled Tasmota binary, simply switch the `tasmota32-webcam.bin` file mentioned in [Flashing Tasmota32 webcam server](#flashing-tasmota32-webcam-server) for the `firmware.bin` file you compiled with TasmoCompiler.

## Wiring and template configuration
Suppose we have a **BME280 sensor module** wired to an ESP32-cam (AI-Thinker) board as follows:

[![Wiring BME280](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-wiring-bme280.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-wiring-bme280.jpg)

Note that to use the VCC output pin as a 3.3V output pin, you need to make sure the board has a **resistor connecting the two 3v3 pads** and nothing connecting the two 5V pads. This was indicated by the red arrow in the image above. From my experience, a resistor connecting the 3v3 pads is the default for all ESP32-cam AI-Thinker boards, meaning that it should output 3.3V by default and if you want to change it to 5V, you need to desolder the resistor between the 3v3 pads and solder it between the 5V pads.  However, make sure to double check this before wiring any peripheral that will use the VCC out pin from the board.
{:.notice .notice--warning}

Then, now that the board is running a customized firmware that should support the BME280 sensor module, all that we need to do is to configure its template to inform the program about (a) which GPIO pins the peripheral is using and (b) how the pins should be configured.

To configure the ESP32-cam template, do the following:

1. Open a web-browser and go to the IP address of your Tasmota ESP32-cam;

2. Follow the instructions in [**Updating the template**](#updating-the-template) if you have not done that before. Afterwards, navigate to **Configuration** > **Configure Other** > **Other parameters** > **Template** and make sure the **Activate** is checked. 

3. Navigate to **Configuration** > **Configure Template**.  The name of the template should be the same one you specified in the previous step.  Remember that according to the wiring of the BME280 board, **SDA** and **SCL** are connected to pins **GPIO14** and **GPIO15**, respectively.  Therefore, **find the GPIO14 pin** and instead of `User`, select `I2C SDA`; and similarly, **find the GPIO15 pin** and instead of `User`, select `I2C SCL`.
   
   [![BME280 template](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-template-bme280.jpg){:.PostImage}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-template-bme280.jpg)

4. Hit *Save* and wait for the device to reboot. Once it comes back on, the firmware should automatically detect and configure the I2C device and on the **Main Page**, there should be some of the metrics associated with the device.  Because we are connecting the board to a BME280 sensor module, the Main page will show measures for the ambient temperature, humidity, dew point, and pressure.
   
   [![ESP32-cam BME280](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-bme280.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-01-15-Esp32cam-tasmota-webcam-server/esp32cam-bme280.jpg)

   Of course, different peripherals will show different metrics, buttons, sliders, etc., on the main page. As before, the camera stream should be available on the main page and via port `81` at `/stream` and `/cam.mjpeg`.

[top](#){: .btn .btn--light-outline .btn--small}


# Conclusion
This concludes the tutorial on how to install and configure the Tasmota32 webcam server onto the ESP32-cam.  As usual, if you spot an error or want to share an idea, feel free to [get in touch with me](/contact).  I try to keep my articles updated as much as possible to reflect my current understanding about the topic.  All such updates are noted in the [changelog](#changelog).

[top](#){: .btn .btn--light-outline .btn--small}


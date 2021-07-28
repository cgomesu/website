---
title: "DIY series: Cheap, reliable, low-profile, USB-powered Tasmota ambient sensor"
date: 2021-07-18 11:20:00 -0300
tags: diy-series tasmota sensor hass iot automation temperature
header:
  overlay_image: "/assets/posts/2021-07-18-diy-tasmota-bme280/header.jpg"
  overlay_filter: "0.6"
toc: true
toc_label: "Table of Contents"
toc_icon: "list"
---

# Changelog
**July 18th, 2021**: Publication of the original article
{:.notice--info }

[top](#){:.btn .btn--light-outline .btn--small}


# Introduction
This is the first article of a **Do It Yourself** (DIY) series in which I describe simple electronic projects that make use of an [ESP8266](https://www.espressif.com/en/products/socs/esp8266)/[ESP32](https://www.espressif.com/en/products/socs/esp32) board running the [Tasmota](https://github.com/arendst/Tasmota) firmware to integrate various modules into a home automation system, such as [Home Assistant](https://www.home-assistant.io/).  In this first iteration of the series, I described how to wire and configure a [**BME280**](https://www.bosch-sensortec.com/products/environmental-sensors/humidity-sensors-bme280/) to the tiny [**ESP-01**](http://www.ai-thinker.com/pro_view-60.html) (or its successor, the [ESP-01S](http://www.ai-thinker.com/pro_view-88.html)) to create a cheap, low-power, and low-profile ambient sensor that provides **temperature**, **humidity**, and **relative pressure** measurements to a Home Assistant instance.

- Here is a preview of the ambient sensor standalone and attached to a different devices:
  
  [![ESP-BME280 sensor 01](/assets/posts/2021-07-18-diy-tasmota-bme280/esp-bme280-sensor-01.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-07-18-diy-tasmota-bme280/esp-bme280-sensor-01.jpg)
  
  [![ESP-BME280 sensor 02](/assets/posts/2021-07-18-diy-tasmota-bme280/esp-bme280-sensor-02.jpg){:.PostImage}](/assets/posts/2021-07-18-diy-tasmota-bme280/esp-bme280-sensor-02.jpg)

  [![ESP-BME280 sensor 03](/assets/posts/2021-07-18-diy-tasmota-bme280/esp-bme280-sensor-03.jpg){:.PostImage}](/assets/posts/2021-07-18-diy-tasmota-bme280/esp-bme280-sensor-03.jpg)

This is a great project for anyone who wants to get started on building their own Internet of Things (IoT) devices.  The article started with my motivation for this particular project.  Next, I talked about the hardware and software components, and finally, at the end, I covered the assembly of it all to create a functional unit.

Because this is such a low-profile project, I do not ever bother printing a case for it.  However, if you designed a case to house this particular project, [get in touch with me](/contact) and I will feature your case here.

[top](#){:.btn .btn--light-outline .btn--small}

# Motivation
Over the years, I have started noticing that multiple devices spread across many households (e.g., smart TVs, sound systems, wireless routers, PC towers and laptops) had one or more **USB ports** that could be used to power a few DIY electronic projects.  

[![Device with USB port 01](/assets/posts/2021-07-18-diy-tasmota-bme280/device-usb-port-01.jpg){:.PostImage}](/assets/posts/2021-07-18-diy-tasmota-bme280/device-usb-port-01.jpg)

[![Device with USB port 02](/assets/posts/2021-07-18-diy-tasmota-bme280/device-usb-port-02.jpg){:.PostImage}](/assets/posts/2021-07-18-diy-tasmota-bme280/device-usb-port-02.jpg)

However, the most common type of USB port available ([USB 2.0](https://en.wikipedia.org/wiki/USB#USB_2.0)) usually delivers a maximum of *5V* at *500mA*, which constraints the type of projects that could reasonably use such USB ports as power supply. In addition, because interfacing via the USB connection might not always be possible, due to proprietary and closed-source firmware, the DIY project should be able to transmit data wirelessly instead.

Fortunately, the **ESP-01 WiFi module** meets all such requirements. Specifically, it requires very little energy to operate safely (roughly 3.3V and at least 300mA) and can be connected to existing USB adapters that have a built-in voltage regulator.

[top](#){:.btn .btn--light-outline .btn--small}


# Overview of the ESP-01
The **ESP-01** module is a **cheap** and very **small** WiFi module based on the *ESP8266* microcontroller by [ESPRESSIF](#). Of note, it **exposes only two GPIO pins** (other than serial RX and TX) to interface with other devices, it is powered via **3v3 DC** to the VCC and Ground pins, and has **very limited flash memory**.  Regarding the latter point, there are actually two versions of the ESP-01 module that differ in flash memory size:

1. **ESP-01**: The original version with **512KB** of flash memory;
2. **ESP-01S**: A revised version with **1MB** of flash memory.

Fortunately, visual inspection of the module can easily indicate which version it is ([image source](https://www.esp8266.com/viewtopic.php?p=78465#p78465)):

[![ESP-01 comparison](/assets/posts/2021-07-18-diy-tasmota-bme280/esp01-version-comparison.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-07-18-diy-tasmota-bme280/esp01-version-comparison.jpg)


# TODO
1. ~~Overview of the ESP01~~
2. ~~Motivation for making the sensor~~
3. Hardware
   1. ESP01 (add pinout)
   2. BME280
   3. USB adapter for the ESP01 (add pinout)
   4. Female-female jumper wires
   5. Soldering stuff for the BME280 module
4. Software
   1. Tasmota and caution about ESP01 storage restrictions
   2. Specific binary for I2C sensors
5. Assembly
   1. Flashing + wiring (grounding GPIO0)
   2. Wiring for BME280 modules
   3. Tasmota configuration for I2C
6. Home Assistant Integration
   1. Auto
   2. Manual
7. Conclusion
8. Review article

[top](#){:.btn .btn--light-outline .btn--small}

---
title: "DIY series: Cheap, reliable, low-profile, USB-powered ambient sensor"
date: 2021-07-18 11:20:00 -0300
tags: diy tasmota sensor hass iot automation
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
This is the first article of a *do it yourself* (DIY) series in which I describe simple electronic projects that make use of an [ESP32](#) board running the [Tasmota](#) firmware to integrate various modules into an existing home automation system, such as [Home Assistant](#).  In this first iteration of the series, I described how to configure and connect a tiny [**ESP-01**](#) to a [**BME280**](#) to create a cheap, low-power, and low-profile ambient sensor that provides **temperature**, **humidity**, and **relative pressure** measurements to a Home Assistant instance.

Here is a preview of the ambient sensor:

*Insert preview image*

The article was organized in a way similar to my previous ones in which I detailed an electronics project. More specifically, it started with a list of the hardware and software components, then at the end, I covered the assembly of it all to create a functional unit.

[top](#){:.btn .btn--light-outline .btn--small}


# TODO
1. Overview of the ESP01
2. Motivation for making the sensor
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

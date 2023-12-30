---
title: Projects
layout: splash
permalink: /projects/
header:
  overlay_color: "#000"
  overlay_filter: "0.7"
  overlay_image: /assets/img/site/banner-page-02.jpeg
  caption: "by [**Frank Frazetta**](https://en.wikipedia.org/wiki/Frank_Frazetta)"
  actions:
    - label: "mcu"
      url: "/projects/#mcu"
    - label: "sbc"
      url: "/projects/#sbc"
    - label: "networking"
      url: "/projects/#networking"
    - label: "other"
      url: "/projects/#other"
excerpt: "A non-exhaustive list of past and current projects."
mcu:
  - title: "ESP-01 ambient sensor"
    excerpt: "The project describes the assembly of a cheap (<$6), low-power (<1W), and low-profile (<5cm) environmental sensor based on the ESP-01 and BME280. The unit runs a free and open source firmware (Tasmota) and provides temperature, humidity, and relative pressure measurements to home automation systems via the MQTT messaging protocol."
    image_path: "/assets/img/projects/thumb-projects-esp-01-bme280.jpg"
    url: "/blog/diy-tasmota-bme280/"
    btn_label: "Read more"
    btn_class: "btn--info"
  - title: "ESP32-cam"
    excerpt: "The ESP32 is a low-cost and low-power microcontroller developed by Espressif. This project describes how to flash the Tasmota32 webcam server firmware onto the ESP32-cam. The Tasmota32 firmware can be used as an alternative to the Arduino sketch for users looking for more options to monitor and control an ESP32-cam remotely or to integrate into an existing home automation server."
    image_path: "/assets/img/projects/thumb-projects-esp32cam-tasmota.jpg"
    url: "/blog/Esp32cam-tasmota-webcam-server/"
    btn_label: "Read more"
    btn_class: "btn--info"
sbc:
  - title: "NanoPi M4 mini-NAS"
    excerpt: "A cheap, low-power, low-profile NAS solution based on FriendlyArm's NanoPi M4 SBC. It has a SATA hat that is connected to the board via PCI-e, allowing up to four HDDs to be connected to the NAS via standard SATA III interface."
    image_path: "/assets/img/projects/thumb-projects-mininas.jpg"
    url: "/blog/Nanopi-m4-mini-nas/"
    btn_label: "Read more"
    btn_class: "btn--info"
  - title: "PWM fan controller"
    excerpt: "A fan controller written in Bash for the 12v fan connector of the NanoPi M4 SATA hat. By default, the script uses either a bounded logistic model with a moving mid-point or a proportional-integral-derivative (PID) controller to set the fan speed dynamically."
    image_path: "/assets/img/projects/thumb-projects-logistic-pwm-fan.jpg"
    url: "https://github.com/cgomesu/nanopim4-satahat-fan"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "RPi SenseHAT MQTT"
    excerpt: "A Python application for the RPi that allows interfacing with the SenseHAT over MQTT. The script publishes sensor and joystick data to the MQTT broker to be consumed by a home automation server. It also subcribes to an LED topic to display payloads published to the broker."
    image_path: "/assets/img/projects/thumb-projects-rpi-sensehat-mqtt.jpg"
    url: "https://github.com/cgomesu/rpi-sensehat-mqtt"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "RPi button box"
    excerpt: "The project describes how to repurpose external HDD enclosures into button boxes for the Raspberry Pi and similar single board computers.  It includes the development of a button box controller, wiring schematics, how-tos, and list of hardware and software components."
    image_path: "/assets/img/projects/thumb-projects-button-box.jpg"
    url: "/blog/Rpi-button-box-ehdd-enclosure/"
    btn_label: "Read more"
    btn_class: "btn--info"
  - title: "Button box controller"
    excerpt: "Core program for a Raspberry Pi button box controller that uses the gpiozero Python library to interface with the wired buttons and switches."
    image_path: "/assets/img/projects/thumb-projects-rpi-button-box.jpg"
    url: "https://github.com/cgomesu/rpi-button-box"
    btn_label: "Github Repo"
    btn_class: "btn--info"
networking:
  - title: "Wireless Mesh"
    excerpt: "Wireless mesh networks using free and open-source software (FOSS) and common hardware.  The project is based on OpenWrt and uses the layer-2 implementation of the Better Approach to Mobile Adhoc Networking (batman-adv) to route packets over multiple mesh topologies."
    image_path: "/assets/img/projects/thumb-projects-mesh.jpg"
    url: "/blog/Mesh-networking-openwrt-batman/"
    btn_label: "Read more"
    btn_class: "btn--info"
  - title: "pfSense box"
    excerpt: "A cheap, customizable pfSense box based on the ASRock J3355b-itx mobo. It features an Intel dual-core processor capable of running IPS/IDS software, multiple VPNs, and more. A passively cooled, low-power, and low-profile firewall that fits the demands of most home users."
    image_path: "/assets/img/projects/thumb-projects-pfsense-box.jpg"
    url: "/blog/Pfsense-white-box/"
    btn_label: "Read more"
    btn_class: "btn--info"
other:
  - title: "Programa Bolsas"
    excerpt: "A CLI program that validates a csv file containing scholarship data from multiple students and allows users to perform a few operations. This program was part of a selection process and most of its content is in Portuguese."
    image_path: "/assets/img/projects/thumb-projects-bolsas.jpg"
    url: "https://github.com/cgomesu/programa-bolsas"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "Pife (pif-paf)"
    excerpt: "This is a CLI implementation of the card game Pife written in Java. It is a fork of another implementation that changes its main logic to adapt to the new game but preserves overlapping classes, such as deck and player. This program is only for educational purposes and the content is in Portuguese."
    image_path: "/assets/img/projects/thumb-projects-pife.jpg"
    url: "https://github.com/cgomesu/Pife"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "Logistic function in Bash"
    excerpt: "An implementation of the logistic function written in GNU bash and GNU basic calculator (bc)."
    image_path: "/assets/img/projects/thumb-projects-bash-logistic.jpg"
    url: "https://github.com/cgomesu/bash-logistic-function"
    btn_label: "Github Repo"
    btn_class: "btn--info"
---
***

# MCU

{% include feature_row id="mcu" %}
[top](#){: .btn .btn--light-outline .btn--small}

# SBC

{% include feature_row id="sbc" %}
[top](#){: .btn .btn--light-outline .btn--small}

# Networking

{% include feature_row id="networking" %}
[top](#){: .btn .btn--light-outline .btn--small}

# Other

{% include feature_row id="other" %}
[top](#){: .btn .btn--light-outline .btn--small}

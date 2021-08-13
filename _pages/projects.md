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
    - label: "IT"
      url: "/projects/#information-technology"
    - label: "Programming"
      url: "/projects/#programming"
    - label: "Academic (latest)"
      url: "/projects/#academic-latest"
excerpt: "A selected list of past and current projects."
it:
  - title: "ESP-01 ambient sensor"
    excerpt: "The project describes the assembly of a cheap (less than USD$6), low-power (less than 1W), and low-profile (less than 5cm long) environmental sensor based on the ESP-01 WiFi module and the BME280. The unit runs a free and open source firmware (Tasmota) and provides temperature, humidity, and relative pressure measurements to home automation systems via the MQTT messaging protocol."
    image_path: "/assets/img/projects/thumb-projects-esp-01-bme280.jpg"
    url: "/blog/diy-tasmota-bme280/"
    btn_label: "Read more"
    btn_class: "btn--info"
  - title: "ESP32-cam"
    excerpt: "The ESP32 is a low-cost and low-power microcontroller developed by Espressif. This project describes how to flash the Tasmota32 webcam server (beta) firmware onto the ESP32-cam module with onboard Wi-Fi. The Tasmota32 firmware can be used as an alternative to the Espressif CamWebServer Arduino sketch for users looking for more options to monitor and control an ESP32-cam remotely or to integrate into an existing home automation server via HTTP requests or MQTT."
    image_path: "/assets/img/projects/thumb-projects-esp32cam-tasmota.jpg"
    url: "/blog/Esp32cam-tasmota-webcam-server/"
    btn_label: "Read more"
    btn_class: "btn--info"
  - title: "Mesh networking"
    excerpt: "Mesh networks using free and open-source software (FOSS) and common hardware.  The project is based on OpenWrt and uses the layer-2 implementation of the Better Approach to Mobile Adhoc Networking (B.A.T.M.A.N.), called batman-adv, to route packets over multiple mesh topologies."
    image_path: "/assets/img/projects/thumb-projects-mesh.jpg"
    url: "/blog/Mesh-networking-openwrt-batman/"
    btn_label: "Read more"
    btn_class: "btn--info"
  - title: "NanoPi M4 mini-NAS"
    excerpt: "A cheap, low-power, low-profile NAS solution based on FriendlyArm's NanoPi M4 SBC. It has a SATA hat that is connected to the board via PCI-e, allowing up to four HDDs to be connected to the NAS via standard SATA III interface."
    image_path: "/assets/img/projects/thumb-projects-mininas.jpg"
    url: "/blog/Nanopi-m4-mini-nas/"
    btn_label: "Read more"
    btn_class: "btn--info"
  - title: "RPi button box"
    excerpt: "The project describes how to repurpose external HDD enclosures into button boxes for the Raspberry Pi and similar single board computers.  It includes the development of a button box controller, wiring schematics, how-tos, and list of hardware and software components."
    image_path: "/assets/img/projects/thumb-projects-button-box.jpg"
    url: "/blog/Rpi-button-box-ehdd-enclosure/"
    btn_label: "Read more"
    btn_class: "btn--info"
  - title: "A chácara microserver"
    excerpt: "The microserver is composed of an HP Proliant Microserver Gen8 running Openmediavault and a Dell Optiplex 3060 micro running Proxmox. Both machines are used mostly for security-related things (surveillance cameras, location tracking) and Plex and Plex-related programs (PMS, Tautulli, Sonarr, Radarr, etc.). Other uses have to do with a universal desktop folder (Syncthing) and data analysis (R server)."
    image_path: "/assets/img/projects/thumb-projects-chacara-microserver.jpg"
    # url: ""
    btn_label: "Read more"
    btn_class: "btn--info"
  - title: "pfSense white-box"
    excerpt: "A cheap, customizable pfSense box based on the ASRock J3355b-itx mobo. It features an Intel dual-core processor capable of running IPS/IDS software, multiple VPNs, and more. A passively cooled, low-power, and low-profile firewall that fits the demands of most home users."
    image_path: "/assets/img/projects/thumb-projects-pfsense-box.jpg"
    url: "/blog/Pfsense-white-box/"
    btn_label: "Read more"
    btn_class: "btn--info"
programming:
  - title: "Button box controller"
    excerpt: "Core program for a Raspberry Pi button box controller that uses the gpiozero Python library."
    image_path: "/assets/img/projects/thumb-projects-rpi-button-box.jpg"
    url: "https://github.com/cgomesu/rpi-button-box"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "Youtube4tvh"
    excerpt: "Youtube4tvh is a Python CLI program for TVHeadend servers that uses either the Youtube API or a custom-built web content parser to find live-streams and create (or update) m3u playlists. The m3u file follows IPTV conventions that allow a TVH server to automatically create an IPTV network with them, and each stream is piped into TVH via a Streamlink shell script."
    image_path: "/assets/img/projects/thumb-projects-youtube4tvh.gif"
    url: "https://github.com/cgomesu/youtube4tvh"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "NetMonitor LCD"
    excerpt: "Python script for a 16x2 LCD that uses ping and netcat to monitor the network status of hosts and services, respectively."
    image_path: "/assets/img/projects/thumb-projects-netmonitor.gif"
    url: "https://github.com/the-raspberry-pi-guy/lcd#netmonitor"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "PWM fan controller"
    excerpt: "A fan controller written in bash for the 12v fan connector of the NanoPi M4 SATA hat. By default, the script uses a bounded logistic model with a moving mid-point (based on the distance between the average temperature over time and a critical temperature threshold) to set the fan speed dynamically."
    image_path: "/assets/img/projects/thumb-projects-logistic-pwm-fan.jpg"
    url: "https://github.com/cgomesu/nanopim4-satahat-fan"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "Audio diagnostic tool"
    excerpt: "This is a CLI diagnostic tool for audio files written in GNU Bash that extends my previous bash-flac-diag tool to mp3 and other audio formats. In brief, it tests a single or multiple audio files and generates logs with good files (no errors found) and bad ones (at least one error found). Tests are performed by codec-specific tools. There are two post-processing modes for bad files: fix or delete."
    image_path: "/assets/img/projects/thumb-projects-adt-demo.gif"
    url: "https://github.com/cgomesu/audio-diagnostic-tool"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "Logistic function in Bash"
    excerpt: "An implementation of the logistic function written in GNU bash and GNU basic calculator (bc)."
    image_path: "/assets/img/projects/thumb-projects-bash-logistic.jpg"
    url: "https://github.com/cgomesu/bash-logistic-function"
    btn_label: "Github Repo"
    btn_class: "btn--info"    
  - title: "Multi-trial data count"
    excerpt: "A VBA macro for MS Excel that counts patterns of correct responses (C) and errors (E) across multiple trials (e.g., CCC, CCE, CEC, ..., EEE). It was initially developed as a tool to help with the data analysis of multi-trial memory experiments in which subjects provide boolean-type responses (yes/no, correct/error) for multiple items across multiple tests. In such designs, each item generates a pattern of C-E responses across tests. This VBA macro counts all such patterns across subjects and items."
    image_path: "/assets/img/projects/thumb-projects-multitrial-datacount.jpg"
    url: "https://github.com/cgomesu/multitrial-datacount"
    btn_label: "Github Repo"
    btn_class: "btn--info"
academic:
  - title: "Multinomial modeling"
    excerpt: "Distribution of educational materials to researchers and students about the use of multinomial modeling in psychological research."
    image_path: "/assets/img/site/thumb-academic.jpg"
    url: "https://doi.org/10.34019/1982-1247.2020.v14.29542"
    btn_label: "Paper (pt-br)"
    btn_class: "btn--info"
  - title: "Bivariate recollection"
    excerpt: "A psychological theory that assumes that memory probes can provoke conscious awareness of either target items (target recollection) or their context (context recollection) or both."
    image_path: "/assets/img/site/thumb-academic.jpg"
    url: "http://dx.doi.org/10.1037/a0037668"
    btn_label: "Paper (en)"
    btn_class: "btn--info"
---
***

# Information Technology

{% include feature_row id="it" %}
[top](#){: .btn .btn--light-outline .btn--small}

# Programming

{% include feature_row id="programming" %}
[top](#){: .btn .btn--light-outline .btn--small}

# Academic (latest)

{% include feature_row id="academic" %}
[top](#){: .btn .btn--light-outline .btn--small}

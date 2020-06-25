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
  - title: "NanoPi M4 mini-NAS"
    excerpt: "A cheap, low-power, low-profile NAS solution based on FriendlyArm's NanoPi M4 SBC. It has a SATA hat that is connected to the board via PCI-e, allowing up to four HDDs to be connected to the NAS via standard SATA III interface."
    image_path: "/assets/img/projects/thumb-projects-mininas.jpg"
    # url: "/blog/"
    btn_label: "Read the blog post"
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
  - title: "Simple RPi GPIO buttons"
    excerpt: "Schematic and a simple python script to run customized commands using three push buttons connected to a Raspberry Pi 3B's general purpose input/output (GPIO) pins. Compatible with RPi 3B+ and any other RPi that has the same GPIO version as the 3B."
    image_path: "/assets/img/projects/thumb-projects-gpiobuttons.jpg"
    url: "https://github.com/cgomesu/rpi-buttons"
    btn_label: "Github Repo"
    btn_class: "btn--info"
programming:
  - title: "Youtube4tvh"
    excerpt: "Youtube4tvh is a Python CLI program that uses Youtube API to find live-streams and create (or update) m3u playlists for a TVHeadend server. The m3u file follows IPTV conventions that allow a TVH server to automatically create an IPTV network with them, and each stream is piped into TVH via a Streamlink shell script."
    image_path: "/assets/img/projects/thumb-projects-youtube4tvh.jpg"
    url: "https://github.com/cgomesu/youtube4tvh"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "Multi-trial data count"
    excerpt: "A VBA macro for MS Excel that counts patterns of correct responses (C) and errors (E) across multiple trials (e.g., CCC, CCE, CEC, ..., EEE). It was initially developed as a tool to help with the data analysis of multi-trial memory experiments in which one or more subjects provide boolean-type responses (yes/no, correct/error) for multiple items across multiple tests. In such designs, each item will generate a pattern of C-E responses across tests. This VBA macro counts all such patterns across subjects and items."
    image_path: "/assets/img/projects/thumb-projects-multitrial-datacount.jpg"
    url: "https://github.com/cgomesu/multitrial-datacount"
    btn_label: "Github Repo"
    btn_class: "btn--info"
  - title: "Webgrab EPG parser"
    excerpt: "A Python CLI program that converts the WebGrab+Plus siteini.pack/ folder data into a SQLite db and runs various tasks that optimize extraction of EPG data from the available sources. (This project's repo is private and under construction.)"
    image_path: "/assets/img/site/thumb-construction.jpg"
    # url: ""
    btn_label: "Github Repo"
    btn_class: "btn--info"
academic:
  - title: "Multinomial modeling"
    excerpt: "Distribution of educational materials to researchers and students about the use of multinomial modeling in psychological research."
    image_path: "/assets/img/site/thumb-academic.jpg"
  - title: "Conjoint-source recognition"
    excerpt: "A series of experiments using a new theory-driven paradigm that combines the conjoint recognition procedure with a source task, called conjoint-source recognition, in order to extract measures of the two recollective processes posited by the dual-recollection theory, namely target and context recollection."
    image_path: "/assets/img/site/thumb-academic.jpg"
  - title: "Episodic over-distribution (EOD) in recall"
    excerpt: "Secondary analysis of published data and new experiments that incorporate theory-driven manipulations to identify the mechanisms that control EOD in recall."
    image_path: "/assets/img/site/thumb-academic.jpg"
  - title: "Memory, Aging, and Cognitive Impairment Study (MACIS)"
    excerpt: "The MACIS is a longitudinal study conducted by the Memory & Neuroscience research lab at Cornell University. Healthy younger and older adults receive batteries of neuropsychological tests and associative memory tasks that allow us to extract measures of retrieval processes in recall, namely direct access, reconstruction, and familiarity judgment. The purpose of the project is to pinpoint the retrieval processes that are affected/spared by healthy aging and cognitive impairment, and their relation with traditional markers of impairment."
    image_path: "/assets/img/site/thumb-academic.jpg"
  - title: "Validity tests of dual-retrieval Markov models of recall"
    excerpt: "Development and phenomenological validity tests of a family of two-stage, finite Markov chains that are applied to multi-trial recall designs to obtain simple measures of recollective retrieval (direct access) and nonrecollective retrieval (reconstruction and familiarity judgment) in recall. Model-based estimates are compared to traditional methods of estimating dual processes, such as remember/know judgments, confidence ratings, and source judgments."
    image_path: "/assets/img/site/thumb-academic.jpg"
  - title: "The ontogenesis of dual processes"
    excerpt: "Counterintuitive effects of memory development and their implications to the law and dual-process theories."
    image_path: "/assets/img/site/thumb-academic.jpg"
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

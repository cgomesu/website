---
title: "Towards a smarter Home Assistant: Getting started on dynamic statistical analysis of devices, sensors, and services"
date: 2021-06-04 10:40:00 -0300
tags: hass iot automation math stats
header:
  overlay_image: "/assets/posts/2021-06-04-smarter-hass/header.jpg"
  overlay_filter: "0.6"
toc: true
toc_label: "Table of Contents"
toc_icon: "list"
---

# Changelog
**June 4th, 2021**: Publication of the original article
{: .notice .notice--info }

[top](#){: .btn .btn--light-outline .btn--small}


# Introduction
[Home Assistant](https://www.home-assistant.io/) (HASS) is a free and open-source software (FOSS) that provides a feature-rich environment for managing, controlling, and automating smart home devices, such as light bulbs, blinders, and LED strips.  In addition, it provides a highly customizable system for collecting and organizing a multitude of data (e.g., on/off device states, local temperature, GPS tracking information, exchange rates), as provided by **more than a thousand [integrations](https://www.home-assistant.io/integrations)** with local [Internet of things](https://en.wikipedia.org/wiki/Internet_of_things) (IoT) devices and sensors (e.g., micro-controllers or single-board computers connected to sensor modules) and cloud-based services (e.g., weather and financial web APIs).

[![HASS demo frontend](/assets/posts/2021-06-04-smarter-hass/hass-demo.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/hass-demo.jpg)

More often than not, people use HASS to view or modify the **current state and value** of integrated devices and sensors via manual triggers (e.g., pressing a button to turn off the AC) or automations (e.g., if the temperature is lower than 14°C, then turn on the heat).  However, the [HASS utility integrations](https://www.home-assistant.io/integrations#utility) offer users the possibility to go (way) beyond with the help of built-in mathematical and statistical tools.  More specifically, such **analytical tools** allow users to summarize past states and measurements to answer questions such as:

- How many times has the front door been opened over the last 24hrs and for how long?
- How much energy (kWh) has my uninterrupted power supply used over the last month?
- What was the average temperature in the living-room during yesterday's morning, afternoon, and evening?  What about last week?
- What is the average level of volatile organic compounds (VOC) measured by the [BME680 sensor](https://www.bosch-sensortec.com/products/environmental-sensors/gas-sensors/bme680/) in my bedroom?  How much does it change over the day?

[![VOC plot](/assets/posts/2021-06-04-smarter-hass/voc-plot.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/voc-plot.jpg)

In addition, the same analytical tools can be used to make statistical, data-driven inferences about the **future states** of smart devices and sensors to create what I call *inferential automations*. Inferential automations determine actions based on abnormal states and measurements, for example, or reliable tendencies over a user-specified period of time:

- if the temperature started *decreasing significantly over the last five minutes*, then ___ .
- if the VOC started *increasing significantly over the last fifteen minutes*, then ___ .
- if the water level is *significantly lower than yesterday*, then ___ .
- if the number of detected cars on my camera is *significantly higher than thirty minutes ago*, then ___ .

[![VOC plot linear fit](/assets/posts/2021-06-04-smarter-hass/voc-plot-linear-fit.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/voc-plot-linear-fit.jpg)

Unfortunately, according to the HASS website, the [Statistics](https://www.home-assistant.io/integrations/statistics/) and related utilities are currently used by less than 5% of the HASS userbase.  I feel there is much to be explored and gained from the application of **dynamic statistical inferences** in home automation systems.  To mention a few reasons, sensors are susceptible to measurement error and many user-defined events (e.g., Carlos is at home) are multidimensional and frequently determined by more factors than integrated with a home automation system (e.g., my cellphone connected to my home's private network is not a sufficient condition to tell that I'm at home but it does inform about the likelihood that I am at home).  This creates uncertainty about the current and future states of things but fortunately, the uncertainty can be quantified and taken into account by various statistical tools that have already been developed.

Furthermore, the fact that HASS integrations are written in the [Python programming language](https://www.python.org/) makes HASS a prime candidate for exploring the use of statistical inference in home automation systems because many statistical packages are already available in Python and are widely used and well-maintained (e.g., [`numpy`](https://pypi.org/project/numpy/) and [`scipy`](https://pypi.org/project/scipy/)). Therefore, porting new and more sophisticated analytical tools to HASS should be fairly straightforward.  (More on this in the [Development](#development) section).

If you find these ideas interesting and want to get started on their implementation in your own personal HASS instances, then read on.  As in my previous guides and tutorials, I tried to unpack and digest as much of the content as possible, the goal being to make it accessible to experts as well as novices.  Check the [Changelog](#changelog) for updates and if you ever get stuck on something or just want to share a few ideas and opinions, feel free to [get in touch with me](/contact).

[top](#){: .btn .btn--light-outline .btn--small}


# Outline
- TODO: General picture of the structure

[top](#){: .btn .btn--light-outline .btn--small}


# Prerequisites
The implementation of analytical tools in HASS the following basic requirements:

1. A [HASS **Core**](#hass-core) instance;
2. Understanding of the [**YAML** syntax](#yaml-syntax);
3. Understanding the [HASS **database**](#hass-database);
4. And of course, [basic **statistics**](#statistics) knowledge.

Those four topics are described separately next.  If you are already familiar with them, feel free to skip straight to the main [Implementation](#implementation) section.  However, at the very least, I suggest to glance over each topic to make sure we are on the same page before moving on.

## HASS Core
Structurally, HASS can be divided into three main layers: (a) Core; (b) Supervisor; and (c) Operating System (OS).  The folks at the HASS wiki were kind enough to put together a plethora of [installation methods](https://www.home-assistant.io/installation/) for all sorts of OSes and environments (bare-metal vs. virtual).  For this guide, however, only the most basic component of the HASS system is needed, namely the **HASS Core**, which is available in *any* installation method.

Instead of using an existing HASS instance, I **strongly** recommend to **create a containerized (Docker) HASS instance** for testing purposes.  This will be much safer than playing with an existing HASS instance and its database.

To create a HASS Docker container, follow the instructions in the **official documentation**:

1. [Install **Docker Engine - Community Edition**](https://docs.docker.com/engine/install/);
2. (*Optional*.) [Install **Portainer - Community Edition**](https://documentation.portainer.io/v2.0/deploy/ceinstalldocker/) to manage your Docker containers;
3. [Install **HASS Docker container**](https://www.home-assistant.io/installation/linux#install-home-assistant-container).

Of note, after deploying the HASS container, use a host's text editor (e.g., `nano`, `vi`, `vim`, `pluma`) to edit the `configuration.yaml` file and related configuration files.  If you create a new `.yaml` file, make sure that the HASS user will have permission to read it, or you will run into issues.  Finally, in the HASS webUI, your user must be able to access the [**Developer Tools** > Services](https://www.home-assistant.io/docs/tools/dev-tools/) tab to check the state of each entity and their attributes.  The default admin user should have access to such a resource.

## YAML syntax
- Basics (https://www.home-assistant.io/docs/configuration/yaml/)
- Spiting up the `configuration.yaml` with `!include` statements and additional `*.yaml` files
- Reloading the `configuration.yaml`

## HASS database
- RDBMSes (HASS uses SQLite by default but you can create an use another RDBMSes, like MongoDB and MySQL)
- Settings (by default, HASS uses 10 days autopurge, 10sec DB write resolution, etc.)
- DB storage considerations
- Customization via `recorder:`

## Statistics
- Assume very basic statistical knowledge
  - Reference to free online stats material
  - Reference to good intro books
- Only simple, widely used inferential methods will be referred to here
- The possibilities are endless for knowledgeable users (application of Bayesian methods, hierarchical modeling, and so on.).  See notes on how to help to developing (porting) new analytic tools (integrations) for the HASS environment.

[top](#){: .btn .btn--light-outline .btn--small}


# Implementation
- HASS utility integrations (https://www.home-assistant.io/integrations/#utility)
  - general picture of the ones I find most useful and mentioned here
  - mention the last topic about development of new, more general analytic tools (porting numpy and other Py packages used in statistical analysis and mathematical modeling)

## Data properties
- Discrete vs. continuous
- How the data from each sensor is being collected
- Examples of sensor data (finance, weather, energy, object detection, gps tracking, etc)
- Setting up the data used to illustrate the application of inferential automations: Weather data

## Utilities
- Tools to prepare the collected data for inferential automations
- Each covered utility includes an usage example that anyone can follow along using a fresh or existing HASS instance

### History Stats
- Counting, compute ratios and state duration over a user-specified time
- Overview of the doc and configuration
- Illustrate usage with an example anyone can use and follow along

- Additional content
  - Codes and docs: 
    - HASS doc: https://www.home-assistant.io/integrations/history_stats/
    - HASS Github: https://github.com/home-assistant/core/tree/dev/homeassistant/components/history_stats
    - Methods defined by the integration itself (no external dependencies)

### Statistics
- Multiple, general purpose descriptive statistics
- Overview of the doc and configuration
- Illustrate with an example

- Additional content
  - Codes and docs:
    - HASS doc: https://www.home-assistant.io/integrations/statistics/
    - HASS Github: https://github.com/home-assistant/core/blob/dev/homeassistant/components/statistics/sensor.py
    - Python (statistics core pkg): https://docs.python.org/3/library/statistics.html

### Trend
- Of note, depends on the `numpy` Python pkg.  More specifically, uses the `np.polyfit(time, values, degree=1)` method to estimate the slope of a linear function over a user-speficied time period (gradient).
- Note about the comparison between the Trend utility vs. Derivative: Derivative is just for the last two values over the last two time points; trend can accomplish the same thing for n=2 and can.
- Overview of the doc
- Illustrate with an example anyone can use and follow along

- Additional content
  - Codes and docs: 
    - HASS doc: https://www.home-assistant.io/integrations/trend/
    - HASS Github: https://github.com/home-assistant/core/blob/dev/homeassistant/components/trend/binary_sensor.py
    - Python (NumPy): https://numpy.org/doc/stable/reference/generated/numpy.polyfit.html

### Integration
- Area under the curve; cumulative measures, such as kWh for energy consumption

- Additional content
  - Codes and docs:
    - HASS doc: https://www.home-assistant.io/integrations/integration/
    - HASS Github: https://github.com/home-assistant/core/blob/dev/homeassistant/components/integration/sensor.py
    - Methods defined by the integration itself (no external dependencies)

## Statistical inference
- Where your statistics knowledge comes in
- At the very basic level:
  - Confidence intervals (CI): https://www.statology.org/test-significance-regression-slope/ 
  - Distance from the mean in SD units
- Notes on sample, power, and null hypothesis testing for robust dynamic predictions

### Visualizing inferences
- Making use of HASS built-in visualization tools
  - Historical plots
  - Filters (state_filter gradients) to create heatmaps
- Customized plots

## Inferential automations
- Automations based on future states

[top](#){: .btn .btn--light-outline .btn--small}


# Development
- A lot more can be done via templating but consider developing and porting new analytic tools as utility integrations
  - Develop new integrations: https://developers.home-assistant.io/docs/development_index/
- Ideas
  - Generalized polynomial fits for the Trend utility, instead of the forced linear
    - default degree = 1 assures backward compatibility
  - Correlations (linear, rank)
  - Genrealized linear models and other common statistical models
  - Endless possibilities for analytic tool development; most require simple porting to the HASS environment because we already have many Python pkgs for statistical analysis (e.g., SciPy)

[top](#){: .btn .btn--light-outline .btn--small}


# Conclusion


This is all served via a beautiful and configurable web interface that can be accessed both locally and remotely using either any of the popular web browsers or its official app for [Android](https://play.google.com/store/apps/details?id=io.homeassistant.companion.android) and [iOS](https://apps.apple.com/us/app/home-assistant/id1099568401). 


[top](#){: .btn .btn--light-outline .btn--small}


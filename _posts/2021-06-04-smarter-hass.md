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
[Home Assistant](https://www.home-assistant.io/) (HASS) is a free and open-source software (FOSS) that provides a feature-rich environment for managing, controlling, and automating smart home devices, such as light bulbs, blinders, and LED strips.  In addition, it provides a highly customizable system for collecting and organizing a multitude of data (e.g., on/off device states, local temperature, GPS tracking, exchange rates), as provided by **more than a thousand [integrations](https://www.home-assistant.io/integrations)** with local [Internet of things](https://en.wikipedia.org/wiki/Internet_of_things) (IoT) devices (e.g., [Sonoff](https://sonoff.tech/), [Wyze](https://wyze.com/), [Z-Wave](https://www.z-wave.com/)), sensors (e.g., micro-controllers or single-board computers connected to sensor modules), and cloud-based services (e.g., weather and financial web APIs).

[![HASS demo frontend](/assets/posts/2021-06-04-smarter-hass/hass-demo.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/hass-demo.jpg)

More often than not, people use HASS to view or modify the **current state and value** of integrated devices and sensors via manual triggers (e.g., pressing a button to turn off the AC) or automations (e.g., if the temperature is lower than 14°C, then turn on the heat).  However, the [HASS utility integrations](https://www.home-assistant.io/integrations#utility) offer users the possibility to go (way) beyond with the help of built-in mathematical and statistical tools.  More specifically, such **analytical tools** allow users to summarize past states and measurements to answer questions such as:

- How many times has the front door been opened over the last 24hrs and for how long?
- How much energy (kWh) has my uninterrupted power supply used over the last month?
- What was the average temperature in the living-room during yesterday's morning, afternoon, and evening?  What about last week?
- What is the average level of volatile organic compounds (VOC) measured by the [BME680 sensor](https://www.bosch-sensortec.com/products/environmental-sensors/gas-sensors/bme680/) in my bedroom?  How much does it change over the day?

[![VOC plot](/assets/posts/2021-06-04-smarter-hass/voc-plot.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/voc-plot.jpg)

In addition, the same analytical tools can be used to make statistical, data-driven inferences about the **future states** of smart devices and sensors to create what I call *inferential automations*. Inferential automations determine actions based on abnormal states and measurements, for example, or reliable tendencies over a user-specified period of time:

- if the water level is *significantly lower than yesterday*, then ___ .
- if the number of detected cars on my camera is *significantly higher than thirty minutes ago*, then ___ .
- if the temperature started *decreasing significantly over the last five minutes*, then ___ .
- if the VOC started *increasing significantly over the last fifteen minutes*, then ___ .

[![VOC plot linear fit](/assets/posts/2021-06-04-smarter-hass/voc-plot-linear-fit.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/voc-plot-linear-fit.jpg)

Unfortunately, according to the HASS website, the [Statistics](https://www.home-assistant.io/integrations/statistics/) and related utilities are currently used by less than 5% of the HASS userbase.  I feel there is much to be explored and gained from the application of **dynamic statistical inferences** in home automation systems.  To mention a few reasons, sensors are susceptible to measurement error and many user-defined events (e.g., Carlos is at home) are multidimensional and frequently determined by more factors than integrated with a home automation system (e.g., my cellphone connected to my home's private network is not a sufficient condition to tell that I'm at home but it does inform about the likelihood that I am at home).  This creates uncertainty about the current and future states of things but fortunately, the uncertainty can be quantified and taken into account by various statistical tools that have already been developed.

Furthermore, the fact that HASS integrations are written in the [Python programming language](https://www.python.org/) makes HASS a prime candidate for exploring the use of statistical inference in home automation systems because many statistical packages are already available in Python and are widely used and well-maintained (e.g., [`numpy`](https://pypi.org/project/numpy/) and [`scipy`](https://pypi.org/project/scipy/)). Therefore, porting new and more sophisticated analytical tools to HASS should be fairly straightforward.  (More on this in the [Development](#development) section).

If you find these ideas interesting and want to get started on their implementation in your own personal HASS instances, then read on.  As in my previous guides and tutorials, I tried to unpack and digest as much of the content as possible, the goal being to make it accessible to experts as well as novices.  Check the [Changelog](#changelog) for updates and if you ever get stuck on something or just want to share a few ideas and opinions, feel free to [get in touch with me](/contact).

[top](#){: .btn .btn--light-outline .btn--small}


# Outline
- TODO: General picture of the structure

[top](#){: .btn .btn--light-outline .btn--small}


# Prerequisites
The implementation of analytical tools in HASS has the following basic requirements:

1. A [HASS **Core**](#hass-core) instance;
2. Understanding of the [**YAML** syntax](#yaml-syntax);
3. Understanding the [HASS **database**](#hass-database);
4. And of course, [basic **statistics**](#statistics) knowledge.

Those four topics are described separately next.  If you are already familiar with them, feel free to skip straight to the main [Implementation](#implementation) section.  However, at the very least, I suggest to glance over each topic to make sure we are on the same page before moving on.

## HASS Core
Structurally, HASS can be divided into three main layers: (a) Core; (b) Supervisor; and (c) Operating System (OS).  The folks at the HASS wiki were kind enough to put together a plethora of [installation methods](https://www.home-assistant.io/installation/) for all sorts of OSes and environments (bare-metal vs. virtual).  For this guide, however, only the most basic layer of the HASS system is needed, namely the **HASS Core**, which is available in *any* installation method.

Instead of using an existing HASS instance, I **strongly** recommend to **create a containerized (Docker) HASS instance** for testing purposes.  This will be much safer than playing with an existing HASS instance and its database.

To create a HASS Docker container, follow the instructions in the **official documentation**:

1. [Install **Docker Engine - Community Edition**](https://docs.docker.com/engine/install/);
2. (*Optional*.) [Install **Portainer - Community Edition**](https://documentation.portainer.io/v2.0/deploy/ceinstalldocker/) to manage your Docker containers;
3. [Install **HASS Docker container**](https://www.home-assistant.io/installation/linux#install-home-assistant-container).

Of note, after deploying the HASS container, use a host's text editor (e.g., `nano`, `vi`, `vim`, `pluma`) to edit the `configuration.yaml` file and related configuration files.  Whenever you create a new `.yaml` file, make sure that the HASS user will have permission to read it at the very least, or you will run into issues.  Finally, in the HASS webUI, your HASS user must be able to access the [**Developer Tools** > Services](https://www.home-assistant.io/docs/tools/dev-tools/) tab to check the state of each entity and their attributes.  The default admin user should have access to such a resource.

## YAML syntax
The configuration files in HASS use a human-readable data serialization language called [**YAML**](https://yaml.org/) (YAML Ain't Markup Language).  In this guide, we will use YAML to edit and create configuration files for HASS that will customize database settings and new entities to collect data and help with their visualization.

If you are new to YAML, take a few minutes right now to familiarize yourself with it.  The HASS wiki has a straight to the point explanation that I invite everyone to read:

- [https://www.home-assistant.io/docs/configuration/yaml/](https://www.home-assistant.io/docs/configuration/yaml/)

To highlight a few important points about the configuration files and the YAML syntax:

- Indentation and spacing in general are **very** important in YAML. Use only the spacebar and maintain consistency across all configuration files. If at all possible, use an editor that shows whitespaces when editing any YAML files;
- UPPER and lower cases matter;
- Use `#` to add comments;
- Use `""` (double quotation marks) for escaping special characters in sequences (e.g., `"Hey! What's up?"`) and `''` (single quotation marks) when escaping is not necessary (e.g., `'Nothing much.'`);
- Use the `!include` for splitting up your `configuration.yaml`. For instance, instead of adding all your `sensor:` and `binary_sensor:` to the `configuration.yaml` file itself, create `sensors.yaml` and `binary_sensors.yaml` files in the `config/` directory and then use `!include` to add them automatically to your `configuration.yaml`:
  ```yaml
  # Sensors
  sensor: !include sensors.yaml
  binary_sensor: !include binary_sensors.yaml
  ```
- Use `!secret` and a `secrets.yaml` for managing passwords.  (See more in the HASS wiki called [Storing secrets](https://www.home-assistant.io/docs/configuration/secrets/).)

Finally, after making any changes to any YAML file (and saving them), it is necessary to [**reload** the `configuration.yaml` file](https://www.home-assistant.io/docs/configuration/#reloading-changes).  If your installation method does not allow for selective reloading, then go ahead and reload the entire HASS.  Keep an eye on the `home-assistant.log` file for errors.  This will help you troubleshooting most issues on your own.

## HASS database
HASS uses a relational database (DB) management system (RDBMS) based on the **SQL engine** and by default, it creates a [**SQLite DB**](https://www.sqlite.org/index.html) in `config/home-assistant_v2.db` to track events and parameters over time.  If you want to learn more about and dive into the HASS DB, including how to create your own SQL backend, take a look at the two following resources:

- [https://www.home-assistant.io/docs/backend/database/](https://www.home-assistant.io/docs/backend/database/)
- [https://www.home-assistant.io/integrations/recorder/](https://www.home-assistant.io/integrations/recorder/)

Fortunately, most users won't need to learn RDBMSes and SQL to take advantage of analytical tools in HASS because the default SQLite DB is usually enough.  However, there are a few aspects about the **default settings** that can affect the way the data are analyzed.  Those aspects are described in more detail next.

### Default database
In the `configuration.yaml` file, the **default SQLite DB** is created along with many of the other default settings by the following configuration variable:

```yaml
default_config:
```

To change the default DB settings, it is necessary to add a `recorder:` configuration variable to the `configuration.yaml` file, as follows:

```yaml
default_config:
# Customized DB settings
recorder:
```

The documentation of the specific `recorder:` variables can be found at the HASS wiki:

- [https://www.home-assistant.io/integrations/recorder/](https://www.home-assistant.io/integrations/recorder/)

In brief, by default, the HASS DB **keeps historical data up to 10 days** (`purge_keep_days: 10`), running an automatic purge of the database every night to prevent the DB from increasing in size indefinitely (`auto_purge: true`).  Therefore, *if you want to keep data from one or more entities for longer than 10 days*, then you must edit the DB default settings.

In addition, as mentioned before, the default DB is stored in your `config/` directory and also by default, **changes are committed to the DB every 1 sec** (`commit_interval: 1`). This matters because if you are collecting data over a time window lower than 1 sec, then you might want to change the commit interval to `0` (zero, or as soon as possible) instead. At this point, it is also important to consider where the DB is being physically stored (SD card, eMMC, HDD, or SSD), owing to **disk I/O** and **wear and tear** considerations.  (More advanced aspects come into play if HASS is not the only application committing to the DB but I trust that if this is your case, then you probably know how to customize the HASS DB accordingly.)

For advanced users who want to browse and manually add/edit entries from the default (SQLite) `home-assistant_v2.db`, it is possible to use the [SQLite Database Browser](https://sqlitebrowser.org/).  Just keep in mind that the default HASS DB might have permissions (ownership and group membership) that are incompatible with your current host user and therefore, you might be unable even to read the DB without first editing the permissions--in Linux, appending `sudo` to `sqlitebrowser` should give you access no matter what though.

## Statistics

[![Humor about stats](/assets/posts/2021-06-04-smarter-hass/humor-stats.png){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/humor-stats.png)

You don't need to be a mathematician who specialized in statistics to make use of it.  In this guide, we will only make reference to very introductory statistical knowledge, such as measures of central tendency (e.g., mean, median), variability (e.g., variance, standard deviation) and simple (univariate) linear regression (e.g., gradient/slope). Similarly, math-wise, I will briefly talk about first-order derivatives (the ratio of the increment) and integrals (the area under the curve).

The goal in this guide is to present a starting point for more advanced usage of analytical tools in home automation systems.  The possibilities are endless for knowledgeable users (e.g., application of Bayesian methods, dynamic mixed-effects modeling) and how far you will go along these paths is up to you.

Fortunately, it doesn't take much to create really good inferential automations for your HASS instances. If you want to refresh your stats knowledge or dig deeper into it, save some time and take a look at the following resources:

- Basic calculus refresher:
  - *TODO: Embed videos? Series + Derivative + Integration*

- Basic stats refresher:
  - *TODO: Embed videos? Probability + Basic stats*

- Reading material:
  - Introduction:
    - *TODO: Link to papers and books*

  - Advanced topics:
    - *TODO: Link to papers and books*

[top](#){: .btn .btn--light-outline .btn--small}


# Implementation
The main functionalities in HASS are extended by the configuration of new **integrations**.  According to the [HASS Glossary](https://www.home-assistant.io/docs/glossary/), 

> [Integrations](https://www.home-assistant.io/integrations/) provide the core logic for the functionality in Home Assistant.

Therefore, the analytical tools covered in this guide are implemented by one or more of the **1800** integrations that are currently supported by a community of home automation enthusiasts.  More specifically, most of the analytical tools are grouped under [Utility](https://www.home-assistant.io/integrations/#utility) integrations and in this guide, I will cover the following four:

1. [History Stats](#history-stats)
2. [Statistics](#statistics-1)
3. [Trend](#trend)
4. [Integration](#integration)

The current set of analytical integrations is fairly limited in what it can do.  For the most part, the tools can be used to create summary statistics of the past states and measurements of integrated devices and sensors.  Inference-wise, a lot can be done via [templates](https://www.home-assistant.io/docs/configuration/templating/) but moving forward, there is a need for more advanced analytical integrations.  For this reason, at the end of this guide, I added a section about [Development](#development) for anyone interested in helping out.  First, however, we need to talk about data.

## Data
Before delving into any analytical integration, there are at least three things that we need to do. First, we need think about the nature of the data.  For example, consider the default [Sun](https://www.home-assistant.io/integrations/sun/) (`sun.sun`) entity in HASS:

[![Sun entity](/assets/posts/2021-06-04-smarter-hass/hass-entity-sun.jpg){:.PostImage}](/assets/posts/2021-06-04-smarter-hass/hass-entity-sun.jpg)

While the `sun.sun` *state* is **discrete** (it's either `above_horizon` or `below_horizon`), its `elevation` *attribute* is actually **continuous** (e.g., `35.84`° between the sun and the horizon) and therefore, it doesn't make sense to use the same tools to analyze both of them, does it?  Nonetheless, discrete variables can be transformed into continuous ones (e.g., the sun was `above_horizon` for `34.1`% of the day), and similarly, continuous variables can be discretised (e.g., the elevation was either `negative` or `positive` or `zero`) in order to better answer our questions of interest.

Second, we need to check whether HASS is keeping track of the data.

Third, we need to check how the data are being represented in the DB.

In HASS, it is possible to find a list of all entities and their attributes in Developer Tools > States.

- Discrete vs. continuous
- Examples of sensor data (finance, weather, energy, object detection, gps tracking, etc)
- Setting up the data used to illustrate the application of inferential automations: Weather data

## Sampling
- How the data from each sensor is being collected affects your conclusions


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
  - Mini-graph card (https://github.com/kalkih/mini-graph-card)

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


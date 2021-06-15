---
title: "Towards a smarter Home Assistant: Getting started on the analytical tools (and moving beyond)"
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
[Home Assistant](https://www.home-assistant.io/) (HASS) is a free and open-source software (FOSS) that provides a feature-rich environment for managing, controlling, and automating smart home devices, such as light bulbs, blinders, and LED strips.  In addition, it provides a highly customizable system for collecting and organizing a multitude of data (e.g., on/off device states, local temperature, GPS tracking, exchange rates), as provided by **more than a thousand [integrations](https://www.home-assistant.io/integrations)** with [Internet of things](https://en.wikipedia.org/wiki/Internet_of_things) (IoT) devices (e.g., [Sonoff](https://sonoff.tech/), [Wyze](https://wyze.com/), [Z-Wave](https://www.z-wave.com/)), local sensors (e.g., micro-controllers or single-board computers connected to sensor modules), and cloud-based services (e.g., weather and financial web APIs).

[![HASS demo frontend](/assets/posts/2021-06-04-smarter-hass/hass-demo.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/hass-demo.jpg)

More often than not, people use HASS to view or modify the **current state and value** of integrated devices and sensors via manual triggers (e.g., pressing a button to turn off the AC) or automations (e.g., if the temperature is lower than 14°C, then turn on the heat).  However, the [HASS utility integrations](https://www.home-assistant.io/integrations#utility) offer users the possibility to go (way) beyond with the help of built-in mathematical and statistical tools.  More specifically, such **analytical tools** allow users to summarize past states and measurements to answer questions such as:

- How many times has the front door been opened over the last 24hrs and for how long?
- How much energy (kWh) has my uninterrupted power supply used over the last month?
- What was the average temperature in the living-room during yesterday's morning, afternoon, and evening?  What about last week?
- What is the average level of volatile organic compounds (VOC) measured by the [BME680 sensor](https://www.bosch-sensortec.com/products/environmental-sensors/gas-sensors/bme680/) in my bedroom?  How much does it change over the day?

[![VOC plot](/assets/posts/2021-06-04-smarter-hass/voc-plot.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/voc-plot.jpg)

In addition, the same analytical tools can be used to make statistical, data-driven inferences about the **future states** of smart devices and sensors to create what I call *inferential automations*. Inferential automations determine actions based on abnormal states and measurements, for example, or reliable tendencies over a user-specified period of time:

- if the water level is *significantly lower than yesterday*, then __ .
- if the number of detected cars on my camera is *significantly higher than thirty minutes ago*, then __ .
- if the temperature started *decreasing significantly over the last five minutes*, then __ .
- if the VOC started *increasing significantly over the last fifteen minutes*, then __ .

[![VOC plot linear fit](/assets/posts/2021-06-04-smarter-hass/voc-plot-linear-fit.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/voc-plot-linear-fit.jpg)

Unfortunately, according to the HASS website, the [Statistics](https://www.home-assistant.io/integrations/statistics/) and related utilities are currently used by less than 5% of the HASS userbase.  I feel there is much to be explored and gained from the application of **dynamic statistical inferences** in home automation systems.  To mention a few reasons, sensors are susceptible to measurement error and many user-defined events (e.g., Carlos is at home) are multidimensional and frequently determined by more factors than integrated with a home automation system (e.g., my cellphone connected to my home's private network is not a sufficient condition to tell that I'm at home but it does inform about the likelihood that I am at home).  This creates uncertainty about the current and future states of things but fortunately, the uncertainty can be quantified and taken into account by various statistical tools that have already been developed.

Furthermore, the fact that HASS integrations are written in the [Python programming language](https://www.python.org/) makes HASS a prime candidate for exploring the use of statistical inference in home automation systems because many mathematical and statistical packages are already available in Python and are widely used and well-maintained (e.g., [`numpy`](https://pypi.org/project/numpy/) and [`scipy`](https://pypi.org/project/scipy/)). Therefore, porting new and more sophisticated analytical tools to HASS should be fairly straightforward.  (More on this in the [Development](#development) section).

If you find these ideas interesting and want to get started on their implementation in your own personal HASS instances, then read on.  As in my previous guides and tutorials, I tried to unpack and digest as much of the content as possible, the goal being to make it accessible to experts as well as novices.  Check the [Changelog](#changelog) for updates and if you ever get stuck on something or just want to share a few ideas and opinions, feel free to [get in touch with me](/contact).

[top](#){: .btn .btn--light-outline .btn--small}


# Outline
- TODO: General picture of the structure

[top](#){: .btn .btn--light-outline .btn--small}


# Prerequisites
The implementation of analytical tools in HASS has the following basic requirements:

1. A [HASS **Core**](#hass-core) instance;
2. Understanding of [the **configuration** files and the **YAML** syntax](#hass-configuration-files);
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

## HASS configuration files
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

Finally, after making any changes to any YAML file (and saving them), it is necessary to [**reload** the `configuration.yaml` file](https://www.home-assistant.io/docs/configuration/#reloading-changes).  If your installation method does not allow for selective reloading, then go ahead and reload the entire HASS.  However, *before reloading any configuration file*, use the `hass --script check_config` script to make sure your `configuration.yaml` file is okay, as follows:

- From within the HASS webUI, navigate to **Configuration** > **Server Controls** > Configuration validation.
  
  [![HASS config validation](/assets/posts/2021-06-04-smarter-hass/hass-config-validation.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/hass-config-validation.jpg)
  
  [![HASS config reload](/assets/posts/2021-06-04-smarter-hass/hass-config-reload.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/hass-config-reload.jpg)
  
  Alternatively, if running the HASS Docker container, use `docker exec homeassistant python -m homeassistant --script check_config --config /config` to run the `check_config` script from your host machine.  (This assumes that your HASS container is called `homeassistant`. If this is not the case, change it accordingly.)
  {:.notice}

Always keep an eye on the `home-assistant.log` file for errors.  This will greatly help you troubleshooting most issues on your own (e.g., incorrect references or operations in templates).

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

In brief, by default, the HASS DB **keeps historical data up to 10 days** (`purge_keep_days: 10`), running an automatic purge of the database every night to prevent the DB from increasing in size indefinitely (`auto_purge: true`).  Therefore, *if you want to keep data from one or more entities for longer than 10 days*, then you must edit the DB default settings.  The `include` and `exclude` filter parameters are particularly useful whenever working with non-default settings, as they help you specify which entities should be tracked.  [Check the HASS wiki Configure Filter for examples](https://www.home-assistant.io/integrations/recorder/#common-filtering-examples).

In addition, as mentioned before, the default DB is stored in your `config/` directory and also by default, **changes are committed to the DB every 1 sec** (`commit_interval: 1`). This matters because if you are collecting data over a time window lower than 1 sec, then you might want to change the commit interval to `0` (zero, or as soon as possible) instead. At this point, it is also important to consider where the DB is being physically stored (SD card, eMMC, HDD, or SSD), owing to **disk I/O** and **wear and tear** considerations.  (More advanced aspects come into play if HASS is not the only application committing to the DB but I trust that if this is your case, then you probably know how to customize the HASS DB accordingly.)

For advanced users who want to browse and manually add/edit entries from the default (SQLite) `home-assistant_v2.db`, it is possible to use the [SQLite Database Browser](https://sqlitebrowser.org/).  Just keep in mind that the default HASS DB might have permissions (ownership and group membership) that are incompatible with your current host user and therefore, you might be unable even to read the DB without first editing the permissions--in Linux, appending `sudo` to `sqlitebrowser` should give you access no matter what though.

### Resetting entity data in the DB
In addition to the SQL browser method of editing the HASS DB, HASS allow users to run a [service call](https://www.home-assistant.io/docs/scripts/service-calls/) from within the webUI to manually run a **purge task**. This is particularly useful to run once you are done making changes to a newly created template entity and want to clean any previous (and possibly erroneous) records from the DB.  To remove all the data from a list of entities, do the following:

1. Navigate to **Developer Tools** > States;
2. Select **Go to YAML mode** and paste the following:
   ```yaml
   service: recorder.purge_entities
   target:
     entity_id:
       - sensor.my_template_entity_01
       - sensor.my_template_entity_02
   ```
   in which `sensor.my_template_entity*` are the target template entities you want to purge;
3. Press **Call Service** to purge their data from the DB. The new data will be collected as soon as an update is triggered (see the [Sampling](#sampling) section).

Of course, there are other `recorder` services available in the Developer Tools > Services tab. Feel free to explore them.

## Statistics
You don't need to be a mathematician who specialized in statistics to make use of it.  In this guide, we will only make reference to very introductory statistical knowledge, such as measures of central tendency (e.g., mean, median), variability (e.g., variance, standard deviation) and simple (univariate) linear regression (e.g., gradient/slope). Similarly, math-wise, I will briefly talk about first-order derivatives (the ratio of the increment) and integrals (the area under the curve).

[![Humor about stats](/assets/posts/2021-06-04-smarter-hass/humor-stats.png){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/humor-stats.png)

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

The current set of analytical integrations is fairly limited in what it can do.  For the most part, the tools can be used to create summary statistics of the past states and measurements of integrated devices and sensors.  Inference-wise, a lot can be done via [templates](https://www.home-assistant.io/docs/configuration/templating/) but moving forward, there is a need for more advanced analytical integrations.  For this reason, at the end of this guide, I added a section about [Development](#development) for anyone interested in helping out.  First, however, we need to talk about [Data](#data) and [Sampling](#sampling).

## Data
Before delving into any analytical integration, there are at least three things that we need to do. First, we need to think about the nature of the data.  For example, consider the default [Sun](https://www.home-assistant.io/integrations/sun/) (`sun.sun`) entity in HASS:

[![Sun entity](/assets/posts/2021-06-04-smarter-hass/hass-entity-sun.jpg){:.PostImage}](/assets/posts/2021-06-04-smarter-hass/hass-entity-sun.jpg)

While the `sun.sun` *state* is a **discrete** variable (it's either `above_horizon` or `below_horizon`), its `elevation` *attribute* is actually **continuous** (e.g., `35.84`° between the sun and the horizon) and therefore, it doesn't make sense to use the same tools to analyze both of them.  Nonetheless, discrete variables can be transformed into continuous ones (e.g., the sun was `above_horizon` for `34.1`% of the day), and similarly, continuous variables can be discretised (e.g., the elevation is either `negative` or `positive` or `zero`) in order to better answer our questions of interest. 

Second, we need to check whether HASS is keeping track of the data we need. There are multiple ways of doing that but by far, the easiest method is to navigate to **Developer Tools** > States and then make sure that the entities whose states and attributes we would like to keep track of are being listed there.  (Alternatively, you can open the HASS DB with a SQL browser and look for the entity in the `states` Table.)

[![HASS developer tools](/assets/posts/2021-06-04-smarter-hass/hass-developer-tools.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/hass-developer-tools.jpg)

Third, we need to check how the data are being represented in the DB.  As in the previous example, some variables might be an attribute of an existing entity in the HASS DB.  If the `recorder:` settings for such entity are fine for the type of analysis you want to automate (e.g., purge every 10 days), then you should be fine.  However, notice that *attributes* cannot be displayed the same way as the *states* of an entity in the HASS webUI.  In addition, you might want to **pre-process** the attributes (e.g., `float` and then `round(2)`) or perform transformations before running the analysis.

For all such a reasons, I always create **new entities** for the variables that will be analyzed. This is accomplished with the [`template` integration](https://www.home-assistant.io/integrations/template/).  Per the HASS wiki:

> The template integration allows creating entities which derive their values from other data. This is done by specifying templates for properties of an entity, like the name or the state.

[Building templates](https://www.home-assistant.io/docs/configuration/templating/#building-templates) is fairly easy once you get the hang of the syntax.  In short, templates follow the [**Jinja2** templating engine](https://palletsprojects.com/p/jinja) and are mainly used to perform mathematical operations (`+`, `-`, `*`, `/`) and logic tests (if `true` , then __ ) but can also do loops (for `i` in `states.sensor`), for example.  As a result, templates give users a scripting tool to go beyond the HASS built-in functionalities.

As an example, let's create an entity to store and round to zero the `sun.sun` `elevation` attribute.  First, in the `configuration.yaml`, **append** (add to the bottom) a reference to the `templates.yaml` configuration file:

```yaml
# Templates
template: !include templates.yaml
```

Then, use a text editor to create an empty `templates.yaml` file and create a sensor template for the sun elevation, as follows:

{% raw %}
```yaml
# Sensor templates
- sensor:
    - name: "template sun elevation"
      unit_of_measurement: "°"
      state: >
        {{ state_attr('sun.sun', 'elevation') | float | round(0) }}
```
{% endraw %}

Notice that in `state:`, we use `state_attr()` to retrieve the `elevation` attribute of the `sun.sun` entity. Then, we use `float` to force the output to a [floating point number](https://en.wikipedia.org/wiki/Floating-point_arithmetic), which ensures that the output is interpreted as a number, and run `round(0)` on the numeric output to round the decimals to zero cases.  The result should be an [integer](https://en.wikipedia.org/wiki/Integer) of the Sun elevation.  (Alternatively, we could simply use `int` to convert the output to an integer, of course.)

I find the use of the folded style (`>`) very helpful in keeping the templates organized. Refer to the [YAML - Scalars](https://yaml.org/spec/1.2/spec.html) documentation for more information.
{:.notice}

Check your `configuration.yaml` file for errors (Configuration > Server Controls > Configuration validation), and finally, **reload your HASS**.  Afterwards, navigate to Developers Tools > States and if everything is correct, you should see a new `sensor.template_sun_elevation` entity:

[![HASS template sun elevation](/assets/posts/2021-06-04-smarter-hass/hass-entity-template-sun-elevation.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/hass-entity-template-sun-elevation.jpg)

By default, the values of this newly created entity will update as soon as the Sun `elevation` attribute changes. However, it is also possible to configure different **triggers** for **template entities**.  This is particularly relevant for the next topic, namely data sampling.

## Sampling
Devices, sensors, and services measure and transmit data with a certain frequency, which I refer by the term **measurement resolution**.  Such a resolution might be determined by a time-based rule (e.g., every 1 sec) or an event (HTTP request) or a combination of both.  Regardless of the nature of the trigger, the *higher* the measurement resolution, the more frequent the measurement and transmission are.  For example, an [ESP32 Development board](https://www.espressif.com/en/products/devkits) connected to a [BME280 environmental sensor](https://www.bosch-sensortec.com/products/environmental-sensors/humidity-sensors-bme280/) might send a temperature measurement every 5 minutes to a MQTT broker.  Therefore, for all intended purposes, the measurement resolution of such a temperature sensor is at best 5 minutes.  Now, if a second ESP32 measures and send data *every 1 minute* instead, then the measurement resolution of the latter ESP32 is higher than the former (i.e., we can expect it to send temperature data more frequently).

As mentioned before, in HASS, **template entities** follow state-based updates by default--that is, they update as soon as the data of any of the referenced entities change.  Let's say that over a 20-min window, none of the referenced data changed.  This means that the template entity also didn't change and more importantly, in the DB, there will be a *single data point* over the 20-min window.  However, let's say that over the same 20-min window, one of the referenced data changed twice. This means that the template entity now changed twice and more importantly, in the DB, there will be *three data points* over the 20-min window.  This a very efficient way of storing data but you can probably see how this might affect the usage of analytical tools, owing to an inconsistent number of data points over equal time-frames as well as the under-representation of data over time.

Fortunately, just like we specify triggers for automations, HASS offers the possibility to specify [triggers for template entities](https://www.home-assistant.io/integrations/template/#trigger-based-template-sensors).  There's a large variety of triggers that can be used here (e.g, `webhook`, `mqtt`) but for **longitudinal analysis**, the most useful one is the [**time pattern** trigger](https://www.home-assistant.io/docs/automation/trigger#time-pattern-trigger), called `time_pattern`.  Time pattern triggers can be specified for hours (`hours:`), minutes (`minutes:`), and seconds (`seconds:`), and for each one, it's possible to prefix the value with a `/` to match whenever the current value is divisible by the specified value (e.g., `hours: "/2"` will match every 2 hours) or use `*` to match any value (e.g., `minutes: "*"` to match every minute).

To create a time pattern trigger for one or more template entity sensor, simply add a list of time-based `sensor:` and `binary_sensor:` under a `trigger:` configuration in the `templates.yaml` configuration file, as follows:

{% raw %}
```yaml
# Time pattern trigger
- trigger:
    - platform: time_pattern
      # Update every 5 minutes
      minutes: "/5"
  # Sensor templates
  sensor:
    - name: "template sun elevation - time-based"
      unit_of_measurement: "°"
      state: >
        {{ state_attr('sun.sun', 'elevation') | float | round(0) }}
      attributes:
        timestamp: >
          {{ as_timestamp(now()) }}
```
{% endraw %}

Of note, the `timestamp:` under `attributes:` forces HASS to create a new entry on the DB (update) whenever the template is triggered.  This keeps the number of data points constant over time and each one of them will have a timestamp as attribute.  (In the DB, you will also see that if the state remained unchanged, it maintains the correct `last_changed` date and updates the `last_updated` variable.)

Also, notice that it does not make sense to use a `time_pattern` trigger rule that has higher resolution than the device's *measurement* resolution because then, there's no chance for a new value to occur and the DB would just repeat the last transmitted measurement.  Ideally, the time pattern should **match the measurement resolution** but to save space, you might want to set a lower time pattern resolution (as in 1:2, or 1:4, and so on).

Beware that depending on how triggers are configured and how many template entities are created, the HASS DB might end up using **a lot of space** and computations are bound to use **ever more CPU (and possibly RAM) resources**, owing to the number of entries in the DB.  Be sure to monitor such resources after configuring your HASS instance; Otherwise, your HASS instance might experience serious problems.
{:.notice .notice--warning}

Finally, there is the topic of statistical sampling (representativeness) when it comes to making generalizations from a couple of samples (e.g., environmental sensors in my bedroom and kitchen) to a population (temperature and humidity in my entire house).  In this guide, however, we will only make extrapolations about the devices, sensors, and services themselves, rather than any population that they might belong.

## Utilities
The [**Utility integrations**](https://www.home-assistant.io/integrations/#utility) offer users tools to parse and analyze data from recorded entities.  There are more than 30 different such integrations and they sometimes have overlapping functionalities.  For example, the gradient (ratio of change) over the last two data points can be computed by both the [Trend](https://www.home-assistant.io/integrations/trend/) integration and the [Derivative](https://www.home-assistant.io/integrations/derivative/).  In what follows, I covered only four of the utility integrations that I find most comprehensive and useful, namely:

1. [History Stats](#history-stats)
2. [Statistics](#statistics-1)
3. [Integration](#integration)
4. [Trend](#trend)

Trend is covered last because out of the four, it's the only one that actually makes use of *inferential* tools via the Python package `numpy`. The others provide counting, other mathematical resources, and ways to summarize historical data.  For each utility, I provided a brief description, usage examples to follow along, and reference to the documentation and code.

### History Stats
The [History Stats](https://www.home-assistant.io/integrations/history_stats/) integration provides useful statistics for discrete variables over a user-specified timespan.  More specifically, this integration can do one of three things depending on the chosen type of sensor:

- `type: time`: calculate the *amount of hours* that an entity has spent on a given state;
- `type: ratio`: calculate the *percentage of time* that an entity has spent on a given state;
- `type: count`: count how many times an entity has changed to a given state.

The use of this integration requires specification of **at least two** of the following time period variables:

- `start:`: It indicates when the time period starts.  Use [time templating](https://www.home-assistant.io/docs/configuration/templating/#time) to specify the time;
- `end:`: It indicates when the time period ends.  Use [time templating](https://www.home-assistant.io/docs/configuration/templating/#time) to specify the time;
- `duration:`: It indicates how long the time period lasts.  If `start:` is specified, then how long forwards;  if `end:` is specified, then how long backwards.  The syntax can be either the traditional `HH:MM:SS` format or as follows:
  ```yaml
  duration:
    days: 2
    hours: 4
    minutes: 15
    seconds: 30
  ```

#### Usage examples
`history_stats` are defined as a platform (`- platform: history_stats`) under `sensor:` in your `configuration.yaml` file.  To keep things organized, go to the `configuration.yaml` and append a reference to `sensors.yaml`, as follows:

```yaml
# Sensors
sensor: !include sensors.yaml
```

Then using a text editor, create a `sensors.yaml` file where we will configure the following three `history_stats` sensors to consume the data from the default [`weather.home`](https://www.home-assistant.io/integrations/weather/) entity:

1. `sunny yesterday hours`: Number of hours that the `weather.home` entity remained in the `sunny` state *yesterday*;
2. `cloudy today ratio`: Percentage of time that the `weather.home` entity remained in the `cloudy` state *today*;
3. `rainy week count`: Number of times that the `weather.home` entity changed to the `rainy` state over the *last seven days*.

The [Meteorologisk institutt (Met.no)](https://www.home-assistant.io/integrations/met/) is the current default meteorological information service used by HASS. This integration's measurement resolution is 1 hour (via cloud polling) and it follows state-based updates (a new entry in the DB is only created if any of the values has changed).
{:.notice}

To create those three `history_stats` sensor entities, append the following to `sensors.yaml`:

{% raw %}
```yaml
# History Stats
- platform: history_stats
  name: "sunny yesterday hours"
  entity_id: weather.home
  state: "sunny"
  type: time
  # end today at 00:00:00
  end: >
    {{ now().replace(hour=0, minute=0, second=0) }}
  # start 24h before the end time
  duration:
    hours: 24
- platform: history_stats
  name: "cloudy today ratio"
  entity_id: weather.home
  state: "cloudy"
  type: ratio
  # start today at 00:00:00
  start: >
    {{ now().replace(hour=0, minute=0, second=0) }}
  # end right now
  end: >
    {{ now() }}
- platform: history_stats
  name: "rainy week count"
  entity_id: weather.home
  state: "rainy"
  type: count
  # end right now
  end: >
    {{ now() }}
  # start 7 days before the end time
  duration:
    days: 7
```
{% endraw %}

Now **check your configuration file** and if everything looks good, **restart HASS**.  Afterwards, check your **log** file for related errors and if it all looks good, then head to **Developer Tools** > States and you should now see the three new entities we just created:

[![HASS utility history stats](/assets/posts/2021-06-04-smarter-hass/hass-utility-historystats.jpg){:.PostImage .PostImage--large}](/assets/posts/2021-06-04-smarter-hass/hass-utility-historystats.jpg)

For my example above, it's been cloudy for 2.6% of the time today, we had roughly 10 hours of sunny weather yesterday, and the weather changed to rainy 5 times over the week. 

Because I have not been running HASS and collecting `weather.home` data over this week in a reliable way, the reported metrics can be quite misleading.  For a proper representation of the stats over the configured timespans, make sure to keep your HASS instance running (and the cloud polling is working) for at least as long as the configured timespans.
{:.notice--warning}

#### Additional references
- History Stats **documentation**: [https://www.home-assistant.io/integrations/history_stats/](https://www.home-assistant.io/integrations/history_stats/)
- History Stats **source**: [Github](https://github.com/home-assistant/core/tree/dev/homeassistant/components/history_stats)


### Statistics
- Multiple, general purpose descriptive statistics
- Overview of the doc and configuration
- Illustrate with an example

- Additional content
  - Codes and docs:
    - HASS doc: https://www.home-assistant.io/integrations/statistics/
    - HASS Github: https://github.com/home-assistant/core/blob/dev/homeassistant/components/statistics/sensor.py
    - Python (statistics core pkg): https://docs.python.org/3/library/statistics.html

### Integration
- Area under the curve; cumulative measures, such as kWh for energy consumption

- Additional content
  - Codes and docs:
    - HASS doc: https://www.home-assistant.io/integrations/integration/
    - HASS Github: https://github.com/home-assistant/core/blob/dev/homeassistant/components/integration/sensor.py
    - Methods defined by the integration itself (no external dependencies)

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

#### Customized plots
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


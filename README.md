# emhass_czech_spot_goodwe

Implementation of EMHASS, Home Assistant Czech Energy Spot Prices and Goodwe solar Inverter

![License](https://img.shields.io/badge/license-MIT-blue.svg) ![Version](https://img.shields.io/badge/version-0.0.1-green.svg)

## Table of Contents

- [Features](#features)
- [Tech Stack]
- [Used apps and integrations]
- [Implementation steps]
- [Usage]
- [FAQ](#faq)
- [License](#license)

## Features

- Optimizes electricity use on solar system with battery 
- Runs EMHASS with price forecast from Czech OTE-CZ
- Controls Goodwe ET inverter from EMHASS forecast

## Tech Stack

- **Home Assistant Green**
- **Goodwe 8K-ET solar inverter**

## Used apps and integrations

- Energy Management for Home Assistant https://github.com/davidusb-geek/emhass
- GoodWe solar inverter for Home Assistant (experimental) https://github.com/mletenay/home-assistant-goodwe-inverter
- Home Assistant Czech Energy Spot Prices https://github.com/rnovacek/homeassistant_cz_energy_spot_prices

## Implementation steps

### Background 
In the past I have been running Home assistant with custom python code (hardwired MPC script) to control the behavior of solar inverter.

In 2026 my electricity provider changed from 1-hour spot prices to 15-min spot prices. As my code was made from the bootom for 1-hour steps, I have decided not to update it, but instead try to setup EMHASS.

I have tried EMHASS before, but in that time, the only possible way to pass price forecasts was by CSV files, which had to fit the time slots of forecast. This was for me too complicated.

In 10/2026, I realized, that EMHASS has implemented much better way of passing data - by JSON object, with timestamped array of prices or power forecasts.

### Installing EMHASS

On first try I installed the addon and tried to setup all the possible parameters. It resulted in non-working installation. So I uninstalled everything and started from blank installation, changing the parameters step by step, and double checking the effects of the changes.

### Installing Home Assistant Czech Energy Spot Prices

The installation process is very easy and well explained on its github site.
The help with setting up selling and buying prices works well.

I created one service with 15-min interval.
From this service I use `sensor.current_buy_electricity_price_15min` and `sensor.current_sell_electricity_price_15min`

### Useful info
There is a page https://koningrenzo.nl/en/home-energy/install-emhass-home-assistant/
which was big help for me. It's worth going through all his site https://koningrenzo.nl/en/home-en/ .



### REST-commands

For calling EMHASS API, I created two REST commands - see `rest_command.yaml`
This file is placed in config directory.

To work preperly, add this line to your `configuration.yaml`:
``` YAML
rest_command: !include rest_command.yaml 
```


### Scripts

For easy calling of REST commands, I made Homeassistant scripts, which will be called from other automations.
`EMHASS call` for calling optimization 
and `EMHASS publish data` for forced pudlishing of forecast data into homeassistant sensors.

#### Script: Emhass call
Somehow it happened, that when I simply caller the REST-command, it returned with an error.
By trial and error I realized, when I call REST command again a little while after the first call, it runs smoothly and performs the optimalization.

So it is reason, why REST command is called twice.

### Automation: EMHASS call 

It simply calls script `EMHASS call`

#### Timing
I have tyrned on automatic publishing of forecast sensor in EMHASS. The update occurs at the beginning of each minute.
When I sheduled the calling at the beginning of 15 min interval, the optimization was run in the same time, when publishing of forecast sensors also took place. It resulted with some errorneous values, which started at xx:00:00 and finished few seconds later. In a timeline it looked like a teeth and was not good for controlling inverter.

Finally I came to solution, that I run optimization 30 seconds before 15 min interval and does not publish the result immedialtely, but wait to the whole 15 min slot, when the forecast is published automaticale at the beginning of a minute.

###  Automation: EMHASS control Goodwe

This automation prepares values for helpers `input_select.gw_req_mode` and `input_number.gw_req_power`.

The `input_select.gw_req_mode` has states:

- GENERAL - Default operation
- ECO_CHARGE - Battery is forcibly charged from grid (buying electricity)
- ECO_DISCHARGE - Battery is forcibly discharged to grid (selling electricity)
- BATT_STANDBY - battery keeps its charge. (load is supplied from the grid)

The reason why I used experimental integration of Goodwe inverter is, that it has EMS operation mode and can directly control battery charging and discharging.

###  Automation: EMHASS control Goodwe push

This automation simply takes `input_select.gw_req_mode` and `input_number.gw_req_power` helpers.
Based in its values, it sets EMS operation mode of Goodwe invertor.

By using two automations, the actual control of inverter occurs only, when some of the helper changes.

 
 

## Labeling
All helpers, scripts, automation, devices etc... I have marked with the same label `FVE`, which helps on finding entites (for example in History dashboard).

## Usage on own risk

This repository is mainly planned as a notepad for my implementation.
There are config files, which can be useful for someone, who wants to replicate my setup.
Anyway I have to stress out, that each one's HoemAssistant environment is different and you have to take this repository as an inspiration and double check all the functions in your environment.


## Roadmap

- [x] Integrate OTE-CR.CZ forecasts with EMHASS calls
- [x] Make forecasting automations
- [x] Control solar inverter from EMHASS forecasts
- [ ] Fine tuning of load forecast 
- [ ] Fine tuning of solar forecast 
- [ ] Deferrable load setup (water heater)

## FAQ

#### Question 1

Answer 1

#### Question 2

Answer 2

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

Made with ❤ by [JanHrebec](https://github.com/JanHrebec)
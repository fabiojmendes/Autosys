<!-- vim: set tw=80: -->

# Autosys

Various automation-related IoT projects using Rust that communicate primarily
via MQTT.

## Architecture

![Architecture Diagram](assets/architecture.svg)

## Doorsys

Doorsys is a door access control system with centralized management and logging.

### Introduction

Doorsys was born out of frustration with consumer-grade smart locks. I needed
something more reliable with centralized logs, multiple access codes and badges,
and more importantly, remote management. After surveying the landscape, I came
to the realization that the options weren't great. Either you settle for a
subpar solution using one of the existing smart locks + app available on the
marketplace, or you have to spend an arm and a leg and step up to a more
professional solution. I did neither. I did what every engineer would do: roll
my own solution! How difficult can it be, right? Famous last words :)

Jokes aside, I took it as a challenge. After some research, I figured the
hardware necessary was readily available online, and an ESP32 microcontroller
would be more than enough to run the firmware. I started the development of the
firmware using C and the official
[ESP-IDF](https://github.com/espressif/esp-idf) library but later on switched to
the experimental yet highly capable rust based
[ESP-RS](https://github.com/esp-rs) implementation.

### Relevant repositories

- [doorsys-hardware](https://github.com/fabiojmendes/doorsys-hardware): in depth
  hardware breakdown, schematics, and PCB design.
- [doorsys-firmware](https://github.com/fabiojmendes/doorsys-firmware): ESP32-C3
  compatible firmware for controlling the hardware and communication with the
  backend.
- [doorsys-protocol](https://github.com/fabiojmendes/doorsys-protocol): Defines
  the messages supported by the platform.
- [doorsys-api](https://github.com/fabiojmendes/doorsys-api): Rest API for the
  admin interface and also handler for the MQTT messages.
- [doorsys-web](https://github.com/fabiojmendes/doorsys-web): Web interface for
  the admin console.

### Observability

The firmware constantly reports heap and flash usage, using MQTT messages in
the
[InfluxDB line protocol](https://docs.influxdata.com/influxdb/v1/write_protocols/line_protocol_tutorial/)
format. Telegraf is used to consume these messages and write them to the
InfluxDB instance, while a Grafana dashboard has been set up for visualization.

![Doorsys Dashboard](./assets/doorsys-dashboard.png)

### Deployment

The firmware is meant to be flashed directly onto the device. Check the
repository for details. Over-the-air updates are not supported at this time.
Container images are provided for the API and WEB components, which can then be
deployed using Docker or Podman. Personally, I run it using Podman and Quadlet
units (Systemd), but pick your poison.

## Tempsys

Tempsys is a monitoring system designed to measure the temperature of commercial
refrigerators and freezers. It is meant to monitor the behavior of these units
and anticipate any possible failures or misuse, thereby preventing food
degradation.

It consists of an extremely power-efficient Bluetooth LE module coupled with a
MCP9808 i2c digital thermometer. The whole system is powered by a single CR-2032
coin cell battery with autonomy of more than a year.

The low-power Bluetooth module is responsible for emitting advertising packets
containing temperature data. A more powerful device connected to the main power
collects those results and send them using MQTT in line protocol format. On the
backend, Telegraf and InfluxDB are used to collect and store those metrics. A
Grafana [dashboard](#tempsys-dashboard) is configured with alerts in case
temperatures are sustained above a certain threshold.

On top of the temperature, the module also sends its current voltage and RSSI
for observability purposes. This helps predict when it is time to change the
battery or if the device needs to be moved for better reception.

### Repositories

- [tempsys-hardware](https://github.com/fabiojmendes/tempsys-hardware) in-depth
  hardware description for the Tempsys device
- [tempsys-firmware](https://github.com/fabiojmendes/tempsys-firmware)
  embassy-rs based firmware for Tempsys
- [tempsys-scan](https://github.com/fabiojmendes/tempsys-scan) Bluez based
  application to read the Bluetooth advertising events from Tempsys and ship
  them via MQTT

### Tempsys dashboard

Here is a sample of what the dashboard looks like:

![Tempsys Dashboard](./assets/tempsys-dashboard.png)

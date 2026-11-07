# Tuya Smart IR AC Integration for Home Assistant

[![hacs_badge](https://img.shields.io/badge/HACS-Default-orange.svg)](https://github.com/hacs/integration)
[![Stable](https://img.shields.io/github/v/release/EnzoD86/tuya-smart-ir-ac)](https://github.com/EnzoD86/tuya-smart-ir-ac/releases/latest)
[![Donate](https://img.shields.io/badge/donate-BuyMeCoffee-yellow.svg)](https://www.buymeacoffee.com/enzod86)

Control air conditioners and other infrared (IR) appliances through a **Tuya / Smart Life IR blaster**, using the **Tuya Cloud (IoT Platform) API**. Optional real-time updates are available through the **Tuya Pulsar** message stream.

With this integration you can:

- 🌡️ Control **IR air conditioners** as full Home Assistant `climate` entities (mode, temperature, fan speed, presets).
- 📺 Control **any other IR remote** (TV, fan, projector, …) with one `button` entity per remote key.
- 💧 Read **Tuya temperature/humidity sensors** (temperature, humidity and battery).

> 💡 This repository is an evolved continuation of [DavidIlie's original project](https://github.com/DavidIlie/tuya-smart-ir-ac). It was rewritten to use **UI configuration (config flow)** instead of `configuration.yaml`, and it adds real-time updates via the **Tuya Pulsar WebSocket stream**.

---

## Table of contents

- [How it works](#how-it-works)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick start](#quick-start)
- [Documentation](#documentation)
- [Known limitations](#known-limitations)
- [Removing the integration](#removing-the-integration)
- [Contributing](#contributing)
- [Support me](#support-me)

---

## How it works

```
 Home Assistant ──HTTPS (OpenAPI)──▶ Tuya Cloud ──▶ IR blaster ──infrared──▶ Air conditioner / TV / …
       ▲                                  │
       └──────── WebSocket (Pulsar) ◀─────┘   (optional, real-time state changes)
```

- Commands go from Home Assistant to the Tuya Cloud, which forwards them to your IR blaster.
- Infrared is **one-way**: the AC never reports its real state. The state you see in Home Assistant is the **last state sent through Tuya** (from Home Assistant or from the Smart Life app). Changes made with the **physical remote control are not detected**.
- State is refreshed by **polling** (every 300 s by default). If you enable **Tuya Pulsar**, changes made in the Smart Life app also show up in Home Assistant almost instantly.

A deeper explanation is in [How the climate entity behaves](docs/climate-behavior.md) and [Architecture](docs/architecture.md).

## Requirements

- Home Assistant **2026.1.0** or newer.
- A Tuya-compatible **IR blaster** (Wi-Fi/Zigbee "Smart IR" hub) added to the **Smart Life** or **Tuya Smart** app.
- The appliances you want to control added **as remotes under the IR blaster** in the app (and working from the app).
- A free account on the [Tuya IoT Development Platform](https://platform.tuya.com) with a cloud project linked to your app account. See [Tuya Cloud setup](docs/tuya-cloud-setup.md).

## Installation

### Option 1: HACS (recommended)

1. Open **HACS** in Home Assistant.
2. Search for **Tuya Smart IR AC** and click **Download**.
   - If it does not appear, open the ⋮ menu → **Custom repositories**, add `https://github.com/EnzoD86/tuya-smart-ir-ac` with category **Integration**, then search again.
3. **Restart** Home Assistant.

[![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=EnzoD86&repository=tuya-smart-ir-ac&category=integration)

### Option 2: Manual

1. Download the latest [release](https://github.com/EnzoD86/tuya-smart-ir-ac/releases/latest).
2. Copy the `custom_components/tuya_smart_ir_ac` folder into your Home Assistant `config/custom_components/` folder.
3. **Restart** Home Assistant.

## Quick start

1. **Prepare the Tuya Cloud project** and collect your **Access ID**, **Access Secret**, data center and device IDs. Follow [Tuya Cloud setup](docs/tuya-cloud-setup.md).
2. **Add the hub** in Home Assistant: **Settings → Devices & services → Add integration → Tuya Smart IR AC**. Enter your credentials and data center.

   [![Add Integration to your Home Assistant instance.](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=tuya_smart_ir_ac)

3. **Add your devices**: on the integration card click **Configure → Device Management** and choose:
   - **Air Conditioner Management → Add a new air conditioner**: enter the IR blaster ID and the AC remote ID.
   - **Generic Devices Management → Add a new generic device**: enter the IR blaster ID and the remote ID.
   - **Temperature/Humidity Sensors Management → Add a new temperature/humidity sensor**: enter the sensor device ID.
4. *(Optional)* Enable **Tuya Pulsar** for real-time updates. See [Tuya Cloud setup → Pulsar](docs/tuya-cloud-setup.md#4-optional-enable-the-message-service-tuya-pulsar).

All options are described in [Configuration reference](docs/configuration.md).

## Documentation

| Page | What you will find |
|------|--------------------|
| [Tuya Cloud setup](docs/tuya-cloud-setup.md) | Creating the Tuya cloud project, linking the app, finding IDs, enabling Pulsar |
| [Configuration reference](docs/configuration.md) | Every hub, air conditioner, generic device, sensor and preset option |
| [Entities](docs/entities.md) | All entities created for each device type |
| [How the climate entity behaves](docs/climate-behavior.md) | Built-in logic: power-on handling, default temperatures and fan speeds, Dry mode, presets, state sync |
| [Automation examples](docs/examples.md) | Ready-to-use automations and scripts |
| [Troubleshooting](docs/troubleshooting.md) | Error messages, debug logging, common problems |
| [Architecture](docs/architecture.md) | Internal design, for contributors |

## Known limitations

- **No feedback from the AC**: infrared is one-way. Changes made with the physical remote are not detected, and Home Assistant cannot confirm that the AC received a command.
- **Cloud dependency**: an Internet connection and the Tuya Cloud are required. There is no local control.
- **Supported data centers**: only Central Europe (`eu`), Western America (`us`), India (`in`), China (`cn`) and Singapore (`sg`).
- **Celsius only for ACs**: AC target temperatures are always in °C, between 16 °C and 30 °C.
- **Fixed sets of modes**: HVAC modes are Auto, Cool, Heat, Dry and Fan only. Fan speeds are Auto, Low, Medium and High. Swing and other special functions are not available on the climate entity (add the AC as a [generic device](docs/configuration.md#generic-device-options) too if you need its extra keys as buttons).
- **Tuya API quotas**: the free Tuya IoT Core plan has call limits and must be renewed periodically. See [Troubleshooting](docs/troubleshooting.md#tuya-subscription-expired).

## Removing the integration

1. *(Optional)* Remove individual devices first with **Configure → Device Management → … → Remove**.
2. Go to **Settings → Devices & services → Tuya Smart IR AC**, open the ⋮ menu of the hub and select **Delete**.
3. If you installed with HACS, remove the repository from HACS. If you installed manually, delete `config/custom_components/tuya_smart_ir_ac`.
4. Restart Home Assistant.
5. *(Optional)* Delete the cloud project in the Tuya IoT Platform or unlink your app account from it.

> If you used the **Custom power-on button** option, or automations reference these entities, update them afterwards.

## Contributing

Contributions are welcome! Open an issue or a pull request and we will figure out together how to implement the improvement. Before you start, read the [Architecture](docs/architecture.md) page. Translations live in [`custom_components/tuya_smart_ir_ac/translations/`](custom_components/tuya_smart_ir_ac/translations/).

## Support me

I develop and support this integration in my free time. If you appreciate my work and want to support me, you can buy me a coffee. Thanks!

<a href="https://www.buymeacoffee.com/enzod86" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

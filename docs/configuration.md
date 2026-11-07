# Configuration reference

All configuration is done in the Home Assistant UI. There is **no YAML** to write. This page describes every field in the configuration dialogs and **exactly what it changes in the integration's behavior**.

← Back to [README](../README.md)

- [Concepts: hub vs. IR blaster](#concepts-hub-vs-ir-blaster)
- [Hub settings](#hub-settings)
- [Options menu overview](#options-menu-overview)
- [Air conditioner options](#air-conditioner-options)
  - [Identification](#identification-add-only)
  - [Sensors](#sensors)
  - [Temperature range](#temperature-range)
  - [Supported modes](#supported-modes)
  - [HVAC automatic presets](#hvac-automatic-presets)
  - [Optional entities](#optional-entities)
  - [Compatibility options](#compatibility-options)
- [Global presets](#global-presets)
- [Generic device options](#generic-device-options)
- [Temperature/humidity sensor options](#temperaturehumidity-sensor-options)
- [Applying changes](#applying-changes)
- [Legacy YAML configuration](#legacy-yaml-configuration)

---

## Concepts: hub vs. IR blaster

| Term | What it is |
|------|------------|
| **Hub** (integration entry) | One entry on **Settings → Devices & services → Tuya Smart IR AC**. Each hub is **one Tuya cloud project** (one Access ID). It is titled `Tuya Smart IR Hub (<access_id>)`. |
| **Infrared hub / IR blaster** | The physical Tuya IR device that emits the infrared signals. Its ID is the **Infrared hub ID**. |
| **Air conditioner / generic device** | A **remote** created under the IR blaster in the Smart Life app. Its ID is the **Air conditioner ID** / **Device ID**. |

One hub can manage **many** IR blasters and devices. You need a second hub only if you use a second Tuya cloud project (a different Access ID). A device can belong to **only one hub**.

---

## Hub settings

Shown when you add the integration, and later under **Configure → Hub Settings**. The same dialog is also used for **re-authentication** when Tuya rejects the stored credentials.

<!-- screenshot: images/hub-settings.png — "Hub Settings" dialog (English UI) -->

| Field | Default | Range | What it does |
|-------|---------|-------|--------------|
| **Tuya access ID** | (none) | (none) | Access ID of the Tuya cloud project. Used as the hub's unique ID, so the same project cannot be added twice. **It cannot be changed afterwards** (error *"The Access ID cannot be modified for an existing hub."*). To use another project, add a new hub. |
| **Tuya access secret** | (none) | (none) | Access Secret of the project. Changing it reloads the whole hub. |
| **Tuya country API** | (none) | Europe, America, India, China, Singapore | Selects the Tuya API and Pulsar endpoints. Must match the project's data center ([table](tuya-cloud-setup.md#choosing-the-data-center)). Changing it reloads the whole hub. |
| **Air conditioner update interval** | `300` s | 10–3600 s | How often the state of **all** air conditioners is fetched from Tuya. It uses one batch request for all ACs of the hub. |
| **Temperature/humidity update interval** | `300` s | 10–3600 s | How often temperature/humidity sensors are fetched (one request per sensor, at most 3 in parallel). With Pulsar enabled, a sensor is **skipped** when all its values were already received through Pulsar during the last interval (see [smart polling](architecture.md#smart-polling-for-sensors)). |
| **Enable Tuya Pulsar Stream** | Off | (none) | Opens a WebSocket to Tuya's message service to receive state changes in real time. It requires the [Message Service setup](tuya-cloud-setup.md#4-optional-enable-the-message-service-tuya-pulsar). Toggling it reloads the whole hub. |

**Validation when you save:** the integration logs in to Tuya with the given credentials and reads the account info. Possible errors:

| Message | Cause |
|---------|-------|
| *Authentication failed: Invalid Access ID or Access Secret.* | Tuya refused the login. |
| *Authentication failed: Invalid data center.* | The login worked, but the project is in a different data center than the one selected (Tuya code `28841107`). |
| *Failed to connect to Tuya servers. Please check your network.* | The request failed (network, DNS, timeout). |

> 💡 **Lower intervals = more API calls.** The free Tuya plan has a monthly quota. 300 s is a safe default, especially with Pulsar enabled.

---

## Options menu overview

After the hub is added, click **Configure** on the integration card:

```
Tuya Smart IR Configuration
├── Device Management
│   ├── Air Conditioner Management ─ Add / Edit / Remove
│   ├── Generic Devices Management ─ Add / Remove
│   └── Temperature/Humidity Sensors Management ─ Add / Remove
├── Global Presets Management ─ choose preset + mode → set temperature and fan speed
└── Hub Settings
```

<!-- screenshot: images/options-menu.png — "Tuya Smart IR Configuration" main menu -->

---

## Air conditioner options

Shown in **Add a new air conditioner** and **Edit an existing air conditioner**. All fields except identification can be edited later.

<!-- screenshot: images/add-climate.png — "Add Air Conditioner" dialog -->

### Identification (add only)

| Field | What it does |
|-------|--------------|
| **Infrared hub ID** | ID of the physical IR blaster that will send the commands. |
| **Air conditioner ID** | ID of the AC remote created under the blaster. |
| **Air conditioner name** | Name of the Home Assistant device. The climate entity uses this name, and the other entities are prefixed with it. |

When you click **Submit**, the integration asks Tuya for the current state of this AC (`GET /v2.0/infrareds/{infrared_id}/remotes/{climate_id}/ac/status`). If that fails, the dialog shows *Tuya Cloud Error: Code …: …* and nothing is saved. It also refuses an ID that is already used by any device on this hub or another hub.

> To change the IDs of an existing AC, remove it and add it again.

### Sensors

| Field | Default | What it does |
|-------|---------|--------------|
| **Temperature sensor** | (none) | Any Home Assistant `sensor` with device class *temperature*. Its value becomes the climate's **current temperature** (converted to °C if the sensor uses another unit). Without it, the climate entity has no current temperature. |
| **Humidity sensor** | (none) | Any `sensor` with device class *humidity*. Its value becomes the climate's **current humidity**. |

These are **display only**: the integration does **not** regulate the AC based on them (no thermostat logic). The AC's own thermostat does that. You can use them in your own automations ([examples](examples.md)).

### Temperature range

| Field | Default | Range | What it does |
|-------|---------|-------|--------------|
| **Minimum temperature** | `16` | 16–30 | Lowest target temperature in the UI. When you switch HVAC mode and the current target is **below** this value, the target is raised to it. |
| **Maximum temperature** | `30` | 16–30 | Highest target temperature in the UI. |
| **Step temperature** | `1.0` | 0.1–1.0 | Increment of the target temperature in the climate card and in the companion number entities. |

The range also applies to the [default temperature](#hvac-automatic-presets) and [temperature control](#optional-entities) number entities.

### Supported modes

| Field | Default | Options | What it does |
|-------|---------|---------|--------------|
| **Supported HVAC modes** | all | Auto, Cool, Heat, Dry, Fan only | Modes offered by the climate entity and the *HVAC mode* select. **Off is always added automatically.** If you clear the list, every mode is enabled. |
| **Supported FAN modes** | all | Auto, Low, Medium, High | Fan speeds offered by the climate entity and the fan selects. If you clear the list, every speed is enabled. |
| **Supported preset modes** | none | Home, Sleep, Eco, Comfort, Away | Presets offered by the climate entity. When at least one is selected, **None** is added and the climate gains *preset mode* support. Each preset applies the temperature and fan speed defined in [Global presets](#global-presets). |

### HVAC automatic presets

These options are named "presets", but they are **per-device defaults that are applied automatically when you change HVAC mode**. They are not climate preset modes.

| Option | Entities created (category *Configuration*) | Effect |
|--------|------------------------------------------------|--------|
| **Default temperature** | Three number sliders: **Auto**, **Cool**, **Heat** | When you **switch to** Auto, Cool or Heat, the target temperature becomes the value of the matching slider instead of keeping the current one. |
| **Default fan speed** | One select: **Fan speed** | When you **switch HVAC mode** or **change the target temperature**, the fan speed is set to this value (except in Dry mode, see below). Presets keep using their own fan speed. |

Details:

- The slider values are stored in Home Assistant (restored after a restart). The first time, each slider starts at the midpoint of min/max (for example, 23 °C for 16–30), and the fan select starts at the first supported fan speed.
- Moving a slider or changing the select **does not send anything** to the AC. It only changes what will be used for the next mode change.
- Dry and Fan only have no default temperature: switching to them keeps the current target (or 16 °C in Dry, see [Compatibility options](#compatibility-options)).

> ⚠️ With **Default fan speed** enabled, **every temperature change also resets the fan speed** to the default. If you change the fan manually and then change the temperature, the fan returns to the default. Disable this option if you do not want that.

See the full rules in [How the climate entity behaves](climate-behavior.md).

<!-- screenshot: images/climate-config-entities.png — device page, "Configuration" card with Auto / Cool / Heat sliders and Fan speed -->

### Optional entities

Extra entities attached to the AC device. They mirror the climate entity and are useful for dashboards, voice assistants, HomeKit, or automations that cannot use `climate` attributes directly.

| Option | Entity | Behavior |
|--------|--------|----------|
| **Temperature sensor** | `sensor` *Temperature* | Copy of the linked **Temperature sensor** (in its original unit). Created only if a temperature sensor is set. |
| **Humidity sensor** | `sensor` *Humidity* | Copy of the linked **Humidity sensor**. Created only if a humidity sensor is set. |
| **HVAC control** | `select` *HVAC mode* | Shows the current HVAC mode. Changing it is the same as changing the mode on the climate entity. |
| **Fan control** | `select` *Fan speed* | Shows the current fan speed. Changing it is the same as changing the fan on the climate entity. |
| **Temperature control** | `number` *Temperature* | Shows the target temperature. Changing it is the same as setting the temperature on the climate entity. |

> ℹ️ In the **logbook / activity**, these mirror entities change **together with** the climate entity, but they show *"No cause was recorded for this activity"*. That is expected: they only reflect the climate's state. Open the **climate entity's** activity to see what triggered the change. See [Troubleshooting → Who turned my AC on?](troubleshooting.md#who-turned-my-ac-on-or-changed-it).

### Compatibility options

A collapsed section for ACs that do not react correctly to the standard commands. **Leave the defaults unless something does not work.**

<!-- screenshot: images/compatibility-options.png — "Compatibility options" section expanded -->

Background: for every change (mode, temperature, fan, preset, turn on), the integration sends Tuya one **scene command** with the full state: `power=on, mode, temperature, fan`. Most ACs turn on when they receive it. Some ACs ignore the "power on" part of the scene and need a separate **power** command first. The options below control when that extra command is sent.

| Field | Default | Options | What it does |
|-------|---------|---------|--------------|
| **Force power on when setting hvac mode** | Never | Never / Always / Only if turned off | Sends a separate *power on* before the scene when you **change HVAC mode**, **select a preset**, or **turn the AC on**. |
| **Force power on when setting temperature** | Never | Never / Always / Only if turned off | Same, when you **change the target temperature**. |
| **Force power on when setting fan speed** | Never | Never / Always / Only if turned off | Same, when you **change the fan speed**. |
| **Set minimum temperature in dry mode** | Off | On/Off | When switching to **Dry**, sends **16 °C** as target (the integration's absolute minimum, not your *Minimum temperature*). When off, Dry keeps the current target temperature. |
| **Set minimum fan speed in dry mode** | Off | On/Off | When switching to **Dry** (or changing temperature while in Dry), sends fan **Low**. When off, Dry always uses fan **Auto**. |
| **Custom power-on button** | (none) | any `button` entity | Use **only if the standard power-on does not work**. This button is **pressed whenever the AC must be turned on** (on every *turn on*, and whenever a "force power on" option above applies). The scene command is sent right after it. Typically it is a key of a [generic device](#generic-device-options) learned from the original remote. |

The three power-on options:

| Value | Separate *power on* is sent… |
|-------|------------------------------|
| **Never** | never; only the scene command is sent. |
| **Always** | before every command of that type, even if the AC is already on. Some ACs **toggle** power on each "power" signal. If yours does, do not use *Always*. |
| **Only if turned off (not recommended)** | only when Home Assistant believes the AC is **off**. That belief can be wrong if the physical remote was used (IR gives no feedback), so it is not recommended. |

Decision tables are in [How the climate entity behaves](climate-behavior.md#power-on-handling).

---

## Global presets

**Configure → Global Presets Management.** Defines the temperature and fan speed for each preset, **for the whole hub** (shared by all its ACs). Choose a **Preset** and a **Mode** (Cool or Heat), then set **Target Temperature** (16–30, step 1) and **Fan Speed**.

<!-- screenshot: images/global-presets.png — "Preset Configuration" dialog -->

Built-in defaults (used until you change them):

| Preset | Cool | Heat |
|--------|------|------|
| Home | 26 °C, Auto | 21 °C, Auto |
| Sleep | 27 °C, Low | 20 °C, Low |
| Eco | 27 °C, Medium | 20 °C, Medium |
| Comfort | 25 °C, Medium | 22 °C, Medium |
| Away | 28 °C, Low | 19 °C, Low |

Notes:

- Presets are defined **only for Cool and Heat**. In Auto, Dry and Fan only, the climate offers only **None**.
- To use a preset on an AC, also enable it in that AC's **Supported preset modes**.
- Saving a preset takes effect immediately. The ACs are not reloaded and nothing is sent to them.
- Preset temperatures are **not limited** by each AC's min/max.

How presets are applied and detected: [How the climate entity behaves → Presets](climate-behavior.md#presets).

---

## Generic device options

**Device Management → Generic Devices Management → Add a new generic device.** Use this for any remote created under the IR blaster (TV, fan, projector, sound bar, or even an AC whose extra keys you want).

<!-- screenshot: images/add-generic.png — "Add Generic Device" dialog -->

| Field | What it does |
|-------|--------------|
| **Infrared hub ID** | ID of the IR blaster. |
| **Device ID** | ID of the remote. |
| **Device name** | Name of the Home Assistant device and prefix of each button name. |

When you add the device, and every time the hub is loaded, the integration downloads the remote's **key list** from Tuya and creates **one `button` per key** (for example, `TV Power`, `TV Volume Up`). Pressing a button sends that key.

- Keys you add or rename in the Smart Life app appear after the hub is reloaded.
- If the key list cannot be downloaded at startup, the device's buttons are skipped (a warning is logged). Reload the hub when Tuya is reachable again.
- Generic devices can only be added or removed, not edited.

---

## Temperature/humidity sensor options

**Device Management → Temperature/Humidity Sensors Management → Add a new temperature/humidity sensor.** This is for Tuya devices that report temperature and/or humidity, such as standalone sensors or IR blasters with a built-in sensor.

| Field | What it does |
|-------|--------------|
| **Device ID** | Tuya ID of the device that reports the values. |
| **Device name** | Name of the Home Assistant device. |

When you add the device, the integration reads its properties (`GET /v2.0/cloud/thing/{device_id}/shadow/properties`) and **detects automatically** which entities to create:

| Tuya property (DP) codes | Entity created |
|--------------------------|----------------|
| `temp_current`, `va_temperature` | Temperature (the value is divided by 10) |
| `humidity_value`, `va_humidity`, `humidity_current` | Humidity (%) |
| `battery_state` (`high`/`middle`/`low`) | Battery: 100 / 50 / 10 % (diagnostic) |
| `temp_unit_convert` (`c`/`f`) | Sets the unit of the temperature entity |

If the device reports neither temperature nor humidity, it is rejected with *The selected device is not a valid temperature/humidity sensor.* Detection happens **only when the sensor is added**. To detect again, remove the sensor and add it again.

---

## Applying changes

- **Adding, editing or removing** devices: the integration unloads its entities, **removes the hub's devices from the device registry**, and creates them again with the new options. The Tuya connection (and Pulsar) is kept open.
- **Hub settings**: changing the secret, data center or Pulsar setting **reloads the whole hub**.
- **Global presets**: applied immediately, without a reload.

---

## Legacy YAML configuration

Older versions used YAML. If this block is still in `configuration.yaml`, it is **imported** into a hub entry at startup:

```yaml
tuya_smart_ir_ac:
  access_id: "xxxxxxxx"
  access_secret: "xxxxxxxx"
  country: "eu"
  update_interval: 300  # optional, becomes the air conditioner update interval
```

Devices from older versions, which were separate integration entries, are **migrated automatically** into the hub. A notification titled **"Tuya Smart IR AC: Migration Completed"** then asks you to delete the YAML block and restart. New installations should not use YAML.

---

Next: [Entities](entities.md) · [How the climate entity behaves](climate-behavior.md)

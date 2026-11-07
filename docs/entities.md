# Entities

The entities each device type creates. Entity IDs are derived from the names you choose. The examples below assume an AC named **Living Room AC**, a generic device named **TV**, and a sensor named **Bedroom Sensor**. Names are shown in English and follow your Home Assistant language where a translation exists.

← Back to [README](../README.md)

---

## Air conditioner

Device model: **IR Air Conditioning**, manufacturer **Tuya**.

<!-- screenshot: images/climate-device-page.png — AC device page showing Controls, Sensors and Configuration cards -->

### Always created

| Entity | Example ID | Description |
|--------|------------|-------------|
| Climate | `climate.living_room_ac` | Main entity. HVAC modes, target temperature, fan modes, presets (if configured), current temperature/humidity (if sensors are linked). Supports `turn_on` / `turn_off`. |

**Climate features:** turn on, turn off, target temperature, fan mode, and preset mode (only if at least one preset is selected).

**Climate attributes worth knowing:**

| Attribute | Source |
|-----------|--------|
| `hvac_mode` | `off` when Tuya reports power off, otherwise the mode stored by Tuya |
| `temperature` | Target temperature stored by Tuya (°C) |
| `fan_mode` | Fan speed stored by Tuya |
| `preset_mode` | Recognized from temperature + fan (see [Presets](climate-behavior.md#presets)) |
| `current_temperature` / `current_humidity` | Linked Home Assistant sensors |

### Created by *HVAC automatic presets* (category: Configuration)

| Option | Entity | Example ID | Description |
|--------|--------|------------|-------------|
| Default temperature | Number **Auto** | `number.living_room_ac_auto` | Target used when switching to Auto |
| Default temperature | Number **Cool** | `number.living_room_ac_cool` | Target used when switching to Cool |
| Default temperature | Number **Heat** | `number.living_room_ac_heat` | Target used when switching to Heat |
| Default fan speed | Select **Fan speed** | `select.living_room_ac_fan_speed` | Fan used on mode and temperature changes |

These entities only store values in Home Assistant (restored after restarts). Changing them sends nothing to the AC. See [HVAC automatic presets](configuration.md#hvac-automatic-presets).

### Created by *Optional entities*

| Option | Entity | Example ID | Description |
|--------|--------|------------|-------------|
| Temperature sensor | Sensor **Temperature** | `sensor.living_room_ac_temperature` | Mirror of the linked temperature sensor |
| Humidity sensor | Sensor **Humidity** | `sensor.living_room_ac_humidity` | Mirror of the linked humidity sensor |
| HVAC control | Select **HVAC mode** | `select.living_room_ac_hvac_mode` | Reads and sets the HVAC mode |
| Fan control | Select **Fan speed** | `select.living_room_ac_fan_speed_2`¹ | Reads and sets the fan speed |
| Temperature control | Number **Temperature** | `number.living_room_ac_temperature` | Reads and sets the target temperature |

¹ The *Default fan speed* select (Configuration) and the *Fan control* select have the same name, **Fan speed**. If both are enabled, Home Assistant adds a `_2` suffix to the second one. Check which is which in the device page: the default one is in the **Configuration** card.

## Generic device

Device model: **IR Remote Control**.

| Entity | Example ID | Description |
|--------|------------|-------------|
| One **button** per remote key | `button.tv_power`, `button.tv_volume_up` | Pressing it sends that IR key. The name is `<device name> <key name>` as defined in Tuya. |

Buttons have no state other than the time of the last press.

## Temperature/humidity sensor

Device model: **T & H Sensor**. Entities are created only if the device reported the corresponding value when it was added ([detection](configuration.md#temperaturehumidity-sensor-options)).

| Entity | Example ID | Unit | Notes |
|--------|------------|------|-------|
| Temperature | `sensor.bedroom_sensor_temperature` | °C or °F (from the device) | |
| Humidity | `sensor.bedroom_sensor_humidity` | % | |
| Battery | `sensor.bedroom_sensor_battery` | % | Diagnostic. Coarse values: 100 (high), 50 (middle), 10 (low) |

These entities become **unavailable** when Tuya cannot be reached.

---

## Unique IDs

Stable identifiers, useful if you need to find entities in the registry:

| Entity | Unique ID |
|--------|-----------|
| Climate | `<infrared_id>_<climate_id>` |
| AC sub-entities | `<infrared_id>_<climate_id>_<type>` (`temp_hvac_mode_cool`, `fan_hvac_mode`, `hvac_mode`, `fan_mode`, `temp_setpoint`, `current_temperature`, `current_humidity`) |
| Button | `<infrared_id>_<device_id>_<key_id>` |
| Sensor | `<device_id>_<temperature\|humidity\|battery>` |

---

Next: [How the climate entity behaves](climate-behavior.md)

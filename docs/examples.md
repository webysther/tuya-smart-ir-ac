# Automation examples

Copy these into **Settings → Automations & scenes → Create automation → ⋮ → Edit in YAML**, then adapt the entity IDs. The examples use the entities from [Entities](entities.md) (`climate.living_room_ac`, and so on).

← Back to [README](../README.md)

> Remember: every command resends the **full state** and **turns the AC on**. See [Key facts](climate-behavior.md#key-facts). To change something only when the AC is already on, add a condition, as in the examples below.

---

## Turn on cooling when the room gets hot

Uses the temperature sensor linked to the AC (exposed as the climate's `current_temperature`).

```yaml
alias: "Living room: cool when hot"
triggers:
  - trigger: numeric_state
    entity_id: climate.living_room_ac
    attribute: current_temperature
    above: 28
    for: "00:10:00"
conditions:
  - condition: state
    entity_id: climate.living_room_ac
    state: "off"
  - condition: state
    entity_id: person.me
    state: home
actions:
  - action: climate.set_hvac_mode
    target:
      entity_id: climate.living_room_ac
    data:
      hvac_mode: cool
mode: single
```

If **Default temperature** is enabled, the *Cool* slider decides the target temperature. Otherwise, use `climate.set_temperature` with `hvac_mode: cool` and `temperature: 24` to set both in one command.

## Switch to the Sleep preset at night (only if already on)

```yaml
alias: "Living room: sleep preset at 23:00"
triggers:
  - trigger: time
    at: "23:00:00"
conditions:
  - condition: not
    conditions:
      - condition: state
        entity_id: climate.living_room_ac
        state: "off"
actions:
  - action: climate.set_preset_mode
    target:
      entity_id: climate.living_room_ac
    data:
      preset_mode: sleep
```

Presets work only in Cool and Heat. Customize the Sleep values in [Global presets](configuration.md#global-presets).

## Turn off when a window stays open

```yaml
alias: "Living room: AC off when window open"
triggers:
  - trigger: state
    entity_id: binary_sensor.living_room_window
    to: "on"
    for: "00:02:00"
conditions:
  - condition: not
    conditions:
      - condition: state
        entity_id: climate.living_room_ac
        state: "off"
actions:
  - action: climate.turn_off
    target:
      entity_id: climate.living_room_ac
```

## Seasonal default temperatures

Change the per-mode defaults (requires **Default temperature**). This sends nothing to the AC. It only changes the target used on the next mode change.

```yaml
alias: "AC: summer defaults"
triggers:
  - trigger: time
    at: "06:00:00"
conditions:
  - condition: template
    value_template: "{{ now().month in [12, 1, 2] }}"  # southern-hemisphere summer
actions:
  - action: number.set_value
    target:
      entity_id: number.living_room_ac_cool
    data:
      value: 24
```

## Press a key of a generic remote

```yaml
alias: "TV: turn on with the evening scene"
triggers:
  - trigger: sun
    event: sunset
actions:
  - action: button.press
    target:
      entity_id: button.tv_power
```

> Many IR devices use a single **toggle** key for power, so pressing *Power* twice turns the device off again. Home Assistant cannot know the TV's real state.

## Low battery notification (T&H sensor)

```yaml
alias: "Bedroom sensor: low battery"
triggers:
  - trigger: numeric_state
    entity_id: sensor.bedroom_sensor_battery
    below: 11
actions:
  - action: notify.notify
    data:
      message: "Bedroom sensor battery is low."
```

## Script: resync after using the physical remote

The physical remote is not detected. This script resends the state Home Assistant believes is correct, so the AC and HA agree again.

```yaml
alias: "Living room AC: resend state"
sequence:
  - if:
      - condition: state
        entity_id: climate.living_room_ac
        state: "off"
    then:
      - action: climate.turn_off
        target:
          entity_id: climate.living_room_ac
    else:
      - action: climate.set_fan_mode   # resends mode + temperature + fan as they are
        target:
          entity_id: climate.living_room_ac
        data:
          fan_mode: "{{ state_attr('climate.living_room_ac', 'fan_mode') }}"
```

---

Next: [Troubleshooting](troubleshooting.md)

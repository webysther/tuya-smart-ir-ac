# Troubleshooting

← Back to [README](../README.md)

- [Who turned my AC on (or changed it)?](#who-turned-my-ac-on-or-changed-it)
- [Enable debug logging](#enable-debug-logging)
- [Error messages in the configuration dialogs](#error-messages-in-the-configuration-dialogs)
- [Errors when controlling a device](#errors-when-controlling-a-device)
- [Common problems](#common-problems)
- [Reporting an issue](#reporting-an-issue)

---

## Who turned my AC on (or changed it)?

**This integration never sends commands by itself.** It has no timers, schedules or thermostat logic. If the AC changed, the change came from one of these:

| What the climate entity's activity shows | Origin |
|------------------------------------------|--------|
| A **user** name | Someone used the UI or the companion app |
| An **automation** or **script** | That automation/script |
| *Home Assistant Core Integration*, with action **Generic turn on** / **Generic turn off** | A `homeassistant.turn_on` / `turn_off` / `toggle` call with no user attached. Typical sources are a scene, an action that targets a whole area/floor/label, or an external tool that calls the Home Assistant API (Node-RED, a voice bridge, etc.). Search your automations, scripts and scenes for this entity and for the area it belongs to. |
| Another integration (Google Assistant, Alexa, HomeKit, …) | That voice assistant / bridge |
| *No cause recorded* | The state was **received from Tuya** (polling or Pulsar): a change made in the **Smart Life app**, a **Tuya scene/schedule**, or another Tuya client |

<!-- screenshot: images/activity-climate.png — climate activity details showing the cause -->

Notes:

- The *HVAC mode*, *Fan speed* and *Temperature* [optional entities](configuration.md#optional-entities) are **mirrors** of the climate entity. Their activity **always** shows *"No cause was recorded for this activity"*, even when the climate's own entry shows the real cause. **Always check the climate entity.**
- The options (default temperature, default fan, power-on, …) only decide **what** is sent when an action happens. They never trigger an action. See [How the climate entity behaves](climate-behavior.md).
- Also check the **Smart Life app** for schedules, timers or "Tap-to-Run / Automation" scenes that control the AC. They run in the Tuya cloud and appear in Home Assistant with no cause.

## Enable debug logging

Add to `configuration.yaml` and restart:

```yaml
logger:
  default: info
  logs:
    custom_components.tuya_smart_ir_ac: debug   # integration: commands, polling, config flow
    custom_components.tuya_connector: debug     # Tuya HTTP client and Pulsar WebSocket
```

Or, without a restart: **Settings → Devices & services → Tuya Smart IR AC → ⋮ → Enable debug logging**. Reproduce the problem, then **Disable debug logging** to download the log. That second method captures only `custom_components.tuya_smart_ir_ac`. Use the YAML method if you also need Pulsar/HTTP details.

Useful log lines:

| Log contains | Meaning |
|--------------|---------|
| `Sending combined IR packet -> Mode: …, Temp: …, Fan: …` | The scene command sent for a change |
| `Sending IR command to turn ON/OFF climate` | A separate power command |
| `Triggering custom power-on button` | The custom power-on button was pressed |
| `Update climate from Pulsar` / `Update sensor from Pulsar` | A real-time update arrived |
| `Keeping fallback cache` | AC polling failed; last state kept |
| `Tuya Pulsar data stream successfully verified and active` | Pulsar is receiving messages |

Sensitive values (tokens, uid, IP, …) are masked in the connector logs.

## Error messages in the configuration dialogs

| Message | Cause and fix |
|---------|---------------|
| *Authentication failed: Invalid Access ID or Access Secret.* | Check the credentials on the Tuya project **Overview** tab. |
| *Authentication failed: Invalid data center.* | The selected country does not match the project's data center. See [Choosing the data center](tuya-cloud-setup.md#choosing-the-data-center). |
| *Failed to connect to Tuya servers. Please check your network.* | Home Assistant cannot reach the Tuya API (Internet, DNS, firewall). |
| *The Access ID cannot be modified for an existing hub.* | Add a new hub for another project instead. |
| *This device is already configured.* | The ID is already used by a device of this hub. |
| *This device is already configured on other Hub.* | The ID is used by another hub. Remove it there first. |
| *The selected device is not a valid temperature/humidity sensor.* | The device reports neither temperature nor humidity. Check the device ID. |
| *Tuya Cloud Error: Code …: …* | Tuya rejected the request. Common causes: the device is not linked to the project, IR Control Hub Open Service is not authorized, wrong IR blaster ID vs. remote ID, or the [subscription expired](#tuya-subscription-expired). |
| *No devices found.* | There is no device of that type to edit or remove. |

## Errors when controlling a device

| Message | Meaning |
|---------|---------|
| *Unable to turn on* / *Unable to turn off* | Tuya rejected the power command. |
| *Unable to change hvac mode* | Tuya rejected the **scene command**. All changes use it (mode, temperature, fan, preset, turn on), so this message can also appear when you changed the temperature or the fan. |
| *Unable to send command* | Tuya rejected a generic button press. |

The exact Tuya error is in the log (`Failed to …: Code …`).

## Common problems

### The AC does not turn on (but changes mode/temperature when it is already on)

The AC ignores the power-on included in the scene command. In the AC's **Compatibility options**, set **Force power on when setting hvac mode** to **Always**, and the temperature/fan options as well if needed. If the AC then turns **off** on some commands, it uses a toggle power signal. Use **Only if turned off** instead, or configure a **Custom power-on button** (learn the remote's power key as a generic device). See [Power-on handling](climate-behavior.md#power-on-handling).

### Changing the temperature also changed the fan speed

Expected when **Default fan speed** is enabled or the AC is in **Dry** mode. See [Fan speed rules](climate-behavior.md#fan-speed-rules).

### Switching mode changed my temperature

Expected when **Default temperature** is enabled (it uses the Auto/Cool/Heat sliders), or in Dry with *minimum temperature in dry mode*. See [Target temperature rules](climate-behavior.md#target-temperature-rules).

### The state in HA is wrong after using the physical remote

IR is one-way, so the physical remote cannot be detected. Use Home Assistant or the Smart Life app instead, or send any command to resync ([script example](examples.md#script-resync-after-using-the-physical-remote)).

### Changes made in the Smart Life app take minutes to appear

Without Pulsar, they appear at the next poll (300 s by default). [Enable Pulsar](tuya-cloud-setup.md#4-optional-enable-the-message-service-tuya-pulsar), or lower the update interval (this uses more API quota).

### "Tuya Pulsar Stream Inactive" notification

About 60 s after startup, the integration checks whether at least one message arrived on the Pulsar stream. If none did:

1. Verify the [Message Service setup](tuya-cloud-setup.md#4-optional-enable-the-message-service-tuya-pulsar), especially the **Production** environment and the four BizCodes.
2. Disable and re-enable the BizCodes to force Tuya to rebuild the routing.
3. The check only looks for **any** message. In a quiet house where no device changed during that first minute, the notification can appear even though Pulsar works. If updates from the app arrive quickly afterwards, you can dismiss it.

The connection reconnects automatically (backoff up to 60 s).

### Tuya subscription expired

The free **IoT Core** plan expires periodically. All API calls then fail, and the logs show errors such as *"No permissions. Your subscription to cloud development plan has expired."* Renew it in the Tuya IoT Platform under **Cloud → Cloud Services → IoT Core** (**Extend trial**).

### Generic device has no buttons

The key list could not be downloaded at startup. Check the log for `Failed to load keys configuration`, then reload the integration.

### Default temperature sliders / fan select have no effect

They only apply when **switching HVAC mode** (and, for the fan, when changing temperature). Turning on resends the previous state, and the presets use their own values.

## Reporting an issue

Open an issue at <https://github.com/EnzoD86/tuya-smart-ir-ac/issues> with:

- Integration version and Home Assistant version.
- What you did, what you expected, and what happened.
- Debug logs ([how](#enable-debug-logging)). **Remove your Access ID, Access Secret and device IDs** before posting.

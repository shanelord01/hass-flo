 Flo by Moen Integration for Home Assistant

[![CurrentVersion](https://img.shields.io/badge/Current_Version-v2.0.0--rc.1-orange.svg)](https://github.com/shanelord01/hass-flo/releases)   [![Beta](https://img.shields.io/badge/Released-March,_2026-red.svg)](https://github.com/shanelord01/hass-flo/releases)   [![HACS](https://img.shields.io/badge/HACS-Custom_Repository-darkorange.svg)](https://github.com/shanelord01/hass-flo)

[![Type](https://img.shields.io/badge/Type-Custom_Component-forestgreen.svg)](https://github.com/shanelord01/hass-flo)   [![HA](https://img.shields.io/badge/Home_Assistant-2024.4+-forestgreen.svg)](https://www.home-assistant.io)   [![ProjectStage](https://img.shields.io/badge/Project_Stage-General_Availability-forestgreen.svg)](https://github.com/shanelord01/hass-flo)

Monitor and control your Flo by Moen smart water devices from Home Assistant.
An updated drop-in replacement for the core Flo integration, fixing a long-standing temperature unit bug and other code quality issues. Compatible with Flo Smart Water Shutoff valves and Flo Smart Water Detectors (pucks).

> **Replaces the built-in integration** — because `custom_components/` takes precedence over core,
> this installs over the default Flo integration with no migration, no entity ID changes, and no
> automation breakage.

---

## What's Fixed vs Core

| Issue | Core | This Integration |
|---|---|---|
| Temperature sensor reports ~99°F instead of ~52°F | ❌ Bug | ✅ Fixed |
| Water detector uses wrong binary sensor device class | ❌ `PROBLEM` | ✅ `MOISTURE` |
| Duplicate logger defined alongside shared logger | ❌ Present | ✅ Removed |
| Failure count shared across all device instances | ❌ Class-level | ✅ Per-instance |
| Orphaned config strings for non-existent `host` field | ❌ Present | ✅ Removed |

### Temperature Bug Detail

The Flo API field `tempF` returns values in **Celsius** despite its name — a Moen API change
that was never corrected in the core integration. This caused reported temperatures of
~99°F when the actual value was ~9.9°C (~49.8°F). Home Assistant handles the Celsius→Fahrenheit
conversion automatically once the native unit is declared correctly.

---

## Entities

### Flo Smart Water Shutoff (flo_device_v2)

| Entity | Type | Description |
|---|---|---|
| Shutoff valve | Switch | Open / close the water valve |
| Water flow rate | Sensor | Current flow in gallons per minute |
| Water pressure | Sensor | Current pressure in PSI |
| Water temperature | Sensor | Current water temperature (°C native, converts to user preference) |
| Today's water usage | Sensor | Cumulative daily consumption in gallons |
| Current system mode | Sensor | Home / Away / Sleep |
| Pending system alerts | Binary Sensor | True if any info, warning, or critical alerts are pending |

### Flo Smart Water Detector / Puck (puck_oem)

| Entity | Type | Description |
|---|---|---|
| Water detected | Binary Sensor | True if water is present (MOISTURE device class) |
| Temperature | Sensor | Ambient area temperature |
| Humidity | Sensor | Ambient humidity percentage |
| Battery | Sensor | Battery level percentage |

---

## Actions (Services)

| Service | Description |
|---|---|
| `flo.set_home_mode` | Set the location to Home mode |
| `flo.set_away_mode` | Set the location to Away mode |
| `flo.set_sleep_mode` | Set the location to Sleep mode for 120, 1440, or 4320 minutes |
| `flo.run_health_test` | Trigger a device health test |

Firewall valve control uses the native `switch.turn_on` / `switch.turn_off` services targeting
the shutoff valve switch entity — no custom service required.

### Set Sleep Mode

```yaml
action: flo.set_sleep_mode
target:
  entity_id: switch.flo_shutoff_valve
data:
  sleep_minutes: 120      # Options: 120, 1440, 4320
  revert_to_mode: home    # Options: home, away
```

### Run a Health Test

```yaml
action: flo.run_health_test
target:
  entity_id: switch.flo_shutoff_valve
```

---

## Installation

### Option 1 — HACS (Recommended)

HACS gives you one-click installs and automatic update notifications.

**If you don't have HACS yet:**
1. Follow the [HACS installation guide](https://hacs.xyz/docs/use/download/download/) to install it in Home Assistant.

**Add this repository to HACS:**
1. In Home Assistant, go to **HACS** in the sidebar
2. Click the three-dot menu (⋮) in the top-right corner
3. Select **Custom repositories**
4. In the **Repository** field paste:
   ```
   https://github.com/YOUR_GITHUB/hass-flo
   ```
5. Set **Type** to **Integration** and click **Add**
6. Search for **Flo** in HACS and click **Download**
7. Restart Home Assistant when prompted

### Option 2 — Manual

1. Download this repository as a ZIP (click **Code → Download ZIP** on GitHub)
2. Unzip it and copy the `custom_components/flo` folder into your Home Assistant
   `config/custom_components/` directory
   *(create `custom_components` if it doesn't exist)*
3. Restart Home Assistant

---

## Setup

If you already have the core Flo integration configured, **no action is needed** — your
existing config entry, entities, and automations carry over automatically.

If setting up for the first time after installing:

1. Go to **Settings → Devices & Services**
2. Click **+ Add Integration** and search for **Flo**
3. Enter your Flo **Username** (email address) and **Password**
4. Click **Submit**

---

## Clearing Old Temperature History

Because the core integration recorded temperatures under the wrong unit, your long-term
statistics graph will show a discontinuity after switching to this integration. This is
cosmetic only and does not affect entity function.

To clean it up:
1. Go to **Developer Tools → Statistics**
2. Find your Flo temperature sensor(s)
3. Click **Fix issue** or clear the history for that sensor

---

## Debug Logging

```yaml
# configuration.yaml
logger:
  default: info
  logs:
    custom_components.flo: debug
```

---

## Credits

- Original `aioflo` library: [bachya/aioflo](https://github.com/bachya/aioflo)
- Original core integration: [home-assistant/core](https://github.com/home-assistant/core/tree/dev/homeassistant/components/flo)
- Bug fixes and HACS packaging: [YOUR_GITHUB/hass-flo](https://github.com/shanelord01/hass-flo)

---

## Changelog

For full release history see [Releases](https://github.com/shanelord01/hass-flo/releases).

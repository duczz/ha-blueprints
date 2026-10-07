## 🏠 My collection of Home Assistant Blueprints

<details>
<summary>💨 Universal Fan Speed Control (Testing)</summary>

A Home Assistant Blueprint that controls fan speed based on room temperature — supports both **Percentage fans** (0–100%) and **Preset fans** (Named Modes), with separate profiles for cooling and heating.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/duczz/ha-blueprints/main/universal_fan_speed_control.yaml)

### Features

- **Universal**: works with any `fan.*` entity that supports `percentage` or `preset_mode`
- **Linear interpolation**: fan speed scales smoothly between min and max temperature
- **Dual mode**: separate temperature profiles for cooling and heating
- **Hysteresis**: stateful dead-band — a running fan stays on until the temperature leaves the hold band
- **Off offset**: separate threshold for fan shutdown to avoid flapping
- **Optional main entity**: e.g. `climate.*` — fan only runs while it is active; climate mode selects the profile
- **Enable/Disable toggle**: `input_boolean` helper to quickly disable the automation (e.g. away mode, sleep mode)
- **Always current values**: re-evaluates on temperature, toggle, main entity changes, Home Assistant start and once per minute; rate limiting never applies stale values

### Requirements

- Home Assistant 2024.10.0 or newer
- An `input_boolean` helper for the enable/disable toggle (create under **Settings → Helpers**)

### Inputs

**Required**

| Input | Description |
|---|---|
| 🌡 Temperatursensor | Temperature sensor entity |
| 💨 Fan Entity | The fan entity to control |
| 🔘 Enable/Disable Toggle | `input_boolean` to enable or disable the automation |

**Optional**

| Input | Description |
|---|---|
| 🏠 Haupt-Entity | e.g. `climate.*`. Fan only runs while it is not `off`; with `climate.*` the mode (cool/heat) selects the profile and the off-temperature/hysteresis do not apply |
| 🔄 Automatisch einschalten | Default `true`. Turns a stopped fan on again once the on-threshold (`off ± gap`) is reached |
| ⚡ Änderungsbegrenzung (nur Percentage) | `speed_limits_enabled`, minimum change and maximum change per step |
| Manuelle Stufen 1–6 | Temperature (`26.5` or `26,5`) → fixed preset/percentage; highest matching step wins |

**Steuerungsmodus (Control Mode)**

| Input | Description |
|---|---|
| Steuerungsmodus | `percentage` or `preset` |
| Preset-Reihenfolge | Comma-separated list of presets, slow → fast (e.g. `1 - Diffuse, 2 - Low, 3 - Medium, 4 - High, 5 - Focus`) |
| Niedrigster / Höchster Preset | Active preset range within the list |
| Minimale / Maximale Fan-Speed (%) | Speed range for percentage mode |

**Hysterese**

| Input | Default | Description |
|---|---|---|
| Hysterese | 1.0° | Dead-band around the threshold — prevents rapid cycling |
| Hysterese aktivieren | `true` | If disabled, Hysterese and Ausschalt-Offset are ignored |
| Ausschalt-Offset | 0° | Offset (−5…0) on the fan-off temperature: a running fan stays on until the temperature passes `off + offset` (cooling) / `off − offset` (heating). 0 = off exactly at the off-temperature |

**Kühlen (Cooling Profile)**

| Input | Default | Description |
|---|---|---|
| Kühlen aktivieren | `true` | Enable cooling profile |
| Temperatur für minimale Speed | 23°C | Fan runs at minimum speed at this temperature |
| Temperatur für maximale Speed | 30°C | Fan runs at maximum speed at this temperature |
| Ausschalt-Temperatur | 22°C | Fan turns off below this temperature |

**Heizen (Heating Profile)**

| Input | Default | Description |
|---|---|---|
| Heizen aktivieren | `false` | Enable heating profile |
| Temperatur für minimale Speed | 21°C | Fan runs at minimum speed at this temperature |
| Temperatur für maximale Speed | 16°C | Fan runs at maximum speed at this temperature |
| Ausschalt-Temperatur | 22°C | Fan turns off above this temperature |

**Allgemein**

| Input | Default | Description |
|---|---|---|
| ⏱ Mindestzeit zwischen Anpassungen | 0 min | Minimum time between two adjustments. The latest value is always applied; turning the fan off is always immediate |

### How It Works

The blueprint calculates a `ratio` (0.0–1.0) based on how far the current temperature is between the configured min and max temperature for the active profile. This ratio is then linearly mapped to either a percentage value or a preset index.

**Cooling example** with `cool_temp_min = 23°C`, `cool_temp_max = 30°C`, `pct_min = 20%`, `pct_max = 100%`:

| Temperature | Fan Speed |
|---|---|
| ≤ 23°C | 20% (minimum) |
| 26.5°C | ~60% |
| ≥ 30°C | 100% (maximum) |

**Hysteresis** (cooling): the fan turns on once the temperature reaches `cool_temp_off + temp_gap` and stays on until it drops below `cool_temp_off + temp_off_offset`.
Heating is mirrored: on at `heat_temp_off − temp_gap`, off once the temperature rises above `heat_temp_off − temp_off_offset`.

Example with defaults (`cool_temp_off = 22°C`, `temp_gap = 1.0`, `temp_off_offset = 0`): on at 23.0°C, stays on down to 22.0°C, off at 21.9°C.

### Example: Preset Fan (AC unit)

For a fan with presets `1 - Diffuse, 2 - Low, 3 - Medium, 4 - High, 5 - Focus`:

- Set **Steuerungsmodus** to `Preset`
- Set **Preset-Reihenfolge** to `1 - Diffuse, 2 - Low, 3 - Medium, 4 - High, 5 - Focus`
- Set **Niedrigster Preset** to `1 - Diffuse`
- Set **Höchster Preset** to `5 - Focus`

The blueprint will automatically map the temperature range to the available preset steps.

</details>

---

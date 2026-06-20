## 🏠 My collection of Home Assistant Blueprints

<details>
<summary>💨 Universal Fan Speed Control (Testing)</summary>

A Home Assistant Blueprint that controls fan speed based on room temperature — supports both **Percentage fans** (0–100%) and **Preset fans** (Named Modes), with separate profiles for cooling and heating.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/duczz/ha-blueprints/main/universal_fan_speed_control.yaml)

### Features

- **Universal**: works with any `fan.*` entity that supports `percentage` or `preset_mode`
- **Linear interpolation**: fan speed scales smoothly between min and max temperature
- **Dual mode**: separate temperature profiles for cooling and heating
- **Hysteresis**: configurable dead-band to prevent rapid on/off cycling
- **Off offset**: separate threshold for fan shutdown to avoid flapping
- **Enable/Disable toggle**: `input_boolean` helper to quickly disable the automation (e.g. away mode, sleep mode)
- **Restart mode**: new temperature changes immediately re-evaluate fan speed

### Requirements

- Home Assistant 2024.6.0 or newer
- An `input_boolean` helper for the enable/disable toggle (create under **Settings → Helpers**)

### Inputs

**Required**

| Input | Description |
|---|---|
| 🌡 Temperatursensor | Temperature sensor entity |
| 💨 Fan Entity | The fan entity to control |
| 🔘 Enable/Disable Toggle | `input_boolean` to enable or disable the automation |

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
| Ausschalt-Offset | 0° | Negative offset for the fan-off temperature (0 = same as on-threshold) |

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
| ⏱ Mindestzeit zwischen Anpassungen | 5 min | Minimum delay between speed adjustments |

### How It Works

The blueprint calculates a `ratio` (0.0–1.0) based on how far the current temperature is between the configured min and max temperature for the active profile. This ratio is then linearly mapped to either a percentage value or a preset index.

**Cooling example** with `cool_temp_min = 23°C`, `cool_temp_max = 30°C`, `pct_min = 20%`, `pct_max = 100%`:

| Temperature | Fan Speed |
|---|---|
| ≤ 23°C | 20% (minimum) |
| 26.5°C | ~60% |
| ≥ 30°C | 100% (maximum) |

**Hysteresis** is applied to the off-threshold: the fan only turns on once the temperature exceeds `cool_temp_off + temp_gap`, and only turns off once it drops below `cool_temp_off + temp_off_offset`.

### Example: Preset Fan (AC unit)

For a fan with presets `1 - Diffuse, 2 - Low, 3 - Medium, 4 - High, 5 - Focus`:

- Set **Steuerungsmodus** to `Preset`
- Set **Preset-Reihenfolge** to `1 - Diffuse, 2 - Low, 3 - Medium, 4 - High, 5 - Focus`
- Set **Niedrigster Preset** to `1 - Diffuse`
- Set **Höchster Preset** to `5 - Focus`

The blueprint will automatically map the temperature range to the available preset steps.

</details>

---

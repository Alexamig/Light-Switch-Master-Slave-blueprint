# Light/Switch Master-Slave Blueprint v5.0 FINAL STABLE

[Russian](README.ru.md)

[![Version](https://img.shields.io/badge/version-v5.0-blue)](https://github.com/Alexamig/ha-light-switch-master-slave/releases)
![Status](https://img.shields.io/badge/status-FINAL%20STABLE-success)
![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.x%2B-41BDF5?logo=home-assistant)
[![License](https://img.shields.io/badge/license-MIT-green)](https://github.com/Alexamig/ha-light-switch-master-slave/blob/main/LICENSE)
![GitHub release](https://img.shields.io/github/v/release/Alexamig/Light-Switch-Master-Slave-blueprint)

**Production-ready Master/Slave automation blueprint for Home Assistant**

Universal blueprint for controlling `light` and `switch` entities with complete fail-safe protection based on motion sensor and timers.

## 📌 Status

✅ **FINAL STABLE** – frozen architecture, thoroughly tested in production

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| **⏱️ Dual motion delays** | Separate delays for motion start (`debounce_no_motion`) and motion end (`debounce_after_motion`) |
| **🎚️ Master/Slave architecture** | One master + unlimited slaves (mixed `light`/`switch`) |
| **🔄 Flexible sync modes** | Group mode or independent operation |
| **⏲️ Timer flexibility** | Separate timers OR one common timer |
| **🌞🌜 Day/Night mode** | Auto-enable only at night (with invert option) |
| **🔕 Per-slave day/night ignore** | Select slaves that work ALWAYS |
| **🛡️ Fail-safe protection** | Auto-timers if sensor unavailable + 5-min safety checks |
| **🔧 Triple config validation** | Master in slaves, missing timers, Master in ignore list |

---

## 📋 Input Fields

| Field | Required | Description |
|-------|----------|-------------|
| `debounce_no_motion` | ✓ | Delay before motion triggers (0-10 sec) |
| `debounce_after_motion` | ✓ | Delay after motion stops (0-30 sec) |
| `use_motion_sensor` | ✓ | Master switch for auto-enable |
| `motion_sensor` | ✓ | Binary motion/occupancy sensor |
| `master_entity` | ✓ | Main light/switch |
| `master_timer_helper` | ✓ | Timer for Master |
| `master_time_off` | ✓ | Master timer duration |
| `sync_slave_group` | ✓ | Group mode ON/OFF |
| `slave_entities` | - | Additional lights/switches |
| `slave_timer_helper` | - | Timer for Slaves |
| `slave_time_off` | - | Slave timer duration |
| `slave_ignore_day_night` | - | Slaves that ignore day/night |
| `night_sensor` | ✓ | Day/Night binary sensor |
| `use_day_night` | ✓ | Enable day/night mode |
| `invert_night_sensor` | ✓ | Invert day/night logic |

---

## 📊 Behavior Reference

### Day/Night Mode

| Mode | Motion | Auto-enable | Action |
|------|--------|-------------|--------|
| 🌞 Day | Yes | ❌ Disabled | No light |
| 🌞 Day | No | ❌ Disabled | - |
| 🌜 Night | Yes | ✅ Enabled | Light ON |
| 🌜 Night | No | ✅ Enabled | - |

### Motion Response × Day/Night

| Motion Response | Day/Night Mode | Motion | Result |
|-----------------|----------------|--------|--------|
| ✅ ON | ❌ OFF | Yes | ✅ Light ON |
| ✅ ON | ❌ OFF | No | ❌ Timer starts |
| ✅ ON | ✅ ON (Day) | Yes | ❌ Light OFF |
| ✅ ON | ✅ ON (Night) | Yes | ✅ Light ON |
| ❌ OFF | Any | Yes | ❌ Light OFF |
| ❌ OFF | Any | No | ❌ Manual only |

### Per-Slave Day/Night Ignore

| Situation | Sync | Ignore List | Result |
|-----------|------|-------------|--------|
| Slave in list | ❌ OFF | ✅ In list | **Always works** |
| Slave NOT in list | ❌ OFF | ❌ Not in list | Follows day/night |
| Any Slave | ✅ ON | Any | ❌ **Does NOT work** |

---

## 🚀 Installation

### Via HACS (recommended)

1. Add this repository to HACS:
   - HACS → Integrations → 3 dots → Custom repositories
   - URL: `https://github.com/Alexamig/Light-Switch-Master-Slave-blueprint`
   - Category: `Blueprint`

2. Find "Light/Switch Master-Slave" in HACS blueprints and download

### Manual installation

1. Copy `blueprint.yaml` to your Home Assistant `blueprints/automation/` folder
2. In HA, go to **Settings → Automations & Scenes → Blueprints**
3. Click **Import Blueprint** and select the file

---

## 🔧 Example Dummy for Night Sensor

```yaml
template:
  - binary_sensor:
      - name: "Night Always"
        unique_id: night_always
        state: "{{ false }}"  # false = Always Night, true = Always Day
        availability: "{{ true }}"

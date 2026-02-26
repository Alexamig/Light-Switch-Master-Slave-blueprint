# Light/Switch Master-Slave Blueprint v5.0 FINAL STABLE

[Russian](README.ru.md)

[![Version](https://img.shields.io/badge/version-v5.0-blue)](https://github.com/Alexamig/Light-Switch-Master-Slave-blueprint/releases/tag/v5.0)
![Status](https://img.shields.io/badge/status-FINAL%20STABLE-success)
![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.x%2B-41BDF5?logo=home-assistant)
[![License](https://img.shields.io/badge/license-MIT-green)](https://github.com/Alexamig/ha-light-switch-master-slave/blob/main/LICENSE)
![GitHub release](https://img.shields.io/github/v/release/Alexamig/Light-Switch-Master-Slave-blueprint)
![GitHub all releases](https://img.shields.io/github/downloads/Alexamig/Light-Switch-Master-Slave-blueprint/total)

# Home Assistant Master/Slave Motion Automation Blueprint

A universal blueprint for managing `light` and `switch` entities with comprehensive fail-safe protection based on motion sensors and timers.
Devices of the type (light or switch) can be used as master devices. Mixed devices of the same type (light and switch) can be used as slave devices at once.

## **❤️ Please support the author by clicking the Star button 🌟**

## ⭐ CORE LOGIC

### 1️⃣ Motion Delays

**⏳ Pre-motion delay:**
- Motion confirmation delay
- `motion_on` event is processed only if the sensor remains in `on` state continuously for the specified time
- Protection against false triggers from brief signals

**⏳ Post-motion delay:**
- Motion end confirmation delay
- `motion_off` event is processed with a delay, timers start only after it expires
- Prevents lights from turning off during brief motion gaps

### 2️⃣ Master Device
- Automatically turns on when motion detected if:
  - **🧍 Motion Control** = ON (auto-on enabled)
  - **🌜 Night Mode** is active (or **🔆 Day/Night Mode** is off)
- When motion stops, starts Master timer if Master is on
- Manual Master turn-on WITHOUT motion ALWAYS starts Master timer
- Manual Master turn-off:
  - Stops Master timer
  - Turns off all Slaves if synchronization is enabled
- **⚡ Auto-on by motion:**
  - Master turns on automatically
  - **Slaves turn on automatically ONLY if:**
    * Sync 🔄 is enabled
    * All Slaves are off
    * Night/Day mode allows it
    * **🧍 Motion Control** = ON

### 3️⃣ Slave Devices
- Slave timer is shared: one timer manages all Slaves simultaneously
- When a Slave turns on (with sync enabled), automatically turns on Master if it was off
- Manual Slave turn-on without motion starts Slave timer (only if sync is disabled)
- When sync is enabled:
  - ALL other Slaves turn on, except the one that triggered the event
  - Slave timer is not used (only Master timer works)
- Manual Slave turn-off:
  - Does NOT turn off Master
  - If Master is off and all Slaves are off, active timer stops
- Slave timer responds to motion: timer cancels when motion detected

### 4️⃣ Timer Logic
- Master and Slave timers start only when in idle state (not active)
- Timers stop when motion detected or devices manually turned off
- Master timer turns off Master and ALL active Slaves (with sync or shared timer)
- Slave timer is shared for all Slaves and turns off ALL active Slaves simultaneously
- **⚠️ Timers ALWAYS work, regardless of day/night mode!**

### 5️⃣ Day/Night Mode (optional)

| Mode | Motion | Auto-on | Action |
|------|--------|---------|--------|
| ☀️ Day | Yes | ❌ Disabled | Lights don't turn on |
| ☀️ Day | No | ❌ Disabled | - |
| 🌙 Night | Yes | ✅ Enabled | Lights turn on |
| 🌙 Night | No | ✅ Enabled | - |

**🔄 Mode switching:**
- Day → Night, motion present → **lights TURN ON** ✅
- Night → Day, motion present → **lights TURN OFF** ✅
- Night → Day, no motion → **lights TURN OFF** ✅

**🔧 Example night_sensor placeholder:**
```yaml
template:
  - binary_sensor:
      - name: "Night Always"
        unique_id: night_always
        state: "{{ false }}"
        availability: "{{ true }}"

state: "{{ false }}" - Always Night (OFF) ✅ auto-on enabled
state: "{{ true }}" - Always Day (ON) ❌ auto-on disabled
```

6️⃣ Fail-safe Protection
If motion sensor becomes unavailable or unknown while Master or any Slave remains on, the corresponding timer starts

Additional periodic check every 5 minutes

7️⃣ Features
Supports any number of Slave devices

Slaves can be mixed domains (light and switch)

Two independent delays: turn-on debounce (debounce_no_motion) and turn-off debounce (debounce_after_motion)

All service calls execute only when target entities exist

8️⃣ 💡 Timer Flexibility
One timer for Master and Slave — use the SAME timer helper in both fields!

✅ Master timer turns off Master and Slaves (with sync)

✅ Slave timer turns off only Slaves

⚡ Works even without synchronization!

🔧 Simply specify one entity in:

⏱️ Master Timer

⏲️ Slave Timer

9️⃣ 🔌 Mode Control

**Interaction between `🧍 Motion Control` and `🔆 Day/Night Mode`:**

| 🧍 Motion Control | 🔆 Day/Night Mode | Motion | Result |
|-------------------|-------------------|--------|--------|
| ✅ ON | ❌ OFF | Yes | ✅ Lights always turn on |
| ✅ ON | ❌ OFF | No | ❌ Timer starts |
| ✅ ON | ✅ ON (Day) | Yes | ❌ Lights DON'T turn on |
| ✅ ON | ✅ ON (Night) | Yes | ✅ Lights turn on |
| ❌ OFF | Any | Yes | ❌ Lights DON'T turn on |
| ❌ OFF | Any | No | ❌ Only timers from manual on |

**Important:**
- `🧍 Motion Control = OFF` completely disables automation, but manual control and timers work normally
- When Day/Night mode changes, lights sync with motion (even if `🧍 Motion Control = OFF`)

---

### 🔟 🔕 Ignoring Time of Day for Slaves

This feature allows specific Slaves to work **independently** of day/night mode.

| Situation | 🔄 Sync | 🔕 Ignore | Result |
|-----------|---------|-----------|--------|
| Slave in list | ❌ OFF | ✅ In list | **Always works** |
| Slave NOT in list | ❌ OFF | ❌ Not in list | Follows day/night |
| Any Slave | ✅ ON | Any | ❌ **DOESN'T WORK** (all as group) |

**📌 IMPORTANT:** This option is only available when **🔄 Sync = OFF**

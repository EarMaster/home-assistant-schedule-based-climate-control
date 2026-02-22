# Schedule Based Climate Control 🌡️

> **Current Version:** 1.1.3

A professional, high-performance Home Assistant blueprint for managing climate entities (ACs, Thermostats, Heaters) based on schedules, safety sensors, and user overrides.

---

## Installation

Use the badge below to import the latest stable version from the `main` branch:

[![Open your Home Assistant instance and show the blueprint import dialog.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A//raw.githubusercontent.com/EarMaster/home-assistant-schedule-based-climate-control/main/schedule_based_climate_control.yaml)

Alternatively, you can import this specific version: [🔗 Import version v1.1.4](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A//github.com/EarMaster/home-assistant-schedule-based-climate-control/blob/v1.1.4/schedule_based_climate_control.yaml)

---

## 🚀 Overview
This blueprint serves as the central intelligence for your climate devices. It ensures energy efficiency by enforcing safety rules (windows/vacation) while providing a "Party Mode" for manual freedom.

---

## 🛠 Required & Optional Helpers

To use this blueprint effectively, navigate to **Settings > Devices & Services > Helpers** and create the following entities:

### 1. Timeframe (Required)
| Helper Type | Suggested Name | Purpose |
| :--- | :--- | :--- |
| **Schedule** | `Climate Schedule` | Defines the hours when the automation is active. |

### 2. Mode Switches (Optional)
| Helper Type | Suggested Name | Purpose |
| :--- | :--- | :--- |
| **Input Boolean** | `Vacation Mode` | If ON, forces all devices OFF regardless of temperature. |
| **Input Boolean** | `Party Mode` | If ON, pauses the automation logic to allow manual control. |

### 3. Temperature Controllers (Optional)
You can either enter fixed numbers in the blueprint or create these **Input Number** helpers to control them from your dashboard:
| Helper Type | Suggested Name | Purpose |
| :--- | :--- | :--- |
| **Input Number** | `Target Temp Cooling` | The temperature the AC should reach (e.g., 21°C). |
| **Input Number** | `Threshold Hot` | The room temperature that triggers cooling (e.g., 26°C). |
| **Input Number** | `Target Temp Heating` | The temperature the heater should reach (e.g., 22°C). |
| **Input Number** | `Threshold Cold` | The room temperature that triggers heating (e.g., 19°C). |

---

## 📖 Step-by-Step Setup

1. **Create Helpers**: Setup the Schedule and any of the optional Input Booleans/Numbers listed above.
2. **Import Blueprint**: Click the badge above or import the GitHub URL manually.
3. **Configure Automation**:
   - **Climate Devices**: Select your AC units or Thermostats.
   - **Thresholds/Targets**: You can type a number (e.g., `21`) or an entity ID (e.g., `input_number.target_temp_cooling`) into these fields.
4. **Window Sensors**: Select your door/window contacts. The `Window Delay` ensures that short openings (like reaching out of a window) don't toggle your HVAC immediately.

---

## 🍱 Modern Dashboard Example
Copy this code into a **Manual Card** in your dashboard for a professional control interface:

```yaml
type: vertical-stack
cards:
  - type: tile
    entity: climate.your_device
    features:
      - type: climate-hvac-modes
        hvac_modes: [off, cool, heat]
  - type: grid
    columns: 2
    square: false
    cards:
      - type: tile
        entity: input_boolean.party_mode
        name: Party Mode
        color: pink
      - type: tile
        entity: input_boolean.vacation_mode
        name: Vacation
        color: amber
  - type: entities
    title: Temperature Settings
    entities:
      - entity: schedule.climate_schedule
      - entity: input_number.target_temp_cooling
      - entity: input_number.threshold_hot
      - entity: input_number.target_temp_heating
      - entity: input_number.threshold_cold
```

## ⚖️ Logic Hierarchy

1. **Safety Cut-off**: If a **Window** is open (after delay), **Vacation Mode** is ON, or the **Schedule** is inactive -> **Turn OFF**.
2. **User Override**: If **Party Mode** is ON -> **Stop** (The automation will not change any settings).
3. **Automatic Control**: If room temp hits a **Threshold**, the automation sets the corresponding **Target Temp**.
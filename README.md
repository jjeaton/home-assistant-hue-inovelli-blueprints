[![Import Automation Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FRatoka%2Fhome-assistant-hue-inovelli-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Finovelli%2Fhue_dimmer_zha.yaml) **ZHA Automation Blueprint**

[![Import Script Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FRatoka%2Fhome-assistant-hue-inovelli-blueprints%2Fblob%2Fmain%2Fblueprints%2Fscripts%2Finovelli%2Fdim_hue_room_lights_zha.yaml) **ZHA Dimming Script Blueprint** *(required for ZHA)*

[![Import Automation Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FRatoka%2Fhome-assistant-hue-inovelli-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Finovelli%2Fhue_dimmer_z2m.yaml) **Z2M Automation Blueprint**

[![Import Automation Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FRatoka%2Fhome-assistant-hue-inovelli-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Finovelli%2Fshower_fan_z2m.yaml) **Shower Humidity Fan Blueprint** *(VZM35-SN, Z2M)*

[![Import Automation Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FRatoka%2Fhome-assistant-hue-inovelli-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomations%2Finovelli%2Fmmwave_presence_diagnostics_z2m.yaml) **mmWave Presence Diagnostics Blueprint** *(VZM32-SN, Z2M)*

> **Helper entity examples:** See [`examples/helpers_z2m.yaml`](examples/helpers_z2m.yaml) (Z2M) and [`examples/helpers_zha.yaml`](examples/helpers_zha.yaml) (ZHA) for ready-to-paste helper configurations.


# 🏠 Home Assistant Inovelli Hue Blueprints

This repository contains **dynamic, reusable blueprints** for integrating Inovelli Blue Series dimmers with Philips Hue lights and scenes. These automations are designed for **state-aware, adaptive lighting control** that evolves with your Hue setup.

**Available for:**
- **ZHA Integration** (via `hue_dimmer_zha.yaml`)
- **Zigbee2MQTT Integration** (via `hue_dimmer_z2m.yaml`)

**Also included** (independent of the Hue blueprints, Zigbee2MQTT only):
- [Shower Humidity Fan](#-shower-humidity-fan-vzm35-sn-z2m) for the VZM35-SN fan switch
- [mmWave Presence Diagnostics](#-mmwave-presence-diagnostics-vzm32-sn-z2m) for the VZM32-SN, with a companion visualizer tool

---

## ✨ Features

- **💡 Light On/Off Control**
  - Single tap up: Turns Hue room **on**
  - Single tap down: Turns Hue room **off**
  - Double tap up: Turns light on to 100%
  - Double tap down: Turns light on to 10%
  - Optional: Automatically activates a default scene when turning on

- **🎚️ Unified Paddle Dimming**
  - Hold paddle: Smooth dim up/down (inline — no separate script needed for Z2M)
  - Release paddle: Stop dimming
  - Z2M: holding down stops at minimum brightness and never turns the light off

- **🎬 Scene Cycling (Config Button)**
  - Press: Cycle forward through Hue scenes
  - Hold: Cycle backward
  - Scenes are dynamically discovered per Hue room
  - LED color/intensity updates to match the activated scene
  - Requires one `input_number` helper (optional — scene cycling disabled if not configured)

- **💡 LED Sync**
  - LED color mirrors the Hue room's light color
  - LED intensity mirrors the Hue room's brightness
  - Z2M: the LED bar level (how much of the bar is lit) also follows the Hue room's brightness. While you hold the paddle the switch moves the bar at its own dimming speed, then it settles on the room's actual level about a second after release.
  - LED switches to a configurable "off" color when the light is off

- **🔄 Bidirectional Sync**
  - Switch and Hue room always stay in sync
  - Works whether the light is changed from the switch, the Hue app, or Home Assistant

---

## Overview

The diagram below shows the **ZHA** flow, including the helper entities and dimming script it relies on. The Z2M blueprint needs neither; its flow is in [`docs/InovelliHueFlow_z2m.drawio`](docs/InovelliHueFlow_z2m.drawio) (open it with [draw.io](https://app.diagrams.net/)).

![Device Overview (ZHA)](/docs/assets/DeviceFlow.png)

---

## 🚀 Getting Started (ZHA)

1. **Create a Hue Room** in the Hue app (e.g. `Bedroom` or `Living Room`). Any room name works, including names with spaces.
2. **Add Hue Scenes** to the room in the Hue app.
3. **Add your Inovelli Blue switch** to Home Assistant via ZHA.
4. **Configure the switch** in Home Assistant:
   - Enable Smart Bulb mode
   - Enable Double Tap Up / Double Tap Down
   - Set button delay to at least 1 (required for double tap)
5. **Create the required helper entities** for each room (see [ZHA — Requirements](#zha--minimum-onoff-dimming-led-sync) below).
6. **Import both ZHA blueprints** using the buttons at the top of this page — the automation **and** the dimming script.
7. **Create a script** from the dimming script blueprint. No inputs need to be filled in; the automation passes them at runtime.
8. **Create an automation** from the ZHA automation blueprint and fill in:
   - **Inovelli Switch (ZHA)** — the ZHA device
   - **Inovelli Switch Entity** — the light entity of the switch (controls LED bar)
   - **Light Group (Hue Room or Zone)** — the Hue room entity (e.g., `light.bedroom`)
   - **Dimming Script** — the script entity you created in step 7
   - **Dimming Speed** — brightness change per step (lower = slower)
   - **LED Color When Off** — Inovelli LED color when light is off (0–255, 24 = warm white)
   - *(Optional)* **Enable Default Scene / Default Scene** — scene to activate on every turn-on
9. **Enjoy.**

> The room name is derived automatically from the Hue light group entity ID (e.g. `light.bedroom` → room = `bedroom`). All helper entity IDs must match this derived name exactly.
>
> **Rooms with spaces:** Home Assistant turns spaces into underscores in entity IDs, so a Hue room called `Living Room` becomes `light.living_room` and the room is `living_room`. Its helpers are `input_boolean.dimmer_living_room`, `input_boolean.sync_living_room` and `input_number.scene_index_living_room`.

---

## 🛠️ Requirements (ZHA)

### ZHA — Minimum (on/off, dimming, LED sync)

| Entity | Example | Purpose | Where to Create |
|--------|---------|---------|-----------------|
| Input Boolean (dimmer) | `input_boolean.dimmer_bedroom` | Controls the dimming loop | Settings → Helpers → Add → Toggle |
| Input Boolean (sync) | `input_boolean.sync_bedroom` | Prevents sync trigger loops | Settings → Helpers → Add → Toggle |

### ZHA — With Scene Cycling

In addition to the two booleans above:

| Entity | Example | Purpose | Where to Create |
|--------|---------|---------|-----------------|
| Input Number | `input_number.scene_index_bedroom` | Tracks active scene index | Settings → Helpers → Add → Number |

Configure the input_number with min: `0`, max: `99`, step: `1`.

> A ready-to-paste example for bedroom and office rooms is in [`examples/helpers_zha.yaml`](examples/helpers_zha.yaml).

> **Scene discovery:** Every Hue scene that belongs to the selected Hue room or zone is found automatically. Discovery follows the room itself rather than entity names, so it works for room names with spaces and for rooms or scenes you have renamed. Scenes cycle in alphabetical order of their entity IDs.

---

## 🧪 Example Setup (ZHA): Room = `Bedroom`

1. In the Hue app, create a room named `Bedroom`
   - Add lights to the room
   - Add scenes (e.g., `Relax`, `Energize`, `Read`)
2. In Home Assistant, create the required helpers:
   - **Settings → Helpers → Add → Toggle** → Name: `Dimmer - Bedroom` → entity: `input_boolean.dimmer_bedroom`
   - **Settings → Helpers → Add → Toggle** → Name: `Sync - Bedroom` → entity: `input_boolean.sync_bedroom`
   - *(Optional)* **Settings → Helpers → Add → Number** → Name: `Scene Index - Bedroom` → entity: `input_number.scene_index_bedroom` → Min: `0`, Max: `99`, Step: `1`
3. Import the ZHA dimming script blueprint and create a script from it (no configuration needed).
4. Import the ZHA automation blueprint and create an automation:
   - **Inovelli Switch (ZHA)**: your ZHA device
   - **Inovelli Switch Entity**: `light.bedroom_switch` (your switch's light entity)
   - **Light Group**: `light.bedroom`
   - **Dimming Script**: the script you created in step 3
   - **Brightness Step**: `10` (adjust to taste)

Your Inovelli switch will now:
- **Tap up**: Turn on `light.bedroom`
- **Tap down**: Turn off `light.bedroom`
- **Double tap up**: Set `light.bedroom` to 100%
- **Double tap down**: Set `light.bedroom` to 10%
- **Hold up/down**: Dim smoothly; release to stop
- **Config press/hold**: Cycle through Hue scenes (if scene helper is configured)
- **LED**: Reflects room color and brightness; off-state color when light is off
- **Hue app / HA changes**: Switch and light stay in sync automatically

---

## 🚀 Getting Started (Z2M)

1. **Create a Hue Room** in the Hue app (e.g. `Bedroom` or `Living Room`). Any room name works, including names with spaces.
2. **Add Hue Scenes** to the room in the Hue app.
3. **Add your Inovelli Blue switch** to Home Assistant via Zigbee2MQTT.
4. **Configure the switch** in Home Assistant:
   - Enable Smart Bulb mode
   - Enable Double Tap Up / Double Tap Down
   - Set button delay to at least 1 (required for double tap)
5. **Import the Z2M blueprint** using the button at the top of this page.
6. **Create an automation** from the blueprint and fill in:
   - **Z2M Action Sensor** — the custom MQTT sensor you created (e.g., `sensor.bedroom_switch_action`). See the [Z2M Action Sensor](#-z2m-action-sensor) section below.
   - **Inovelli Switch Light Entity** — the light entity of the switch (controls LED bar)
   - **Hue Light Group** — the Hue room or zone entity (e.g., `light.bedroom`)
   - **Z2M Device Name** — the friendly name in Zigbee2MQTT (e.g., `bedroom_switch`)
   - **Dimming Speed** — brightness change per step (lower = slower)
   - **LED Color When Off** — Inovelli LED color when light is off (0–255, 24 = warm white)
   - *(Optional)* **Scene Index Helper** — entity ID of an `input_number` helper for scene cycling
   - *(Optional)* **Default Scene** — scene to activate on every turn-on
7. **Enjoy.**

> No script blueprint or manual MQTT sensor configuration is required.

---

## 🛠️ Requirements (Z2M)

### Z2M — Minimum (on/off, dimming, LED sync)

No helper entities required. Only the automation blueprint is needed.

### Z2M — With Scene Cycling

| Entity | Example | Purpose | Where to Create |
|--------|---------|---------|-----------------|
| Input Number | `input_number.bedroom_scene_index` | Tracks active scene index | Settings → Helpers → Add → Number |

Configure the input_number with min: `0`, max: `99`, step: `1`.

> A ready-to-paste example for bedroom and office rooms is in [`examples/helpers_z2m.yaml`](examples/helpers_z2m.yaml).

> **Scene discovery:** Every Hue scene that belongs to the selected Hue room or zone is found automatically. Discovery follows the room itself rather than entity names, so it works for room names with spaces and for rooms or scenes you have renamed. Scenes cycle in alphabetical order of their entity IDs.

---

## 🔌 Z2M Action Sensor

The blueprint needs an MQTT sensor that reports button presses from your switch.

> **Note:** Newer versions of Zigbee2MQTT do not expose an action sensor in Home Assistant by default. You must create the sensor manually as described below.

### Required: Custom MQTT Sensor (with `expire_after`)

Add this to your `configuration.yaml` (or a file included by it):

```yaml
mqtt:
  sensor:
    - name: "bedroom_switch_action"
      state_topic: "zigbee2mqtt/bedroom_switch/action"
      icon: mdi:access-point
      expire_after: 1
```

- **`name`**: Creates `sensor.bedroom_switch_action` in HA
- **`state_topic`**: Must match your Z2M device's MQTT topic exactly (case-sensitive). The device name is the friendly name shown in the Zigbee2MQTT dashboard.
- **`expire_after: 1`**: Resets the sensor to `unavailable` after 1 second. This is required so that repeated presses of the same button action (e.g., cycling scenes twice with `config_single`) each trigger the automation.

Restart Home Assistant after adding the sensor.

> A ready-to-paste example for bedroom and office switches is in [`examples/helpers_z2m.yaml`](examples/helpers_z2m.yaml).

### Alternative: Auto-Created Z2M Sensor

Older versions of the Z2M HA integration may create an action sensor automatically. You can use it without any `configuration.yaml` changes, but **repeated presses of the same button action will not re-trigger the automation** (the state doesn't change if the same action fires twice). This means cycling scenes multiple times with a single `config_single` press will only work every other press.

**To find your auto-created action sensor:**
1. Press any button on your Inovelli switch
2. Go to **Developer Tools → States**
3. Search for `sensor.<device_name>_action` (e.g., `sensor.bedroom_switch_action`)

**Verify the sensor works:**
- The state should change each time you press a button (e.g., `up_single`, `down_held`, `config_single`)
- If it doesn't update, confirm Z2M is running and the device is paired

---

## 🧪 Example Setup (Z2M): Room = `Bedroom`

1. In the Hue app, create a room named `Bedroom`
   - Add lights to the room
   - Add scenes (e.g., `Relax`, `Energize`, `Read`)
2. *(Optional)* In Home Assistant, create one helper for scene cycling:
   - **Settings → Helpers → Add → Number**
   - Name: `Bedroom Scene Index` → entity: `input_number.bedroom_scene_index`
   - Min: `0`, Max: `99`, Step: `1`
3. Import the Z2M blueprint and create an automation:
   - **Z2M Action Sensor**: `sensor.bedroom_switch_action`
   - **Inovelli Switch Light Entity**: `light.bedroom_switch` (your switch's light entity)
   - **Hue Light Group**: `light.bedroom`
   - **Z2M Device Name**: `bedroom_switch`
   - **Dimming Speed**: `10` (adjust to taste)
   - **Scene Index Helper**: `input_number.bedroom_scene_index` *(optional)*

Your Inovelli switch will now:
- **Tap up**: Turn on `light.bedroom`
- **Tap down**: Turn off `light.bedroom`
- **Double tap up**: Set `light.bedroom` to 100%
- **Double tap down**: Set `light.bedroom` to 10%
- **Hold up/down**: Dim smoothly; release to stop
- **Config press/hold**: Cycle through Hue scenes (if scene helper is configured)
- **LED**: Reflects room color and brightness; off-state color when light is off
- **Hue app / HA changes**: Switch and light stay in sync automatically

---

## ⚠️ Common Pitfalls

| Issue | Fix |
|-------|-----|
| Scenes not cycling | Confirm the Light Group you selected is a Hue **room or zone** (not a single bulb or a Home Assistant light group) and that it has scenes in the Hue app. Also verify the scene index helper exists: for Z2M it must be selected in the blueprint; for ZHA it must be named `input_number.scene_index_<room>`. |
| Action sensor not found | Press a button on the switch, then search Developer Tools → States for `_action`. Confirm Z2M is running and the device is paired. |
| Wrong Z2M device name | The Z2M Device Name must exactly match the friendly name in the Z2M dashboard (case-sensitive). |
| Lights not turning on/off (ZHA) | The helper entity IDs must match the room part of the Hue light entity ID exactly (e.g. `light.living_room` → `input_boolean.sync_living_room` and `input_boolean.dimmer_living_room`). |
| LED color not updating | Confirm the Z2M Device Name matches your Z2M config exactly. |

---

## 🚿 Shower Humidity Fan (VZM35-SN, Z2M)

`shower_fan_z2m.yaml` turns a bathroom exhaust fan on when humidity rises and off again once the room has dried out. It is independent of the Hue blueprints and needs no helpers.

- **On:** room humidity rises above the **On Threshold**. If a **Reference Humidity Sensor** is selected (hallway, bedroom, outdoors), the room must also exceed it by the **Humidity Delta**, which stops the fan running on humid or rainy days. If the reference sensor is unavailable, only the On Threshold is used.
- **Off:** humidity falls below the **Off Threshold**, then the fan waits out any remaining **Minimum Run Time**, then the **Off Delay**, and re-checks humidity before switching off. If humidity climbs back above the On Threshold while it is waiting, the fan stays on.

| Input | Default | Notes |
|-------|---------|-------|
| Fan Entity | – | The VZM35-SN fan entity from Zigbee2MQTT |
| Room Humidity Sensor | – | Drives all on/off decisions |
| Reference Humidity Sensor | *(none)* | Optional |
| Humidity On Threshold | 70 % | Set above your normal resting humidity |
| Humidity Delta Above Reference | 8 % | Only used with a reference sensor |
| Humidity Off Threshold | 60 % | Keep below the On Threshold to avoid rapid cycling |
| Minimum Run Time | 15 min | |
| Off Delay After Humidity Drops | 5 min | |

---

## 📡 mmWave Presence Diagnostics (VZM32-SN, Z2M)

`mmwave_presence_diagnostics_z2m.yaml` is a troubleshooting aid for mmWave false positives. Each time the switch reports presence it posts a Home Assistant notification showing which of the four detection areas were active, and when presence clears it reports how long the detection lasted. Short detections in an empty room point to a false positive. Disable the automation once your zones are tuned.

1. Create one automation per VZM32-SN and select its **Occupancy Sensor**.
2. *(Optional)* Select the **Area 1–4 Occupancy** sensors. These only exist once the `manuSpecificInovelliMMWave` cluster is bound to Source endpoint 1 in the Zigbee2MQTT **Bind** tab.
3. *(Optional)* Set a **Location Label**, and an **Additional Notify Service** (service name without the `notify.` prefix) to also receive the reports on your phone.
4. Leave the room, wait for a false positive, and note which areas were flagged.
5. In Zigbee2MQTT → device → **Configure**, add an Interference Area covering that zone. Repeat until the false positives stop.

### Visualizer tool

`tools/mmwave_visualizer/` is a small local web app for deeper analysis. It reads the last 24 hours of sensor history from Home Assistant, draws your detection zones in 3D with a timeline of presence events, and generates Interference Area settings for a time window you select.

```sh
pip install -r tools/mmwave_visualizer/requirements.txt
python tools/mmwave_visualizer/app.py
```

Then open <http://localhost:8050> and enter your Home Assistant URL, a long-lived access token (Profile → Security), the sensor entity IDs, and the same X/Y/Z zone bounds you configured in Zigbee2MQTT.

---

## 🔍 Resources

- [Blueprint Exchange Thread](https://community.home-assistant.io/t/inovelli-blue-zha-hue-lights/910208)
- [Inovelli Blue Devices](https://inovelli.com/collections/inovelli-blue-series)
- [Inovelli Blue Devices ZHA Documentation](https://help.inovelli.com/en/articles/8452425-blue-series-dimmer-switch-setup-instructions-home-assistant-zha)
- [Philips Hue Integration](https://www.home-assistant.io/integrations/hue/)
- [Philips Hue App Documentation](https://www.philips-hue.com/en-us/explore-hue/apps/bridge)
- Created by [@Ratoka](https://github.com/Ratoka)

---

## 📜 License

See the LICENSE file

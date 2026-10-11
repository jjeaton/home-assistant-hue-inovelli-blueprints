# 🧾 Changelog

All notable changes to this project will be documented in this file, following [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format.

---

## [Unreleased]

Covers everything since 0.1.1; no versions were tagged in between.

### Added
- Zigbee2MQTT version of the Hue dimmer blueprint (`hue_dimmer_z2m.yaml`). Self-contained: inline dimming, no `input_boolean` helpers and no script blueprint; one optional `input_number` for scene cycling.
- Double tap up / down sets the room to 100% / 10%.
- Switch LED mirrors the Hue room's color and brightness, with a configurable color when the light is off.
- Bidirectional sync between the switch and the Hue room, including changes made in the Hue app.
- Shower Humidity Fan blueprint for the VZM35-SN (`shower_fan_z2m.yaml`).
- mmWave Presence Diagnostics blueprint for the VZM32-SN (`mmwave_presence_diagnostics_z2m.yaml`) and a companion visualizer in `tools/mmwave_visualizer/`.
- Example helper files per integration (`examples/helpers_z2m.yaml`, `examples/helpers_zha.yaml`) and a Z2M flow diagram.
- README sections for the shower fan and mmWave blueprints.

### Changed
- Hue scenes are now discovered through the Hue room/zone device instead of by `scene.<room>_*` entity naming. Room names with spaces, renamed rooms and renamed scenes all work. Scenes created outside the Hue room (for example in Home Assistant) are no longer picked up by name.
- The room is derived from the selected Hue light group instead of being typed in.
- ZHA dimming script renamed to `dim_hue_room_lights_zha.yaml`.
- ZHA LED color now uses the same rule as Z2M: warm or cool white by color temperature, otherwise the light's hue.
- The Z2M blueprint expects a custom MQTT action sensor with `expire_after: 1` so repeated presses of the same button register.

### Fixed
- LED color template no longer fails for Hue groups whose lights report no color temperature or color attributes (for example dimmable-only bulbs).
- ZHA: the LED color set when the light turns on included template comment text alongside the number.
- mmWave visualizer: leaving an Area entity blank shifted the following areas onto the wrong zone and timeline color.
- Various Z2M fixes to MQTT topics, LED sync and warm/cool LED color.

---

## [0.1.1] - 2025-06-29

### Added
- Visual setup diagram illustrating integration between Hue, ZHA-based Inovelli Blue switches, and Home Assistant helpers.
- Added troubleshooting info to the README documenting debug log examples and helper usage.
- Scene cycling automation with dynamic scene discovery using `input_number.scene_index_<room>`.
- Paddle dimming functionality via `input_boolean.dimmer_<room>` and `dim_hue_room_lights.yaml` script.
- Support for directional hold (up/down) with brightness adjustment logic.
- YAML templates referencing `room_slug` to standardize helper and scene naming.

### Changed
- Updated debug logs to standardize format and include helper states, active scenes, and light modes (e.g. `xy_color`).
- README enhancements to clarify helper setup and event mapping.

### Fixed
- Color mismatch between identical Hue bulbs by preferring `xy_color` mode in automations.
- Scene restoration logic to better preserve original color temperature vs color mode.

### Deprecated
- Manual scene list definitions — replaced with dynamic discovery via templates.

---

## [0.1.0] - 2025-06-29

Initial release of:
- Dimmer automation blueprint
- Scene cycler blueprint
- Light level script

Includes basic README and prerequisites checklist.

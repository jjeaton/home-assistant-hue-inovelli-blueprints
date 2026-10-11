# Project: Home Assistant Hue + Inovelli Blueprints

## Key Architecture Decisions

### Z2M Blueprint (hue_dimmer_z2m.yaml)
- **mode: restart** is intentional — it terminates the inline dimming loop when release fires
- Dimming is inlined using `repeat...until: false` (infinite loop killed by restart)
- Sync is trigger-based (specific `from/to` state transitions) — no `input_boolean.sync_<room>` needed
- LED update and default scene run **before** switch turn-on in `light_on` handler to avoid being killed by the subsequent `switch_on` restart
- `switch_on` and `switch_off` handlers have state checks (`states(light_entity) != 'on'`) to prevent loops
- Z2M action freshness condition (`< 30s`) prevents spurious triggers after HA restart
- MQTT payload for LED uses Z2M property names: `ledColorWhenOn`, `ledIntensityWhenOn`, `ledColorWhenOff`, `ledIntensityWhenOff`

### ZHA Blueprint (hue_dimmer_zha.yaml)
- No ZHA test hardware available, so changes here are untested on a real switch
- Scene discovery and the LED color rule were brought in line with the Z2M blueprint; the helper/script design is unchanged

## Helper Requirements
- Z2M: Zero required helpers for on/off/dimming/sync
- Z2M: One optional `input_number` helper for scene cycling (any entity name)
- ZHA: Still requires `input_boolean.dimmer_<room>`, `input_boolean.sync_<room>`, `input_number.scene_index_<room>`, and separate script blueprint

## Scene Discovery
- Scenes are found via the Hue room/zone device: `device_entities(device_id(light_entity))` filtered to `scene.*`
- No dependency on room or scene naming, so room names with spaces and renamed entities work
- ZHA still derives `room` from the Hue light entity (`light_entity.split('.')[1]`) to build its helper entity IDs

## Z2M LED Control
- LED colors set via MQTT publish to `zigbee2mqtt/<z2m_device_name>/set`
- User provides `z2m_device_name` as a text input (friendly name in Z2M dashboard)
- Action sensor: a custom MQTT sensor with `expire_after: 1` (see README) so repeated presses of the same action re-trigger; newer Z2M versions no longer auto-create one
- LED color rule (same in both blueprints): color temp ≤ 4000 K → 24 (warm), above → 170 (cool), otherwise hue scaled to 0–255, fallback 24; read via `state_attr()` so missing attributes don't error

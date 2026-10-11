# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Home Assistant blueprints for Inovelli Blue Series switches, plus one standalone Python diagnostic tool. There is no build step, package manifest, linter config, CI, or test suite. A blueprint is "run" by importing it into a Home Assistant instance and creating an automation/script from it, so behaviour changes can only be truly verified on real hardware.

`origin` is a fork (`jjeaton/...`) of `Ratoka/...`. The README import badges point at the fork, so they must be reverted to `Ratoka` in any pull request to upstream; the Blueprint Exchange thread and credits still refer to upstream. Import URLs are tied to file paths on `main`, so renaming or moving a blueprint file breaks existing users' re-import links and the README badges.

## Commands

Syntax-check all YAML (dependency-free; Ruby's parser tolerates HA's `!input` tag, PyYAML's `safe_load` does not):

```sh
ruby -ryaml -e 'ARGV.each { |f| YAML.load_file(f); puts "ok #{f}" }' blueprints/*/inovelli/*.yaml examples/*.yaml
```

This only catches YAML errors. It does not validate the blueprint schema, selectors, or Jinja templates; those only fail when HA loads the blueprint.

mmWave visualizer (Dash app, serves on `0.0.0.0:8050`):

```sh
pip install -r tools/mmwave_visualizer/requirements.txt
python tools/mmwave_visualizer/app.py
```


## Layout

- `blueprints/automations/inovelli/hue_dimmer_z2m.yaml`: Hue room control from a Zigbee2MQTT switch. Self-contained. The actively developed one.
- `blueprints/automations/inovelli/hue_dimmer_zha.yaml` + `blueprints/scripts/inovelli/dim_hue_room_lights_zha.yaml`: the ZHA equivalent, which needs the script blueprint and helper entities.
- `blueprints/automations/inovelli/shower_fan_z2m.yaml` (VZM35-SN fan, humidity driven) and `mmwave_presence_diagnostics_z2m.yaml` (VZM32-SN presence notifications): independent of the Hue blueprints.
- `tools/mmwave_visualizer/app.py`: single-file companion to the mmWave blueprint.
- `examples/helpers_z2m.yaml`, `examples/helpers_zha.yaml`: the helper configs the README links to.
- `docs/*.drawio`: flow diagrams (ZHA and Z2M). `docs/assets/DeviceFlow.png` is the image embedded in the README.

## The two Hue dimmer blueprints

Both do the same job (tap on/off, double-tap 100%/10%, hold-to-dim, config-button scene cycling, LED mirroring, bidirectional switch/light sync) but are built differently. A feature or fix in one does not carry over to the other, and there is no ZHA test hardware, so ZHA changes can only be checked by reasoning and by rendering templates.

| | Z2M | ZHA |
|---|---|---|
| Button input | state trigger on an MQTT action sensor (`up_single`, `down_held`, `config_single`, ...) | `zha_event` with `command` like `button_2_press`; button 1 = down, 2 = up, 3 = config |
| Dimming | inline `repeat ... until: false` loop, ended by `mode: restart` when the release action arrives | sets `input_boolean.dimmer_<room>`, starts the script blueprint, which loops `while` the flag is on; release clears the flag |
| Sync | per-handler, driven by explicit `from`/`to` state triggers | handlers only set `input_boolean.sync_<room>`; a second top-level `choose` at the end reconciles both light and switch to that flag on every run |
| LED | `mqtt.publish` to `zigbee2mqtt/<z2m_device_name>/set` with `ledColorWhenOn`, `ledIntensityWhenOn`, `ledColorWhenOff`, `ledIntensityWhenOff` | `number.set_value` on `number.<switch>_default_all_led_{on,off}_{color,intensity}`, where `<switch>` is derived from the device's entities |
| Helpers | none required; one optional `input_number` of any name, passed as an input | `input_boolean.dimmer_<room>`, `input_boolean.sync_<room>`, `input_number.scene_index_<room>`, all found by naming convention |

### Shared conventions

- **Scene discovery.** The Hue integration attaches a room/zone's light group and its scenes to the same device, so `available_scenes` is `device_entities(device_id(light_entity))` filtered to `scene.*`, sorted, and indexed by the `input_number`. It does not depend on room or scene naming. The template is duplicated in both blueprints.
- **Room derivation (ZHA only).** `room_lower` comes from the Hue group entity ID (`light.living_room` gives `living_room`) and is used to build the helper entity IDs, so a mismatch fails silently.
- **Inputs are re-exposed as `variables:`.** `!input` cannot appear inside a Jinja template, so every input used in a template is copied into the top-level `variables:` block first. A new input needs both.
- **Optional entity inputs** use `default: ""` and are tested with `!= ''`.
- **Legacy HA syntax throughout** (`trigger:`/`action:` top-level keys, `platform:`, `service:`). Match it rather than mixing in the newer `triggers:`/`actions:`/`action:` forms.
- **LED colour scale** is Inovelli's 0-255 hue wheel. 24 is warm white, 170 is cool white.
- **The LED colour template is copy-pasted six times** (Z2M: `config_single`, `config_held`, `light_on`, `light_settled`; ZHA: light-on and light-state-change handlers): colour temp at or below 4000 K maps to 24, above to 170, otherwise `hs_color` hue scaled to 0-255, else 24. It reads attributes with `state_attr()` because lights that lack colour support omit those attributes entirely, and `states[...].attributes.x | float` errors on a missing one. It has to be evaluated after each handler's delay, so it cannot simply be hoisted into the top-level `variables:`. Edit all copies together.

### Z2M specifics that are easy to break

- **`mode: restart` is load-bearing.** It is what stops the dimming loop. The side effect is that any new trigger kills the run in progress, including triggers the run caused itself (turning the switch on fires `switch_on`). Handlers are therefore ordered so the LED publish and default scene happen *before* the step that can cause a retrigger. Keep that ordering when editing.
- **Loop guards.** `switch_on`/`switch_off` only act when the Hue light is not already in that state.
- **`light_settled` is blocked while a run is in progress.** These triggers (attribute changes on `brightness` / `color_temp_kelvin`, debounced `for: 800ms`) would otherwise restart the automation mid-dim whenever Hue is slow to report a step. A top-level condition checks `this.attributes.current`; a failed top-level condition does not restart a running automation, whereas a condition inside `choose` would.
- **The dimming loop sets absolute targets** (`dim_start` plus step times `repeat.index`, clamped to 1-100), not `brightness_step_pct`, so it does not depend on Hue's reported state keeping up. It ends itself at either limit.
- **Two different LED things.** `ledIntensityWhenOn` is how brightly the bar glows. The bar's *height* is the switch's own dim level, which the switch ramps by itself while the paddle is held; the blueprint sets it by calling `light.turn_on` on the switch entity with the room's brightness (Smart Bulb mode keeps the load at full power).
- **Action sensor.** The README requires a hand-made MQTT sensor with `expire_after: 1`, so the state returns to `unavailable` and a repeat of the same action registers as a state change. That is why the action trigger has no `not_from`. The `< 30s` freshness condition drops stale actions replayed after an HA restart.

### ZHA script blueprint

`dim_hue_room_lights_zha.yaml` declares `room`, `direction` and `brightness_step` as blueprint inputs, but its sequence never reads them with `!input`. The values arrive as run-time variables from the automation's `script.turn_on` call (`data.variables`), which is why the README tells users to leave the script's inputs empty.

## mmWave visualizer

A Dash app with all layout, callbacks and figure builders in one file. It pulls 24 h of history from HA's REST API (`/api/history/period`) using a URL and long-lived token typed into the UI (never persisted), draws the four detection zones as 3D boxes from manually entered bounds, and generates Interference Area JSON / MQTT commands for a selected time window. The coordinate system and room presets it assumes are defined in the description block of `mmwave_presence_diagnostics_z2m.yaml`; keep the two consistent.

## Docs to keep in step

- `README.md` is the user-facing spec for all the blueprints. Input names, defaults, helper entity IDs and setup steps there must be updated alongside any blueprint change that touches them, as should the matching file in `examples/`.
- `CHANGELOG.md` has no tagged versions after 0.1.1; later work is collected under `[Unreleased]`. Add user-visible changes there.
- `memory/MEMORY.md` is a design-notes file checked into the repo (not Claude Code's memory directory). Keep it in step when the Z2M design changes.
- `docs/assets/DeviceFlow.png` shows the ZHA flow only and cannot be regenerated without draw.io; the Z2M flow exists only as `docs/InovelliHueFlow_z2m.drawio`.

# Adam's IKEA Bilresa Blueprint

A Home Assistant automation blueprint for the IKEA BILRESA Matter scroll-wheel remote (3 channels).
Fork of [jhol-byte's original blueprint](https://community.home-assistant.io/t/ikea-bilresa-scroll-wheel-blueprint-matter/965365)
with three optional additions. Everything from the original works unchanged, and every new feature is off by default.

[![Open your Home Assistant instance and show the blueprint import dialog with this blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fadgsmarthome%2Fadams-ikea-bilresa-blueprint%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fadgsmarthome%2Fadams_ikea_bilresa_blueprint.yaml)

Or manually: **Settings → Automations & Scenes → Blueprints → Import Blueprint** and paste
`https://github.com/adgsmarthome/adams-ikea-bilresa-blueprint/blob/main/blueprints/automation/adgsmarthome/adams_ikea_bilresa_blueprint.yaml`

Requires Home Assistant 2024.10 or newer. The colour modes need `script.light_color_temp_helper` and `script.light_color_hs_helper`, the same as the original.

## New features

### 1. Scroll acceleration (fine / coarse)
*Section: "Scroll acceleration (fine / coarse)"*

In **relaxed** mode the remote reports how many clicks one turn had (1–8). A turn below the
**fast-spin threshold** is *fine* and a turn at or above it is *coarse*. Each level has its own **fixed step (%)**,
and a separate **per click** toggle sets whether that step is multiplied by the number of clicks.

| Defaults | Result |
|---|---|
| Threshold 5, fine 2 % **per click**, coarse 10 % **fixed** | 1–4 clicks → 2 / 4 / 6 / 8 %, 5–8 clicks → 10 % |

Steps are in dimmer percent. Colour temp, hue and volume scale the same way relative to their own step settings.
User-defined scroll actions also get `scroll_speed`, `scroll_step_pct`, `scroll_units` and `scroll_multiplier`.
Instant mode is not affected, because it reports one click at a time.

**Separate dim step for instant mode:** *Global light settings → Dimmer step percentage (instant mode)*.
`0` uses the relaxed-mode value.

### 2. Range-limit feedback ("duck")
*Section: "Range-limit feedback (duck)"*

If a light is **already** at the end of its range and you keep scrolling in that direction, it gives you a short cue and then returns:

| Situation | Cue (defaults) |
|---|---|
| At 100 % brightness, scrolling up | dips 30 points for 0.4 s |
| At minimum brightness, scrolling down | bumps **up** 15 points (dipping could turn it off) |
| At max / min colour temperature | dips (or bumps, if the light is too dim to dip visibly) |
| Media player at max volume, scrolling up (optional) | volume dips 10 points |

Top and bottom have separate switches and amounts. Hold time, fade, number of pulses (1–3), and
"only one cue per spin" are configurable too. Lights that hit the limit together pulse together.

### 3. Busy indicator
*Section: "Busy indicator"*

The automation runs in `queued` mode, so while one action is still running (for example a script that waits a minute),
every new press waits in line. With the busy indicator on, if an action is still running after **Show after**
(default 2 s), the indicator light pulses (or stays solid) in a colour you choose until the action finishes.
It then restores its previous state, turns off, or stays as it is, depending on your setting.

* Choose which actions are watched: click, double, triple, long/on-hold, or user-defined scroll.
  Long-click is off by default so on-hold dimming doesn't trigger it.
* You can set the colour, brightness, pulse low level, pulse period, and a safety timeout.
* Optional **extra start/end actions** let you signal busy on something that isn't a light.
* An action only counts as busy if the automation waits for it. Calling `script.my_script` directly waits, but `script.turn_on` doesn't.

### 4. Turn-on brightness
*Section: "Global light settings"*

With **Dim turns lights on** enabled, scrolling up on a light that is off normally brings back its previous brightness.
Set **Turn-on brightness** (e.g. `2`) to always start at that level instead. `0` keeps the original behaviour.

## Known limits
* Acceleration only works in relaxed mode.
* Hue has no end stop (it wraps around), so it has no limit cue.
* When the busy indicator is watching an action and that action uses `stop`, the indicator stays on until the safety timeout.

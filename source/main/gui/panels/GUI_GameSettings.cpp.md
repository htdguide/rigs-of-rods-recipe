# source/main/gui/panels/GUI_GameSettings.cpp

> Tab contents and the few settings that need side effects when changed.

**Needs** — [`GUI_GameSettings.h`](GUI_GameSettings.h.md) · [`AppContext.h`](../../AppContext.h.md) · [`resources/CacheSystem.h`](../../resources/CacheSystem.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`audio/SoundManager.h`](../../audio/SoundManager.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_GameSettings.h`](GUI_GameSettings.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_GameSettings.h`](GUI_GameSettings.h.md). Most rows are plain bound widgets ([`GUIUtils`](../GUIUtils.cpp.md#settings-bound-controls)); only side effects are called out.

## State

See header.

## Tabs

**Render System** (restart required): renderer choice (stored as an override); every mutable renderer option with choices, except NVPerfHUD, colour depth, fixed pipeline, floating-point mode, resource policy, VSync interval, sRGB; values listed in reverse order. A valid change is saved to the renderer config and shows a yellow "You must restart" banner (window grows to fit).

**General**: language (from installed catalogs; applying re-initialises translations); country (two-letter codes from the flag icon set — the protocol carries 2 letters); screenshot format png/jpg; extra mod path; skip main menu; async physics; disable online API; *Update cache* (closes, requests cache update, returns to menu).

**Gameplay**: gearbox mode (auto, semi-auto, manual, manual stick, manual ranges); engines spawn running; replay mode (+ length, stepping); realistic forward commands; races; no intra-/inter-vehicle collisions; Discord presence; quickload confirmation; vehicle tuning.

**UI**: UI preset (tooltip shows the preset table; applying writes its values); default truck and boat dashboards (button opens the dashboard selector, remembering which category is being chosen); imperial units; live-repair controls; vehicle buttons; full-size help image; map icons (+ declutter).

**Graphics**: light sources (none, no light sources, current vehicle head only, all vehicles head only, all lights); shadows (none, PSSM — restart; + optimisations, PSSM quality 0–3); sky (sandstorm, Caelum, SkyX; sight range 100–5000 m except SkyX); texture filter (none, bilinear, trilinear, anisotropic + 1/2/4/8/16); vegetation (none, 20 %, 50 %, full); water (none, basic, reflect, full fast, full HQ, Hydrax); FPS limit 0–240; particles; skidmarks; auto mesh LOD; realtime reflections (+ rate 0–2); video cameras; water waves; alternate vehicle materials; exterior camera mode (none, static, pitching); static camera height 1–50; exterior/interior FOV 10–120.

**Audio** (with audio support): output device (from the audio library's device list); creak; EFX (+ reverb engine none/reverb/EAX reverb, obstruction (+ force inside vehicles), occlusion, directed sounds, early-reflection panning for EAX, engine-controlled environment (+ default and forced reverb preset names)); menu music; master volume 0–1; Doppler 0–10.

**Controls**: input grab mode (none, all, dynamic) — a change requests input re-initialisation and shows a "restarting input" notice; analog smoothing and sensitivity 0.5–2; blinker lock range 0.1–1; arcade controls; invert orbit camera; force feedback (+ camera, centre, master, stress gains); OutGauge (+ IP, port, ID, delay).

**Diagnostic** (red warning): preselected terrain / vehicle / config / enter; spawner report; node import/stats logging; debug camera, mass, reflections, video cameras, textures; hide broken beams, wheel info, wheels, nodes; echo log to console; log beam break/deform/trigger; allow window resize; retained-GUI log file; *Rebuild cache* (purge, return to menu).

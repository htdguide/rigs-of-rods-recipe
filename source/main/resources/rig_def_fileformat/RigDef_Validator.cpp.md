# source/main/resources/rig_def_fileformat/RigDef_Validator.cpp

> The validation rules.

**Needs** — [`RigDef_Validator.h`](RigDef_Validator.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`physics/SimConstants.h`](../../physics/SimConstants.h.md) · [`Application.h`](../../Application.h.md) · [`system/Console.h`](../../system/Console.h.md)
**Used by** — callers of [`RigDef_Validator.h`](RigDef_Validator.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

These rules are part of a vehicle's acceptance criteria (see SYSTEM-REQUIREMENTS conformance). Messages go to the console, area ACTOR: fatal → error, error/warning → warning.

## State

See header twin.

## `Validate`

```text
FUNCTION validate()
  valid = check_submesh_groundmodel_unique() AND check_gearbox()
  FOR EACH selected module
    remove every line failing its check from: animators, shocks2, shocks3, commands2, triggers, flares2
  RETURN valid                     # line removals do not affect validity
```

## Configuration rules

- **`submesh_groundmodel` unique** — at most one selected module may contain it; otherwise fatal "Duplicate inline-section 'submesh_groundmodel'; found in modules: 'A' & 'B'".
- **Gearbox** — the first selected module that has an `engine` decides: its last engine line must have ≥ 1 forward gear, else fatal "Engine must have at least 1 forward gear."

## Line rules (failing lines are removed)

- **Animator** — must have at least one *source* flag (anything other than vis/inv/short-limit/long-limit) or an aero-engine source: "Animator: No animator source defined".
- **Shocks2 / Shocks3** — every numeric field must be ≥ −0.8 (i.e. not the −1 "missing" marker): "Invalid values in section 'shocksN', fields: …".
- **Command** — contract and extend keys must be non-zero and ≤ 84 (`MAX_COMMANDS`); negative keys are allowed.
- **Trigger** —
  - not an engine trigger (`E` absent):
    - ordinary trigger (no `B`, `A`, `h`, `H`): short-bound action must be in 1..84;
    - trigger blocker (`B`/`A`, no hook toggle): both actions must be ≥ 0;
    - hook toggles (`h`/`H`): no numeric check.
  - engine trigger: must not also be a blocker, hook toggle, or `s` switch.
- **Flare2** — control number in −1..500; blink delay in −2..60000 ms.

**Notes** — a video-camera check (positive texture size, 0 ≤ near ≤ far, mode ≥ −2, role −1..1) exists but is never invoked; the role check would also wrongly reject the valid role 2. A rebuild should either wire it up with role range −1..2 or leave it out.

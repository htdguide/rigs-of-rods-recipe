# source/main/physics/SimData.h

> The soft-body vocabulary: nodes, beams, shocks, wheels, hooks, ropes, ties, commands, flares, collision boxes, ground models, free forces, and the spawn/modify/link request records.

**Needs** — [`Application.h`](../Application.h.md) · [`ForwardDeclarations.h`](../ForwardDeclarations.h.md) · [`SimConstants.h`](SimConstants.h.md) · [`utils/BitFlags.h`](../utils/BitFlags.h.md) · [`CmdKeyInertia.h`](CmdKeyInertia.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md)
**Used by** — [`GameContext.h`](../GameContext.h.md) · [`gameplay/AutoPilot.cpp`](../gameplay/AutoPilot.cpp.md) · [`gameplay/Landusemap.h`](../gameplay/Landusemap.h.md) · [`gameplay/SceneMouse.h`](../gameplay/SceneMouse.h.md) · [`gfx/SimBuffers.h`](../gfx/SimBuffers.h.md) · [`gfx/Skidmark.cpp`](../gfx/Skidmark.cpp.md) · [`gui/panels/GUI_FlexbodyDebug.cpp`](../gui/panels/GUI_FlexbodyDebug.cpp.md) · [`gui/panels/GUI_FlexbodyDebug.h`](../gui/panels/GUI_FlexbodyDebug.h.md) · [`gui/panels/GUI_FrictionSettings.cpp`](../gui/panels/GUI_FrictionSettings.cpp.md) · [`gui/panels/GUI_FrictionSettings.h`](../gui/panels/GUI_FrictionSettings.h.md) · [`gui/panels/GUI_MainSelector.h`](../gui/panels/GUI_MainSelector.h.md) · [`gui/panels/GUI_SurveyMap.h`](../gui/panels/GUI_SurveyMap.h.md) · [`gui/panels/GUI_VehicleInfoTPanel.cpp`](../gui/panels/GUI_VehicleInfoTPanel.cpp.md) · [`Actor.cpp`](Actor.cpp.md) · [`Actor.h`](Actor.h.md) · [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md) · [`ActorManager.h`](ActorManager.h.md) · [`ActorSpawner.h`](ActorSpawner.h.md) · [`SimData.cpp`](SimData.cpp.md) · [`SlideNode.cpp`](SlideNode.cpp.md) · [`physics/air/AeroEngine.h`](air/AeroEngine.h.md) · [`physics/air/AirBrake.cpp`](air/AirBrake.cpp.md) · [`physics/air/TurboJet.cpp`](air/TurboJet.cpp.md) · [`physics/air/TurboJet.h`](air/TurboJet.h.md) · [`physics/air/TurboProp.cpp`](air/TurboProp.cpp.md) · [`physics/air/TurboProp.h`](air/TurboProp.h.md) · [`physics/collision/Collisions.h`](collision/Collisions.h.md) · [`physics/collision/DynamicCollisions.cpp`](collision/DynamicCollisions.cpp.md) · [`physics/collision/DynamicCollisions.h`](collision/DynamicCollisions.h.md) · [`physics/flex/FlexAirfoil.cpp`](flex/FlexAirfoil.cpp.md) · [`physics/flex/FlexAirfoil.h`](flex/FlexAirfoil.h.md) · [`physics/flex/FlexBody.cpp`](flex/FlexBody.cpp.md) · [`physics/flex/FlexBody.h`](flex/FlexBody.h.md) · [`physics/flex/FlexMesh.cpp`](flex/FlexMesh.cpp.md) · [`physics/flex/FlexMesh.h`](flex/FlexMesh.h.md) · [`physics/flex/FlexMeshWheel.cpp`](flex/FlexMeshWheel.cpp.md) · [`physics/flex/FlexObj.h`](flex/FlexObj.h.md) · [`physics/flex/Locator_t.h`](flex/Locator_t.h.md) · [`physics/water/Buoyance.cpp`](water/Buoyance.cpp.md) · [`physics/water/ScrewProp.cpp`](water/ScrewProp.cpp.md) · [`physics/water/ScrewProp.h`](water/ScrewProp.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`resources/odef_fileformat/ODefFileFormat.h`](../resources/odef_fileformat/ODefFileFormat.h.md) · [`scripting/bindings/ActorAngelscript.cpp`](../scripting/bindings/ActorAngelscript.cpp.md) · [`scripting/bindings/AircraftEngineAngelscript.cpp`](../scripting/bindings/AircraftEngineAngelscript.cpp.md) · [`scripting/bindings/AutopilotAngelscript.cpp`](../scripting/bindings/AutopilotAngelscript.cpp.md) · [`scripting/bindings/MsgQueueAngelscript.cpp`](../scripting/bindings/MsgQueueAngelscript.cpp.md) · [`scripting/bindings/ScrewpropAngelscript.cpp`](../scripting/bindings/ScrewpropAngelscript.cpp.md) · [`scripting/bindings/TurbojetAngelscript.cpp`](../scripting/bindings/TurbojetAngelscript.cpp.md) · [`scripting/bindings/TurbopropAngelscript.cpp`](../scripting/bindings/TurbopropAngelscript.cpp.md)
**Tier floor** — T2: `node_t` and `beam_t` must be flat, unboxed, contiguous records

## Purpose

Everything the simulation, the network or the player can change about an actor is described here; visuals live separately in `gfx/GfxData.h`. The physics loop streams over arrays of these records at 2 kHz, so their layout is a performance requirement: one contiguous array of nodes and one of beams per actor, with beams referring to nodes directly.

## State — enums

```text
ENUM CollisionEventFilter = NONE | ALL | AVATAR | TRUCK | TRUCK_WHEELS | AIRPLANE | BOAT
ENUM BeamType   = NORMAL | HYDRO (actuated or shock) | VIRTUAL (no mass contribution, never drawn)
ENUM SpecialBeam (aka "bounded") = NOSHOCK | SHOCK1 (shocks, wheel beams) | SHOCK2 | SHOCK3 | TRIGGER | SUPPORTBEAM | ROPE
ENUM HookState  = UNLOCKED | PRELOCK (pulling in) | LOCKED
ENUM ActorType  = NOT_DRIVEABLE 0 | TRUCK 1 | AIRPLANE 2 | BOAT 3 | MACHINE 4 | AI 5    # stored in the mod cache
ENUM BlinkType  = NONE | LEFT | RIGHT | WARN
ENUM ActorState = LOCAL_SIMULATED | NETWORKED_OK | NETWORKED_HIDDEN | LOCAL_REPLAY | LOCAL_SLEEPING | DISPOSED
ENUM AeroEngineType = UNKNOWN | XPROP | TURBOJET
ENUM LocalizerType  = VERTICAL | HORIZONTAL | NDB | VOR
ENUM EngineTriggerType = CLUTCH 0 | BRAKE 1 | ACC 2 | RPM 3 | SHIFTUP 4 | SHIFTDOWN 5

FLAGS HydroFlags  = SPEED, DIR, AILERON, RUDDER, ELEVATOR, REV_AILERON, REV_RUDDER, REV_ELEVATOR   (bits 1..8)
FLAGS AnimFlags   = AIRSPEED, VVI, ALTIMETER, AOA, FLAP, AIRBRAKE, ROLL, PITCH, THROTTLE, RPM, ACCEL, BRAKE,
                    CLUTCH, TACHO, SPEEDO, PBRAKE, TURBO, SHIFTER, AETORQUE, AEPITCH, AESTATUS, TORQUE,
                    HEADING, DIFFLOCK, STEERING, EVENT, AILERONS, ARUDDER, BRUDDER, BTHROTTLE, PERMANENT, ELEVATORS (bits 1..32)
FLAGS AnimModes   = ROTA_X, ROTA_Y, ROTA_Z, OFFSET_X, OFFSET_Y, OFFSET_Z, AUTOANIMATE, NOFLIP, BOUNCE
FLAGS ShockFlags  = NORMAL(1), LACTIVE(3), RACTIVE(4), ISSHOCK2(5), ISSHOCK3(6), SOFTBUMP(7), ISTRIGGER(8),
                    TRG_BLOCKER(9), TRG_CMD_SWITCH(10), TRG_CMD_BLOCKER(11), TRG_BLOCKER_A(12),
                    TRG_HOOK_UNLOCK(13), TRG_HOOK_LOCK(14), TRG_CONTINUOUS(15), TRG_ENGINE(16)
```

## State — soft body

```text
RECORD node_t                       # a point mass
  RelPosition   : Vec3              # relative to the actor's physics origin (keeps floats precise far from world origin)
  AbsPosition   : Vec3              # = origin + RelPosition
  Velocity, Forces : Vec3           # Forces is the accumulator for the current step
  mass, buoyancy, friction_coef, surface_coef, volume_coef : real
  pos           : NodeNum           # own index in the actor's node array
  coll_bbox_id  : int8 = -1         # which collision sub-box it belongs to
  lockgroup     : int16             # -1 default, 9999 cannot be hooked
  flags: cab_node, rim_node, tyre_node, contacter, contactable, has_ground_contact, has_mesh_contact,
         immovable, loaded_mass, no_ground_contact, override_mass, under_water, no_mouse_grab, cinecam_node
  avg_collision_slip : real; last_collision_slip, last_collision_force : Vec3
  last_collision_gm  : ground model

RECORD beam_t                       # a damped spring between two nodes
  p1, p2        : node references   # p2 may belong to another actor (inter-actor beams)
  k, d          : spring, damping
  L             : current rest length (actuators and deformation change it)
  refL          : reference (spawn) length
  strength      : breaking stress
  maxposstress  : compressive yield stress (positive)
  maxnegstress  : tensile yield stress (negative)
  minmaxposnegstress : min(maxposstress, -maxnegstress, strength)   # fast "anything to check?" threshold
  stress        : last computed force magnitude (signed)
  plastic_coef  : 0..1
  shortbound, longbound : shock/command limits (fractions of L)
  detacher_group: int                # breaking a master beam (>0) breaks every beam with |group| equal
  bounded       : SpecialBeam; bm_type : BeamType
  inter_actor   : bool; locked_actor : actor
  disabled, broken : bool
  shock         : shock_t reference (for SHOCK*/TRIGGER beams)
  initial_beam_strength, default_beam_deform, default_beam_diameter : for reset/export
  debug_k, debug_d, debug_v          # last shock force components, for UI

RECORD shock_t
  beamid, flags
  trigger state: enabled, switch_state, boundary_t, cmdlong, cmdshort, last_debug_state
  shocks2/3 params: springin, dampin, springout, dampout, sprogin, dprogin, sprogout, dprogout,
                    splitin, dslowin, dfastin, splitout, dslowout, dfastout
  sbd_spring, sbd_damp, sbd_break   # the beam defaults in force at definition (used for bump-stop stiffness)
  shock_precompression              # export only

RECORD collcab_rate_t { rate: cycles to skip, distance: cycles since last check }   # adaptive cab collision cadence
```

## State — wheels and gameplay links

```text
RECORD wheel_t
  nodes[≤50] (tyre), rim_nodes[≤50]  # alternate between the two axle sides
  braking : WheelBraking; propulsion : WheelPropulsion
  arm_node, near_attach_node, axis_node_0, axis_node_1
  radius, rim_radius, width, mass
  speed (m/s), avg_speed, alb_coef, tc_coef, torque, last_torque, last_retorque, net_rp (accumulated rotation)
  is_detached : bool
  editing/export args: keyword, num_rays, rigidity_node, rim/simple spring & damping, side, media1, media2, beam_start
  debug: rpm, torque, vel, slip, force, scaled_cforce

RECORD wheeldetacher_t { wheel_id, detacher_group }

RECORD hook_t
  locked: HookState; group, lockgroup; selflock, autolock, nodisable
  maxforce, lockrange, lockspeed, timer, timer_preset, min_length (m)
  hook_node, lock_node, beam, locked_actor

RECORD ropable_t { node, pos, group, attached_ties, attached_ropes, multilock }
RECORD rope_t    { locked, group, beam, locked_ropable, locked_actor }
RECORD tie_t     { locked_actor, beam, locked_ropable, group, contract_speed, max_stress, min_length (fraction),
                   no_self_lock, tied, tying }
RECORD wing_t    { airfoil, scene node }

RECORD commandbeam_t
  beam_index, engine_coupling, center_length (fraction), speed, boundary_length (fraction)
  is_contraction, is_force_restricted, needs_engine, is_autocentering, plays_sound, is_1press, is_1press_center
  state (shared between the contraction and extension halves): auto_moving_mode (int8), pressed_center_mode, auto_move_lock

RECORD command_t                     # one command key
  commandValueState (-1 just stopped, 0, 1 just started), commandValue, triggerInputValue, playerInputValue
  trigger_cmdkeyblock_state
  beams : list<commandbeam_t>; rotators : list<int>   # rotator index +1, sign = direction
  description; rotator_inertia, command_inertia

RECORD hydrobeam_t { beam_index, ref_length, speed, flags (HydroFlags), anim_flags, anim_param, inertia }

RECORD rotator_t { needs_engine, nodes1[4], nodes2[4], axis1, axis2, angle, rate, force, tolerance, engine_coupling, debug_rate, debug_aerror }

RECORD flare_t
  noderef, nodex, nodey, offset (x,y,z); visual handles (scene node, billboard, light)
  type : FlareType; controlnumber (user lights 0..9); dashboard_link
  blinkdelay, blinkdelay_curr, blinkdelay_state, size, intensity
  uses_inertia (flares3), inertia

RECORD PropAnimKeyState { eventlock_present, event_active_prev, anim_active, event_id }
```

## `CmdKeyArray`

**Contract** — command keys are addressed **1..84**; index 0 is accepted (for compatibility) and aliases a real slot; **any negative index** is valid and maps to an on-demand "virtual" command key (legacy trigger content uses them as signals). Indices above 84 are a programming error.

## State — world and requests

```text
RECORD collision_box_t
  virt, refined (rotated), selfrotated, camforced, enabled
  event_filter, eventsourcenum
  lo, hi (world AABB); center, rot, unrot; selfcenter, selfrot, selfunrot; relo, rehi (local box)
  campos; debug_verts[8]; reverb_preset_name

RECORD ground_model_t              # a surface type (asphalt, gravel, mud, water…)
  va (adhesion velocity), ms (static friction), mc (sliding friction), t2 (hydrodynamic friction s/m),
  vs (Stribeck velocity), alpha (steady-state exponent), strength (ground strength),
  fluid_density, flow_consistency_index, flow_behavior_index, solid_ground_level, drag_anisotropy,
  fx_type, fx_colour, name, basename, particle_name, fx_particle_amount/min_velo/max_velo/fade/timedelta/velo_factor/ttl

ENUM FreeForceType = DUMMY | CONSTANT | TOWARDS_COORDS | TOWARDS_NODE | HALFBEAM_GENERIC | HALFBEAM_ROPE
RECORD FreeForce                   # script-defined force on one node
  id, type, magnitude, base_actor, base_node,
  const_direction (normalised), target_coords, target_actor, target_node,
  half-beam params: spring, damp, deform, strength, diameter, plastic_coef   (defaults = beam defaults)
  half-beam state: L, stress, minmaxposnegstress, maxposstress, maxnegstress
RECORD FreeForceRequest            # the same, as script-dictionary-friendly 64-bit ints/doubles
                                   # (note: default halfb_strength is BEAM_DEFORM, not BEAM_BREAK — original quirk)

RECORD ActorSpawnRequest
  instance_id (optional), cache_entry | filename ("bundle.zip:file.truck" allowed), config (module name)
  position, rotation, spawnbox, skin_entry, tuneup_entry, working_tuneup
  origin : UNKNOWN | CONFIG_FILE | TERRN_DEF | USER | SAVEGAME | NETWORK | AI
  debugview, net_username, net_color, net_peeropts, net_source_id, net_stream_id
  free_position (skip ground adjustment), enter (seat the player; default true), terrn_machine
  saved_state : JSON document (restored right after spawn)

RECORD ActorModifyRequest
  actor : instance id (not a handle — must be thread-safe)
  type : RELOAD | RESET_ON_INIT_POS | RESET_ON_SPOT | SOFT_RESPAWN | SOFT_RESET | RESTORE_SAVED | WAKE_UP | REFRESH_VISUALS
  saved_state, addonpart, addonpart_fname, softrespawn_position, softrespawn_rotation

ENUM ActorLinkingRequestType = LOAD_SAVEGAME | HOOK_LOCK | HOOK_UNLOCK | HOOK_TOGGLE | HOOK_MOUSE_TOGGLE | HOOK_RESET
                             | TIE_TOGGLE | TIE_RESET | ROPE_TOGGLE | ROPE_RESET | SLIDENODE_TOGGLE
RECORD ActorLinkingRequest { actor_instance_id, type, hook_group, hook_mousenode, tie_group, rope_group }
```

**Notes** — linking two actors (hooks, ties, ropes, slide nodes) rebuilds a global link table, so it is always requested through the message queue and executed on the main thread between physics steps.

## `ActorSimAttr`

Script-tunable internals (`Actor::setSimAttribute`), with the "safe value" ranges the original documents but does not enforce: traction control ratio (1–20), pulse time (0.00005–1 s), wheel-slip constant (0.25); engine shift-down/up rpm, torque, differential ratio, gear-ratio array ("reverse neutral fwd1 fwd2…"); every `engoption` parameter (inertia, type, clutch force, shift/clutch/post-shift time, stall/idle rpm, min/max idle mixture, braking torque); turbo v2 parameters (inertia factor, count, max rpm, operating rpm, blow-off valve on/min psi, wastegate on/max psi/threshold ±, anti-lag on/chance/min rpm/power). `ActorSimAttrToString` gives the enum names.

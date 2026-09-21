# source/main/physics/Actor.h

> A simulated soft-body object (vehicle, load, machine, aircraft, boat): its node/beam arrays, subsystems, gameplay state, and public API (also the script API `BeamClass`).

**Needs** — [`SimData.h`](SimData.h.md) · [`air/AeroEngine.h`](air/AeroEngine.h.md) · [`Differentials.h`](Differentials.h.md) · [`CmdKeyInertia.h`](CmdKeyInertia.h.md) · [`gameplay/AutoPilot.h`](../gameplay/AutoPilot.h.md) · [`gameplay/Engine.h`](../gameplay/Engine.h.md) · [`gameplay/TyrePressure.h`](../gameplay/TyrePressure.h.md) · [`gameplay/VehicleAI.h`](../gameplay/VehicleAI.h.md) · [`gui/DashBoardManager.h`](../gui/DashBoardManager.h.md) · [`gfx/GfxActor.h`](../gfx/GfxActor.h.md) · [`gfx/camera/PerVehicleCameraContext.h`](../gfx/camera/PerVehicleCameraContext.h.md) · [`water/ScrewProp.h`](water/ScrewProp.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`network/RoRnet.h`](../network/RoRnet.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md) · [`resources/rig_def_fileformat/RigDef_Prerequisites.h`](../resources/rig_def_fileformat/RigDef_Prerequisites.h.md)
**Used by** — [`AppContext.cpp`](../AppContext.cpp.md) · [`GameContext.cpp`](../GameContext.cpp.md) · [`audio/SoundManager.h`](../audio/SoundManager.h.md) · [`audio/SoundScriptManager.cpp`](../audio/SoundScriptManager.cpp.md) · [`gameplay/Character.cpp`](../gameplay/Character.cpp.md) · [`gameplay/CharacterFactory.cpp`](../gameplay/CharacterFactory.cpp.md) · [`gameplay/ChatSystem.cpp`](../gameplay/ChatSystem.cpp.md) · [`gameplay/CruiseControl.cpp`](../gameplay/CruiseControl.cpp.md) · [`gameplay/Engine.cpp`](../gameplay/Engine.cpp.md) · [`gameplay/Landusemap.cpp`](../gameplay/Landusemap.cpp.md) · [`gameplay/RepairMode.cpp`](../gameplay/RepairMode.cpp.md) · [`gameplay/Replay.cpp`](../gameplay/Replay.cpp.md) · [`gameplay/SceneMouse.cpp`](../gameplay/SceneMouse.cpp.md) · [`gameplay/TyrePressure.cpp`](../gameplay/TyrePressure.cpp.md) · [`gameplay/VehicleAI.cpp`](../gameplay/VehicleAI.cpp.md) · [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`gfx/GfxData.cpp`](../gfx/GfxData.cpp.md) · [`gfx/GfxScene.cpp`](../gfx/GfxScene.cpp.md) · [`gfx/GfxWater.cpp`](../gfx/GfxWater.cpp.md) · [`gfx/HydraxWater.cpp`](../gfx/HydraxWater.cpp.md) · [`gfx/ShadowManager.cpp`](../gfx/ShadowManager.cpp.md) · [`gfx/SimBuffers.cpp`](../gfx/SimBuffers.cpp.md) · [`gfx/Skidmark.cpp`](../gfx/Skidmark.cpp.md) · [`gfx/SkyManager.cpp`](../gfx/SkyManager.cpp.md) · [`gfx/SkyXManager.cpp`](../gfx/SkyXManager.cpp.md) · [`gfx/SurveyMapTextureCreator.cpp`](../gfx/SurveyMapTextureCreator.cpp.md) · [`gfx/camera/CameraManager.cpp`](../gfx/camera/CameraManager.cpp.md) · [`gui/DashBoardManager.cpp`](../gui/DashBoardManager.cpp.md) · [`gui/GUIManager.cpp`](../gui/GUIManager.cpp.md) · [`gui/GUIUtils.cpp`](../gui/GUIUtils.cpp.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`gui/panels/GUI_ConsoleView.cpp`](../gui/panels/GUI_ConsoleView.cpp.md) · [`gui/panels/GUI_ConsoleWindow.cpp`](../gui/panels/GUI_ConsoleWindow.cpp.md) · [`gui/panels/GUI_DirectionArrow.cpp`](../gui/panels/GUI_DirectionArrow.cpp.md) · [`gui/panels/GUI_FlexbodyDebug.cpp`](../gui/panels/GUI_FlexbodyDebug.cpp.md) · [`gui/panels/GUI_GameAbout.cpp`](../gui/panels/GUI_GameAbout.cpp.md) · [`gui/panels/GUI_GameChatBox.cpp`](../gui/panels/GUI_GameChatBox.cpp.md) · [`gui/panels/GUI_GameControls.cpp`](../gui/panels/GUI_GameControls.cpp.md) · [`gui/panels/GUI_LoadingWindow.cpp`](../gui/panels/GUI_LoadingWindow.cpp.md) · [`gui/panels/GUI_NodeBeamUtils.cpp`](../gui/panels/GUI_NodeBeamUtils.cpp.md) · [`gui/panels/GUI_ScriptMonitor.cpp`](../gui/panels/GUI_ScriptMonitor.cpp.md) · [`gui/panels/GUI_SimPerfStats.cpp`](../gui/panels/GUI_SimPerfStats.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](../gui/panels/GUI_SurveyMap.cpp.md) · [`gui/panels/GUI_TextureToolWindow.cpp`](../gui/panels/GUI_TextureToolWindow.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`gui/panels/GUI_VehicleInfoTPanel.cpp`](../gui/panels/GUI_VehicleInfoTPanel.cpp.md) · [`main.cpp`](../main.cpp.md) · [`network/OutGauge.cpp`](../network/OutGauge.cpp.md) · [`Actor.cpp`](Actor.cpp.md) · [`ActorExport.cpp`](ActorExport.cpp.md) · [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md) · [`ActorManager.cpp`](ActorManager.cpp.md) · [`ActorSlideNode.cpp`](ActorSlideNode.cpp.md) · [`ActorSpawner.cpp`](ActorSpawner.cpp.md) · [`ActorSpawnerFlow.cpp`](ActorSpawnerFlow.cpp.md) · [`Savegame.cpp`](Savegame.cpp.md) · [`SimData.cpp`](SimData.cpp.md) · [`SlideNode.cpp`](SlideNode.cpp.md) · [`physics/air/AirBrake.cpp`](air/AirBrake.cpp.md) · [`physics/air/TurboJet.cpp`](air/TurboJet.cpp.md) · [`physics/air/TurboProp.cpp`](air/TurboProp.cpp.md) · [`physics/collision/Collisions.cpp`](collision/Collisions.cpp.md) · [`physics/collision/DynamicCollisions.cpp`](collision/DynamicCollisions.cpp.md) · [`physics/collision/PointColDetector.cpp`](collision/PointColDetector.cpp.md) · [`physics/flex/FlexAirfoil.cpp`](flex/FlexAirfoil.cpp.md) · [`physics/flex/FlexFactory.cpp`](flex/FlexFactory.cpp.md) · [`physics/water/ScrewProp.cpp`](water/ScrewProp.cpp.md) · [`physics/water/Wavefield.cpp`](water/Wavefield.cpp.md) · [`resources/CacheSystem.cpp`](../resources/CacheSystem.cpp.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.cpp`](../resources/addonpart_fileformat/AddonPartFileFormat.cpp.md) · [`resources/odef_fileformat/ODefFileFormat.cpp`](../resources/odef_fileformat/ODefFileFormat.cpp.md) · [`resources/rig_def_fileformat/RigDef_File.cpp`](../resources/rig_def_fileformat/RigDef_File.cpp.md) · [`resources/rig_def_fileformat/RigDef_SequentialImporter.cpp`](../resources/rig_def_fileformat/RigDef_SequentialImporter.cpp.md) · [`resources/rig_def_fileformat/RigDef_Validator.cpp`](../resources/rig_def_fileformat/RigDef_Validator.cpp.md) · [`resources/tobj_fileformat/TObjFileFormat.cpp`](../resources/tobj_fileformat/TObjFileFormat.cpp.md) · [`resources/tuneup_fileformat/TuneupFileFormat.cpp`](../resources/tuneup_fileformat/TuneupFileFormat.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`scripting/LocalStorage.cpp`](../scripting/LocalStorage.cpp.md) · [`scripting/OgreScriptBuilder.cpp`](../scripting/OgreScriptBuilder.cpp.md) · [`scripting/ScriptEngine.cpp`](../scripting/ScriptEngine.cpp.md) · [`scripting/bindings/ActorAngelscript.cpp`](../scripting/bindings/ActorAngelscript.cpp.md) · [`scripting/bindings/CacheSystemAngelscript.cpp`](../scripting/bindings/CacheSystemAngelscript.cpp.md) · [`system/AppConfig.cpp`](../system/AppConfig.cpp.md) · [`system/ConsoleCmd.cpp`](../system/ConsoleCmd.cpp.md) · [`terrain/ProceduralRoad.cpp`](../terrain/ProceduralRoad.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md) · [`terrain/TerrainEditor.cpp`](../terrain/TerrainEditor.cpp.md) · [`terrain/TerrainGeometryManager.cpp`](../terrain/TerrainGeometryManager.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md) · [`utils/ForceFeedback.cpp`](../utils/ForceFeedback.cpp.md) · [`utils/MeshObject.cpp`](../utils/MeshObject.cpp.md)
**Tier floor** — T2

## Purpose

The "actor" is the unit of simulation. It is built by [`ActorSpawner`](ActorSpawner.cpp.md) from a truck document, stepped by [`ActorForcesEuler`](ActorForcesEuler.cpp.md) under the control of [`ActorManager`](ActorManager.cpp.md), drawn by `GfxActor`, and scripted through the same method names listed below (the order of methods is mirrored in the script binding). It is reference-counted because scripts hold actors. Actors are never destroyed mid-frame: deletion goes through a message, `dispose()` releases everything, and the object stays in memory in state DISPOSED until the last handle drops. Implementation: [`Actor.cpp`](Actor.cpp.md).

## State

Grouped; all arrays are per actor and sized at spawn.

```text
# identity & origin
instance_id, vector_index, driveable (ActorType), state (ActorState), design_name, filename, filehash,
section_config (chosen module), used cache entries (actor, skin, add-on parts, asset packs), working tuneup
net_source_id, net_stream_id, net timers/results; preloaded_with_terrain

# soft body
nodes[num_nodes] : node_t; nodes_id[] (file number or -1), nodes_name[] (named nodes)
nodes_default_loadweights[], nodes_override_loadweights[], nodes_spawn_offsets[] (tuned spawn layout),
nodes_options[], minimass[] (+ original), initial_node_masses[], initial_node_positions[]
beams[num_beams] : beam_t; initial_beam_defaults[] (k, d pairs); beams_invisible[]; beams_user_defined[]
inter_beams : list<beam> (hooks/ties/ropes currently attached to other actors)
shocks[], rotators[], wings[], hydros[], command_key (CmdKeyArray), ropes, ropables, ties, hooks,
flares, airbrakes, wheels[≤64], soundsources[≤128], aeroengines[≤8], screwprops[≤8]
cabs[3·≤3000] (triangle node triples), collcabs[] (colliding cabs) with inter/intra collision cadence,
buoycabs[] + types, cabs_buoy_cache_ids[], camera_rail[≤50]
node_to_node_connections, node_to_beam_connections   # adjacency, built at spawn
bounding boxes: whole, event-box eligible (within 15 m of camera node), predicted (pos + vel), per collision sub-box
origin : Vec3            # physics origin; nodes store positions relative to it

# mass
dry_mass, load_mass (+ originals), masscount, total_mass, initial_total_mass

# driving state
engine, autopilot, vehicle_ai, dashboard, brake_force, handbrake_force, brake (0..1), parking_brake,
trailer_parking_brake, wheel_speed, wheel_spin, avg_wheel_speed, top_speed,
hydro dir/aileron/rudder/elevator command & state, hydro_dir_wheel_display, speed_coupling,
aileron, rudder, elevator, aerial_flap (0..5), airbrake_intensity (0..5), fusedrag,
anti-lock: ratio, minspeed, mode, pulse_time/state/timer, nodash, notoggle
traction control: ratio, mode, pulse_time/state/timer, nodash, notoggle, wheelslip_constant (0.25)
cruise control: mode, can_brake, target_rpm, target_speed (+ lower limit), recent accelerations
speed limiter: enabled, limit
axle diffs[≤33], wheel diffs[≤32], transfer case, propelled wheel count/pairs/avg radius
stabiliser shock ratio/request/sleep; odometers; sleep_counter
light mask (RoRnet lightmask bits), flaregroups_no_import mask, blinker autoreset & lit states, flares mode snapshot

# cameras
num_cameras, camera nodes (pos/dir/roll per camera), main camera nodes, camera_dir_corr (quaternion),
cinecam nodes[≤10], current_cinecam, forced_cinecam (+ flags), custom_camera_node, extern camera mode/node,
camera_context, min_camera_radius, g-force accumulators (current / max, local to camera frame)

# linking
linked_actors : all actors connected directly or indirectly through hooks/ties/ropes/slide nodes
slidenodes, railgroups, slidenodes_locked

# node-beam editing (live tuning UI)
nb_* : scales and search intervals for beam/shock/wheel k & d, measurement buffers

# misc
gfx_actor, flexbody_tasks, replay recorder, buoyance, tyre_pressure, collision detectors (intra, inter),
mouse grab node/pos/force, rotation/translation/anglesnap requests (applied during live repair),
ongoing_reset, physics_paused, muted_by_peeropt, water_contact (+ previous), debug flags,
force-feedback sensors (accumulated body forces and hydro forces), incoming network update queue
```

## API groups

- **Networking** — `sendStreamSetup`, `sendStreamData`, `pushNetwork(data)`, `calcNetwork`.
- **Physics state** — position (average node position), rotation/heading, orientation, speed, g-forces, masses, node positions/masses/velocities/forces, wheel info, shock debug values, reset, resetPosition, softRespawn, rotation/translation/angle-snap requests, min/max height, height above ground, slide node helpers.
- **Physics editing** — `scaleTruck`, `setMass`, `setLoadedMass`, `setNodeMass`, `setNodeMassOptions`, `setSimAttribute`/`getSimAttribute`, `recalculateNodeMasses`, airbrake/flaps, `applyNodeBeamScales`, `searchBeamDefaults`, `updateInitPosition`, `propagateNodeBeamChangesToDef`.
- **User interaction** — parking brake, TC, ABS, custom particles, locks/ties/ropes/slide-node toggles, forced cinecam, mouse drag, diff and transfer-case modes, cruise control, smoke.
- **Lights** — blink type, custom lights 0..9, beacons, headlights/high beams/fog/side lights, light mask get/set/import, counts.
- **Visual/audio updates** — skidmarks, prepare inside, flare states, visual update, dashboards, sound sources, mute.
- **Subsystems** — dashboard, AI, aero engines, autopilot, screw props, managed materials, replay, tyre pressure, engine.
- **Organisational** — names, file, resource group, type, section config, instance id, cache entries, tuneup management, authors, description.
- **Collision helpers** — `Intersects`, `resolveCollisions` (two forms), `GetNumActiveConnectedBeams`, bounding boxes, origin update.

Private step functions (`Calc*`) live in [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md) and [`Actor.cpp`](Actor.cpp.md).

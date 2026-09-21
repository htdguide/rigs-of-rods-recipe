# source/main/system/CVar.cpp

> Registers every built-in setting with its type, persistence and default; name lookup and typed assignment.

**Needs** — [`Application.h`](../Application.h.md) · [`Console.h`](Console.h.md) · [`CVar.h`](CVar.h.md)
**Used by** — callers of [`CVar.h`](CVar.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

The single table of all built-in settings. `Console::cVarSetupBuiltins` creates them all at startup (before the config file is read), filling the `App::` handles declared in [`Application.h`](../Application.h.md).

## State

The console's two maps (by short name, by long name) — see [`Console.h`](Console.h.md).

## `cVarCreate(name, long_name, flags, default)`

**Contract** — creates the variable (long name defaults to the short name when empty), assigns the default through `cVarAssign` if non-empty, and registers it under both names. Duplicate registration keeps the first.

## `cVarAssign(cvar, text)`

**Contract** — parse by type: bool accepts `true/yes/1`/`false/no/0`; int and float are locale-independent decimal; unparsable text yields 0/false. Text variables store the text as is.

## `cVarFind(name)` · `cVarSet(name, text)` · `cVarGet(name, flags)`

**Contract** — find looks up the short name, then the long name. Set assigns if found (and returns nothing useful). Get returns the existing variable or **creates a new one** with that name and the given flags — this is how user- and script-defined variables come into existence (e.g. unknown keys in `RoR.cfg`).

## `logUpdate` · `convertStr`

Behaviour described in [`CVar.h`](CVar.h.md#setvalnumber).

## Built-in settings

Every row below is created at startup. "archived" = saved to `RoR.cfg`; "no-log" = changes are not written to the log. Rows with a long name can also be addressed by it (and older config files use it). Enum-typed ints use the orders listed in [`Application.h`](../Application.h.md#settings-enums).

| Name | Long name | Type | Flags | Default |
|---|---|---|---|---|
| `app_state` |  | int |  | `0` |
| `app_language` | `Language` | text | archived | `en` |
| `app_country` | `Country` | text | archived | `us` |
| `app_skip_main_menu` | `SkipMainMenu` | bool | archived | `false` |
| `app_async_physics` | `AsyncPhysics` | bool | archived | `true` |
| `app_num_workers` | `NumWorkerThreads` | int | archived |  |
| `app_screenshot_format` | `Screenshot Format` | text | archived | `png` |
| `app_rendersys_override` | `Render system` | text | archived |  |
| `app_extra_mod_path` | `Extra mod path` | text | archived |  |
| `app_force_cache_purge` |  | bool | archived | `false` |
| `app_force_cache_update` |  | bool | archived | `false` |
| `app_disable_online_api` | `Disable Online API` | bool | archived | `false` |
| `app_config_long_names` | `Config uses long names` | bool | archived | `true` |
| `app_custom_scripts` |  | text | archived |  |
| `app_recent_scripts` |  | text | archived |  |
| `sim_state` |  | int |  | `0` |
| `sim_terrain_name` |  | text |  |  |
| `sim_terrain_gui_name` |  | text |  |  |
| `sim_spawn_running` | `Engines spawn running` | bool | archived | `true` |
| `sim_replay_enabled` | `Replay mode` | bool | archived | `false` |
| `sim_replay_length` | `Replay length` | int | archived | `200` |
| `sim_replay_stepping` | `Replay Steps per second` | int | archived | `1000` |
| `sim_realistic_commands` | `Realistic forward commands` | bool | archived | `false` |
| `sim_races_enabled` | `Races` | bool | archived | `true` |
| `sim_no_collisions` | `DisableCollisions` | bool | archived | `false` |
| `sim_no_self_collisions` | `DisableSelfCollisions` | bool | archived | `false` |
| `sim_gearbox_mode` | `GearboxMode` | int | archived |  |
| `sim_soft_reset_mode` |  | bool |  | `false` |
| `sim_quickload_dialog` |  | bool | archived | `true` |
| `sim_live_repair_interval` |  | float | archived | `2.f` |
| `sim_tuning_enabled` |  | bool | archived | `true` |
| `mp_state` |  | int |  | `0` |
| `mp_join_on_startup` | `Auto connect` | bool | archived | `false` |
| `mp_chat_auto_hide` | `Auto hide chat` | bool | archived | `true` |
| `mp_hide_net_labels` | `Hide net labels` | bool | archived | `false` |
| `mp_hide_own_net_label` | `Hide own net label` | bool | archived | `true` |
| `mp_pseudo_collisions` | `Multiplayer collisions` | bool | archived | `false` |
| `mp_server_host` | `Server name` | text | archived |  |
| `mp_server_port` | `Server port` | int | archived |  |
| `mp_server_password` | `Server password` | text | archived no-log |  |
| `mp_player_name` | `Nickname` | text | archived | `Player` |
| `mp_player_token` | `User Token` | text | archived no-log |  |
| `mp_api_url` | `Online API URL` | text | archived | `http://api.rigsofrods.org` |
| `mp_cyclethru_net_actors` |  | bool | archived | `false` |
| `remote_query_url` |  | text | archived | `https://v2.api.rigsofrods.org` |
| `diag_auto_spawner_report` | `AutoActorSpawnerReport` | bool | archived | `false` |
| `diag_camera` | `Camera Debug` | bool | archived | `false` |
| `diag_rig_log_node_import` | `RigImporter_LogAllNodes` | bool | archived | `false` |
| `diag_rig_log_node_stats` | `RigImporter_LogNodeStats` | bool | archived | `false` |
| `diag_truck_mass` | `Debug Truck Mass` | bool | archived | `false` |
| `diag_envmap` | `EnvMapDebug` | bool | archived | `false` |
| `diag_videocameras` | `VideoCameraDebug` | bool | archived | `false` |
| `diag_preset_terrain` | `Preselected Terrain` | text | archived |  |
| `diag_spawn_position` |  | text | archived |  |
| `diag_spawn_rotation` |  | text | archived |  |
| `diag_preset_vehicle` | `Preselected Truck` | text | archived |  |
| `diag_preset_veh_config` | `Preselected TruckConfig` | text | archived |  |
| `diag_preset_veh_enter` | `Enter Preselected Truck` | bool | archived | `false` |
| `diag_log_console_echo` | `Enable Ingame Console` | bool | archived | `false` |
| `diag_log_beam_break` | `Beam Break Debug` | bool | archived | `false` |
| `diag_log_beam_deform` | `Beam Deform Debug` | bool | archived | `false` |
| `diag_log_beam_trigger` | `Trigger Debug` | bool | archived | `false` |
| `diag_simple_materials` | `SimpleMaterials` | bool | archived | `false` |
| `diag_warning_texture` | `Warning texture` | bool | archived | `false` |
| `diag_hide_broken_beams` | `Hide broken beams` | bool | archived | `false` |
| `diag_hide_beam_stress` | `Hide beam stress` | bool | archived | `true` |
| `diag_hide_wheel_info` | `Hide wheel info` | bool | archived | `true` |
| `diag_hide_wheels` | `Hide wheels` | bool | archived | `false` |
| `diag_hide_nodes` | `Hide nodes` | bool | archived | `false` |
| `diag_terrn_log_roads` |  | bool | archived | `false` |
| `diag_actor_dump` |  | bool | archived | `false` |
| `diag_allow_window_resize` |  | bool | archived | `false` |
| `diag_use_mygui_logfile` |  | bool | archived | `false` |
| `diag_load_devel_scripts` |  | bool | archived | `false` |
| `diag_profiler_enabled` |  | bool |  | `false` |
| `diag_profiler_rate` |  | int | archived | `10` |
| `sys_process_dir` |  | text |  |  |
| `sys_user_dir` |  | text |  |  |
| `sys_config_dir` | `Config Root` | text |  |  |
| `sys_cache_dir` | `Cache Path` | text |  |  |
| `sys_thumbnails_dir` | `Thumbnails Path` | text |  |  |
| `sys_logs_dir` | `Log Path` | text |  |  |
| `sys_resources_dir` | `Resources Path` | text |  |  |
| `sys_profiler_dir` | `Profiler output dir` | text |  |  |
| `sys_savegames_dir` |  | text |  |  |
| `sys_screenshot_dir` |  | text |  |  |
| `sys_scripts_dir` |  | text |  |  |
| `sys_projects_dir` |  | text |  |  |
| `sys_repo_attachments_dir` |  | text |  |  |
| `cli_server_host` |  | text |  |  |
| `cli_server_port` |  | int |  | `0` |
| `cli_preset_vehicle` |  | text |  |  |
| `cli_preset_veh_config` |  | text |  |  |
| `cli_preset_terrain` |  | text |  |  |
| `cli_preset_spawn_pos` |  | text |  |  |
| `cli_preset_spawn_rot` |  | text |  |  |
| `cli_preset_veh_enter` |  | bool |  | `false` |
| `cli_force_cache_update` |  | bool |  | `false` |
| `cli_resume_autosave` |  | bool |  | `false` |
| `cli_custom_scripts` |  | text |  |  |
| `io_analog_smoothing` | `Analog Input Smoothing` | float | archived | `1.0` |
| `io_analog_sensitivity` | `Analog Input Sensitivity` | float | archived | `1.0` |
| `io_blink_lock_range` | `Blinker Lock Range` | float | archived | `0.1` |
| `io_ffb_enabled` | `Force Feedback` | bool | archived | `false` |
| `io_ffb_camera_gain` | `Force Feedback Camera` | float | archived |  |
| `io_ffb_center_gain` | `Force Feedback Centering` | float | archived |  |
| `io_ffb_master_gain` | `Force Feedback Gain` | float | archived |  |
| `io_ffb_stress_gain` | `Force Feedback Stress` | float | archived |  |
| `io_input_grab_mode` | `Input Grab` | int | archived | `1` |
| `io_arcade_controls` | `ArcadeControls` | bool | archived | `false` |
| `io_hydro_coupling` | `Keyboard Steering Speed Coupling` | bool | archived | `true` |
| `io_outgauge_mode` | `OutGauge Mode` | int | archived |  |
| `io_outgauge_ip` | `OutGauge IP` | text | archived | `192.168.1.100` |
| `io_outgauge_port` | `OutGauge Port` | int | archived | `1337` |
| `io_outgauge_delay` | `OutGauge Delay` | float | archived | `10.0` |
| `io_outgauge_id` | `OutGauge ID` | int | archived |  |
| `io_discord_rpc` | `Discord Rich Presence` | bool | archived | `true` |
| `io_invert_orbitcam` | `Invert orbit camera` | bool | archived | `false` |
| `audio_master_volume` | `Sound Volume` | float | archived | `1.0` |
| `audio_enable_creak` | `Creak Sound` | bool | archived | `false` |
| `audio_enable_obstruction` | `Obstruction of sounds` | bool | archived | `false` |
| `audio_enable_occlusion` | `Occlusion of sounds` | bool | archived | `false` |
| `audio_enable_directed_sounds` | `Directed sounds` | bool | archived | `false` |
| `audio_enable_reflection_panning` | `Pan reflections` | bool | archived | `false` |
| `audio_device_name` | `AudioDevice` | text | archived |  |
| `audio_doppler_factor` | `Doppler Factor` | float | archived | `1.0` |
| `audio_menu_music` | `MainMenuMusic` | bool | archived | `false` |
| `audio_enable_efx` | `Enable OpenAL EFX` | bool | archived | `true` |
| `audio_engine_controls_environmental_audio` | `Engine-controlled environm. audio` | bool | archived | `true` |
| `audio_efx_reverb_engine` | `OpenAL EFX Reverb Engine` | int | archived | `2` |
| `audio_default_efx_preset` | `OpenAL default EFX preset` | text | archived |  |
| `audio_force_listener_efx_preset` | `OpenAL forced listener EFX preset` | text | archived |  |
| `audio_force_obstruction_inside_vehicles` | `Force obstruction inside vehicles` | bool | archived | `false` |
| `audio_sim_pause_disables_doppler_effect` | `Disable Doppler effect on sim pause` | bool | archived | `true` |
| `gfx_flares_mode` | `Lights` | int | archived | `4` |
| `gfx_polygon_mode` | `Polygon mode` | int |  | `1` |
| `gfx_shadow_type` | `Shadow technique` | int | archived | `1` |
| `gfx_extcam_mode` | `External Camera Mode` | int | archived | `2` |
| `gfx_sky_mode` | `Sky effects` | int | archived | `1` |
| `gfx_texture_filter` | `Texture Filtering` | int | archived | `3` |
| `gfx_vegetation_mode` | `Vegetation` | int | archived | `3` |
| `gfx_sky_time_cycle` |  | bool |  | `false` |
| `gfx_sky_time_speed` |  | int |  | `300` |
| `gfx_water_mode` | `Water effects` | int | archived | `3` |
| `gfx_anisotropy` | `Anisotropy` | int | archived | `4` |
| `gfx_water_waves` | `Waves` | bool | archived | `false` |
| `gfx_particles_mode` | `Particles` | int | archived |  |
| `gfx_enable_videocams` | `gfx_enable_videocams` | bool | archived | `false` |
| `gfx_window_videocams` | `UseVideocameraWindows` | bool | archived | `false` |
| `gfx_surveymap_icons` | `Overview map icons` | bool | archived | `true` |
| `gfx_declutter_map` | `Declutter overview map` | bool | archived | `true` |
| `gfx_envmap_enabled` | `Reflections` | bool | archived | `true` |
| `gfx_envmap_rate` | `ReflectionUpdateRate` | int | archived | `1` |
| `gfx_shadow_quality` | `Shadows Quality` | int | archived | `2` |
| `gfx_skidmarks_mode` | `Skidmarks` | int | archived | `0` |
| `gfx_sight_range` | `SightRange` | int | archived | `5000` |
| `gfx_camera_height` | `Static camera height` | int | archived | `5` |
| `gfx_fov_external` |  | int |  | `60` |
| `gfx_fov_external_default` | `FOV External` | int | archived | `60` |
| `gfx_fov_internal` |  | int |  | `75` |
| `gfx_fov_internal_default` | `FOV Internal` | int | archived | `75` |
| `gfx_static_cam_fov_exp` |  | float | archived | `1.0` |
| `gfx_fixed_cam_tracking` |  | bool | archived | `false` |
| `gfx_fps_limit` | `FPS-Limiter` | int | archived | `0` |
| `gfx_speedo_imperial` | `gfx_speedo_imperial` | bool | archived | `false` |
| `gfx_flexbody_cache` | `Flexbody_UseCache` | bool | archived | `false` |
| `gfx_reduce_shadows` | `Shadow optimizations` | bool | archived | `true` |
| `gfx_enable_rtshaders` | `Use RTShader System` | bool | archived | `false` |
| `gfx_alt_actor_materials` | `Use alternate vehicle materials` | bool | archived | `false` |
| `gfx_auto_lod` | `Use OGREs Automatic Mesh LOD Generator` | bool | archived | `true` |
| `flexbody_defrag_enabled` |  | bool |  |  |
| `flexbody_defrag_const_penalty` |  | int |  | `7` |
| `flexbody_defrag_prog_up_penalty` |  | int |  | `3` |
| `flexbody_defrag_prog_down_penalty` |  | int |  | `1` |
| `flexbody_defrag_reorder_indices` |  | bool |  | `true` |
| `flexbody_defrag_reorder_texcoords` |  | bool |  | `true` |
| `flexbody_defrag_invert_lookup` |  | bool |  | `true` |
| `ui_show_live_repair_controls` |  | bool | archived | `true` |
| `ui_show_vehicle_buttons` | `Show vehicle buttons menu` | bool | archived | `true` |
| `ui_preset` |  | int | archived | `0` |
| `ui_hide_gui` |  | bool |  | `false` |
| `ui_default_truck_dash` |  | text | archived | `default_truck_digital.dashboard` |
| `ui_legacy_truck_renderdash` |  | text | archived | `rdd_classic.dashboard` |
| `ui_default_boat_dash` |  | text | archived | `default_boat.dashboard` |
| `ui_always_show_fullsize` |  | bool | archived | `false` |

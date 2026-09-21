# source/main/physics/SimConstants.h

> The physics timestep, capacity limits, and default material constants of the soft-body model.

**Needs** — nothing
**Used by** — [`gfx/GfxData.h`](../gfx/GfxData.h.md) · [`SimData.h`](SimData.h.md) · [`resources/otc_fileformat/OTCFileFormat.cpp`](../resources/otc_fileformat/OTCFileFormat.cpp.md) · [`resources/rig_def_fileformat/RigDef_File.cpp`](../resources/rig_def_fileformat/RigDef_File.cpp.md) · [`resources/rig_def_fileformat/RigDef_File.h`](../resources/rig_def_fileformat/RigDef_File.h.md) · [`resources/rig_def_fileformat/RigDef_Parser.cpp`](../resources/rig_def_fileformat/RigDef_Parser.cpp.md) · [`resources/rig_def_fileformat/RigDef_Serializer.cpp`](../resources/rig_def_fileformat/RigDef_Serializer.cpp.md) · [`resources/rig_def_fileformat/RigDef_Validator.cpp`](../resources/rig_def_fileformat/RigDef_Validator.cpp.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.cpp`](../resources/terrn2_fileformat/Terrn2FileFormat.cpp.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.h`](../resources/terrn2_fileformat/Terrn2FileFormat.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md)
**Tier floor** — T4

## Purpose

Every number here is either part of the simulation's character (tuned against thousands of community vehicles, so changing it changes how every vehicle drives) or a capacity limit that content relies on. A rebuild should reproduce them exactly.

## State

```text
PHYSICS_DT = 0.0005 s              # fixed step, 2000 Hz

# Capacity limits (per session / per actor)
MAX_ACTORS = 5000
MAX_WHEELS = 64            MAX_SUBMESHES = 500        MAX_TEXCOORDS = 3000
MAX_CABS = 3000            MAX_COMMANDS = 84          MAX_CAMERAS = 10
MAX_AEROENGINES = 8        MAX_SCREWPROPS = 8         MAX_SOUNDSCRIPTS_PER_TRUCK = 128
MAX_CPARTICLES = 10        MAX_CAMERARAIL = 50        MAX_CLIGHTS = 10   # user lights 1..10 on the wire

RAD_PER_SEC_TO_RPM = 9.5492965855137    # 60 / 2π
TRUCKFILEFORMATVERSION = 3

# Beam and node defaults
DEFAULT_SPRING           = 9,000,000     N/m
DEFAULT_DAMP             = 12,000        N·s/m
DEFAULT_RIGIDIFIER_SPRING= 1,000,000     DEFAULT_RIGIDIFIER_DAMP = 50,000   (unused legacy)
BEAM_DEFORM              = 400,000       N   # stress at which plastic deformation starts
BEAM_BREAK               = 1,000,000     N   # stress at which a beam breaks
BEAM_CREAK_DEFAULT       = 100,000
BEAM_PLASTIC_COEF_DEFAULT= 0
DEFAULT_BEAM_DIAMETER    = 0.05 m        BEAM_SKELETON_DIAMETER = 0.01 m
MIN_BEAM_LENGTH          = 0.1 m         INVERTED_MIN_BEAM_LENGTH = 10
DEFAULT_GRAVITY          = -9.807 m/s²
DEFAULT_DRAG             = 0.05          # air drag coefficient per node
DEFAULT_WATERDRAG        = 10.0
DEFAULT_COLLISION_RANGE  = 0.02 m
DEFAULT_MINIMASS         = 50 kg         # minimum node mass
IRON_DENSITY             = 7874 kg/m³
WHEEL_FRICTION_COEF      = 2.0           CHASSIS_FRICTION_COEF = 0.5
SPEED_STOP               = 0.2
STAB_RATE                = 0.025         # active-shock stabiliser rate
NODE_FRICTION/VOLUME/SURFACE_COEF_DEFAULT = 1.0
NODE_LOADWEIGHT_DEFAULT  = 10 kg
SUPPORT_BEAM_LIMIT_DEFAULT = 4.0         # support beams break beyond 4× extension
ROTATOR_FORCE_DEFAULT    = 10,000,000    ROTATOR_TOLERANCE_DEFAULT = 0
HOOK_FORCE_DEFAULT = 10,000,000   HOOK_RANGE_DEFAULT = 0.4 m
HOOK_SPEED_DEFAULT = 0.00025      HOOK_LOCK_TIMER_DEFAULT = 5 s
NODE_LOCKGROUP_DEFAULT = -1       DEFAULT_DETACHER_GROUP = 0
DEFAULT_SPEEDO_MAX_KPH = 140
FLAP_ANGLES = [0, -0.07, -0.17, -0.33, -0.67, -1.0]   # aircraft flap positions 0..5 (fraction of max deflection)
```

**Notes** — 2 kHz is not arbitrary: beams are stiff springs (k ≈ 9·10⁶ N/m) on light nodes (tens of kg), and an explicit integrator is only stable for Δt well below √(m/k) (≈ 2 ms for 50 kg). Halving the rate makes stiff vehicles explode; the "anti-explosion" reset in [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md#calcnodes) is the safety net.

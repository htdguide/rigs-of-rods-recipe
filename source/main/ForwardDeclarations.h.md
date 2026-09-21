# source/main/ForwardDeclarations.h

> Names every major type once, defines the typed-index conventions, and fixes which objects are shared handles.

**Needs** — [`utils/memory/RefCountingObjectPtr.h`](utils/memory/RefCountingObjectPtr.h.md)
**Used by** — [`Application.h`](Application.h.md) · [`gameplay/Character.h`](gameplay/Character.h.md) · [`gfx/EnvironmentMap.h`](gfx/EnvironmentMap.h.md) · [`gfx/GfxActor.h`](gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](gfx/GfxScene.h.md) · [`gfx/IGfxWater.h`](gfx/IGfxWater.h.md) · [`gui/panels/GUI_CollisionsDebug.h`](gui/panels/GUI_CollisionsDebug.h.md) · [`gui/panels/GUI_DirectionArrow.h`](gui/panels/GUI_DirectionArrow.h.md) · [`gui/panels/GUI_MainSelector.h`](gui/panels/GUI_MainSelector.h.md) · [`gui/panels/GUI_MessageBox.h`](gui/panels/GUI_MessageBox.h.md) · [`gui/panels/GUI_VehicleInfoTPanel.h`](gui/panels/GUI_VehicleInfoTPanel.h.md) · [`physics/SimData.h`](physics/SimData.h.md) · [`physics/SlideNode.h`](physics/SlideNode.h.md) · [`physics/collision/DynamicCollisions.h`](physics/collision/DynamicCollisions.h.md) · [`physics/flex/FlexFactory.h`](physics/flex/FlexFactory.h.md) · [`physics/flex/FlexMeshWheel.h`](physics/flex/FlexMeshWheel.h.md) · [`physics/water/Wavefield.h`](physics/water/Wavefield.h.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.h`](resources/terrn2_fileformat/Terrn2FileFormat.h.md) · [`resources/tobj_fileformat/TObjFileFormat.h`](resources/tobj_fileformat/TObjFileFormat.h.md)
**Tier floor** — T4

## Purpose

Two jobs. The incidental one — letting headers mention a type without including its definition — disappears in most languages. The load-bearing one is the set of **typed ids with an explicit "invalid" sentinel** and the list of **which types are shared-ownership handles**; both shape every other chapter.

## State

Stateless.

## Typed ids

Each id is a plain integer index into one specific array; mixing them is a bug the source language cannot catch, so a rebuild should make them distinct types.

| Id | Indexes | Invalid |
|---|---|---|
| `ActorInstanceID` | unique per session, sequential | 0 |
| `ScriptUnitID` | loaded script units, sequential | −1 (−2 = "the terrain's default script") |
| `PointidID` | collision detector hit list | −1 |
| `RefelemID` | collision detector reference list | −1 |
| `CacheEntryID` | mod cache entries | −1 |
| `NodeNum` | nodes of one actor — **16-bit unsigned** | 65535 (max usable 65534) |
| `WheelID`, `PropID`, `FlexbodyID`, `FlareID`, `ExhaustID`, `CParticleID`, `CineCameraID`, `VideoCameraID` | per-actor arrays | −1 |
| `CommandkeyID` | command keys, **1-based**; 0 invalid; negative indices are legal (see [`SimData.h`](physics/SimData.h.md) `CmdKeyArray`) | 0 |
| `FreeForceID` | free forces, sequential | −1 |
| `TerrainEditorObjectID` | editable terrain objects | −1 |
| `FreeBeamGfxID` | free-beam visuals | −1 |
| `BuoyCachedNodeID` | buoyancy node cache | −1 |
| `RepoFileInstallRequestID` | repository downloads, sequential | −1 |
| `ScriptRetCode` | script engine return codes merged with RoR codes | — |

**Notes** — `NodeNum` being 16-bit caps an actor at 65,534 nodes and halves the size of every beam record (two node refs each); it is a real constraint on content, not an accident.

## Shared handles

Reference-counted (see [`RefCountingObjectPtr.h`](utils/memory/RefCountingObjectPtr.h.md)) because scripts may hold them: `ActorPtr`, `AeroEnginePtr`, `AutopilotPtr`, `CacheEntryPtr`, `DashBoardManagerPtr`, `EnginePtr`, `GenericDocumentPtr`, `GenericDocContextPtr`, `LocalStoragePtr`, `ProceduralPoint/Object/Road/ManagerPtr`, `ScrewpropPtr`, `SoundPtr`, `SoundScriptInstancePtr`, `SoundScriptTemplatePtr`, `TerrainPtr`, `TerrainEditorObjectPtr`, `TuneupDefPtr`, `VehicleAIPtr`.

Shared but host-only (ordinary shared ownership): parsed documents of the terrain formats (`ODef`, `OTC`, `Skin`, `TObj`, `Terrn2`) and the truck document `RigDef::Document`.

Everything else is owned by exactly one manager.

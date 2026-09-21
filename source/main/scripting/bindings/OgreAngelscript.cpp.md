# source/main/scripting/bindings/OgreAngelscript.cpp

> Exposes math value types and a subset of the rendering engine scene graph, resources and overlays to scripts.

**Needs** — [`Application.h`](../../Application.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [`ScriptUtils.h`](../ScriptUtils.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine) · [Seam: 3D rendering engine](../../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## Math value types (global namespace)

Plain-data values with constructors, arithmetic and comparison operators:

- `vector2` {x, y}, `vector3` {x, y, z} — length, squaredLength, distance, dot/cross product, normalise, midPoint, makeFloor/Ceil, perpendicular, randomDeviant, angleBetween, getRotationTo (3D), isZeroLength, normalisedCopy, reflect, position/direction equality tests, isNaN.
- `quaternion` {w, x, y, z} — Dot, Norm, normalise, Inverse, UnitInverse, Exp, Log, getRoll/Pitch/Yaw, equals, isNaN; global Slerp, SlerpExtraSpins, Intermediate, Squad, nlerp.
- `radian`, `degree` — full arithmetic, conversions valueRadians/valueDegrees/valueAngleUnits, implicit conversion between them.
- `color` {r, g, b, a}, `box` {left, top, right, bottom, front, back}.

## Scene and resources (namespace `Ogre`)

Non-owning references (the engine owns lifetime) unless noted as value handles:

- `Root` (singleton; list scene managers), `SceneManager` (name, movable objects by type, create/destroy entity and manual object, root scene node, destroy scene node, ambient light).
- `Node` (name, position, orientation, scale, parent, children) and `SceneNode` (attach/detach objects, child create/destroy, bounding box display, yaw, direction, lookAt, auto-tracking, visibility); `MovableObject` (name, type, parent node, attached/in-scene, visibility, shadows, rendering distance/min pixel size, bounding radius, detach) with casts to `Entity`, `ManualObject`, `Light`.
- `Entity` (material, animation states, skeleton display, LOD, sub-entities, mesh), `SubEntity`, `AnimationState` / `AnimationStateSet` (time, length, weight, enable, loop, blend masks).
- `ManualObject` (begin/update, position, normal, textureCoord, colour, index, end, counts, convertToMesh).
- Value handles: `TexturePtr`, `MeshPtr`, `MaterialPtr`, `HardwarePixelBufferPtr` (lock, blit, unlock), `PixelBox` and `Image` (get/set colour at, flip, resize, sizes), `GpuProgramParametersPtr` (set constants by index/name, list named constants), `Timer`.
- Managers: `TextureManager` (load), `MeshManager` (load, remove), `MaterialManager` (getByName, create); `SubMesh` (material, raw vertex positions/texcoords, 16/32-bit index buffers); `Technique`, `Pass` (texture units, per-stage GPU program parameters get/set), `TextureUnitState`.
- `Light` (type, colours, attenuation, position, direction, spotlight params, power scale, derived pose).
- Overlays: `OverlayManager` (create/get/destroy overlays and elements, templates, viewport size), `Overlay` (z-order, show/hide, 2D elements, scroll/rotate/scale), `OverlayElement` (position, size, material, caption, colour, metrics mode, alignment).
- Enums: IndexType, TransformSpace, RenderOperation, ImageFilter, HardwareBufferLockOptions, LightTypes, GuiMetricsMode, GuiHorizontalAlignment.

**Notes** — methods prefixed `__` return native data as script arrays and are meant to be wrapped by script-side helpers. A rebuild on a different renderer needs a shim that preserves this surface for existing scripts, or declares script compatibility broken.

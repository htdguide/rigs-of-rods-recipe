# source/main/pch.h

> Precompiled-header list: the third-party headers every translation unit pulls in.

**Needs** — [Seam: 3D rendering engine](../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) · [Seam: Windowing and input devices](../../SYSTEM-REQUIREMENTS.md#seam-windowing-and-input-devices) · [Seam: Immediate-mode GUI](../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [Seam: JSON](../../SYSTEM-REQUIREMENTS.md#seam-json) · [Seam: Positional audio](../../SYSTEM-REQUIREMENTS.md#seam-positional-audio) · [Seam: Localization catalogs](../../SYSTEM-REQUIREMENTS.md#seam-localization-catalogs) · [Seam: TCP sockets](../../SYSTEM-REQUIREMENTS.md#seam-tcp-sockets) · [Seam: Discord rich presence](../../SYSTEM-REQUIREMENTS.md#seam-discord-rich-presence) · [Seam: Script engine](../../SYSTEM-REQUIREMENTS.md#seam-script-engine) · [Seam: HTTP client](../../SYSTEM-REQUIREMENTS.md#seam-http-client) · [Seam: Sky rendering](../../SYSTEM-REQUIREMENTS.md#seam-sky-rendering) · [Seam: Vegetation paging](../../SYSTEM-REQUIREMENTS.md#seam-vegetation-paging)
**Used by** — every translation unit (precompiled header; build-time only)
**Tier floor** — T4

## Purpose

A compile-time speed-up with no behaviour. Its one useful fact for a rebuilder is that it is an honest inventory of the external libraries, and of which are optional: sockets, Discord, scripting, HTTP, the Caelum sky and paged vegetation are each guarded by a feature switch; rendering, input, GUI, JSON, audio and the translation reader are unconditional.

## State

Stateless.

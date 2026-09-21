# source/main/scripting/bindings/ImGuiAngelscript.cpp

> Exposes a subset of the immediate-mode GUI to scripts under namespace `ImGui`.

**Needs** — [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `ImGui::` functions (214)

Scripts call these inside their `frameStep`; the frame is the game's GUI frame.

- **Windows** — Begin/End, BeginChild/EndChild, BeginChildFrame/EndChildFrame, window pos/size/collapse/focus (current and next), content region queries, font scale, scroll get/set, window focus/hover tests, draw list access.
- **Layout** — Separator, SameLine, NewLine, Spacing, Dummy, Indent/Unindent, groups, cursor get/set (local and screen), text line heights, legacy Columns API, clip rects, PushID/PopID/GetID, style var/colour push/pop, next item width/open.
- **Widgets** — Text, TextDisabled, TextColored, TextWrapped, LabelText, Bullet, BulletText, Button, SmallButton, InvisibleButton, Image, Checkbox, CheckboxFlags, RadioButton, ProgressBar, Combo, Drag/Input/Slider for float×1–4 and int×1–3, ranges, InputText (single and multiline), ColorEdit/ColorPicker 3/4, ColorButton, Selectable, ListBoxHeader, Value, PlotLines.
- **Trees, tabs, menus, popups, tooltips** — TreeNode/Push/Pop, CollapsingHeader, tab bars and items, main and window menu bars, menus and items, all popup variants, tooltips.
- **Queries and input** — item hovered/active/clicked/visible and rect, frame count, time, CalcTextSize, list clipping, keyboard and mouse state, drag delta, cursor shape, capture requests, clipboard.

## `ImDrawList`

AddLine, AddTriangle(Filled), AddRect(Filled), AddCircle(Filled), AddText, AddImage.

## Enums

ImGuiStyleVar (23), ImGuiWindowFlags (24), ImGuiCol (48), ImGuiCond (Always, Once, FirstUseEver, Appearing), ImGuiTabBarFlags (11), ImGuiTabItemFlags (5) — names and values as in the GUI library.

**Notes** — the binding keeps the upstream spellings, including the misspelt `GetWindowWedth`. Engine property "allow unsafe references" exists for this binding, since many widgets take in/out references.

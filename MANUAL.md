# Manual

This guide describes the structural architecture, module layout, internal algorithms, optimization behaviors, and technical specifications of the **CherryPucker** codebase.
---
## Back to...
- ▪️[AGENTS.md](AGENTS.md)
- ▪️[AILOG.md](AILOG.md)
- ▪️[AITASKS.md](AITASKS.md)
- ▪️[BUILD.md](BUILD.md)
- ▪️[CODE.md](CODE.md)
- ▪️[FEATURES.md](FEATURES.md)
- 🔸[MANUAL.md](MANUAL.md)
- ▪️[README.md](README.md)
- ▪️[SPEC.md](SPEC.md)
- ▪️[TESTING.md](TESTING.md)

<!--
AI to use a free form format with groups here
free to add/remove/modify these sections
-->
## 🏗️ 1. Architecture Overview
HotWinAHK operates as a low-overhead orchestrator for the Windows desktop environment using an event-driven and polling-hybrid design:
- **Central Core (`HotWinAHK.ahk`)**: Running with administrator authorization, this module loads global configuration matrices, locks the script's physical handle string sequence, initiates WinEvent hooks, and fires polling ticks for mouse collisions.
- **Dynamic Key Listener Mappings (`HotWinAHK_aux.ahk`)**: Compiled on-the-fly, this file registers relative hotkey combos with the Windows kernel, routing them to action selectors based on state filters.
- **Isolator Subprocesses (`HotWinAHK_tray.ahk`)**: Minimizes background locking by spawning lightweight individual handlers whenever windows are stowed away into custom system trays.

## 🧠 2. Core Modules & Systems
The codebase is composed of highly specialized systems that collaborate without thread blocks:
- **Hotkeys Dynamic Compiler**: Parsed using `IniRead` arrays. Identifies active sections, extracts mapping keys, maps virtual modifier sequences (Ctrl, Alt, Shift, Win), verifies standard character formats, and structures `.ahk` trigger blocks with execution safety gates.
- **Velocity Bump Tracker**: An active thread loop running on a tight 25ms interval. Evaluates physical mouse velocities and boundary positions against configured edge parameters.
- **Focus Tone Feedback listener**: Uses a Microsoft Win32 event hook callback registration (`EVENT_SYSTEM_FOREGROUND`) to detect active window changes and beep audible sound frequencies using non-blocking DLL calls.
- **Dark-Themed Tooltip Engine**: Intercepts active Windows tooltips (`ahk_class tooltips_class32`) and forces dark theme styling natively using the Windows UX Theme library (`SetWindowTheme` API).

## 🔎 3. Core Algorithm
The application's crown jewel is its mathematical screen-edge docking and mouse-bumping algorithm:
- **Euclidean Vector Speed Calculation**: On every 25ms cycle, it measures the vector travel distance of the mouse:
  $$\text{pixelDistance} = \sqrt{\Delta X^2 + \Delta Y^2}$$
  If the cursor enters an edge zone (`g_BumpEdgeZonePixels`) and the speed matches of exceeds `g_BumpVelocityThreshold` in the towards-screen boundary direction, an Edge Bump occurs.
- **Adaptive Snapping Alignment**: Aspect tile positions are derived using mathematical rounding maps:
  $$\text{cellIndex} = \text{Round}\left(\frac{\text{Coordinate} - \text{Padding}}{\text{TileDimension} + \text{Margin}}\right)$$
  This aligns window corners cleanly to standard columns and rows without overlapping borders.
- **Quadratic Easing Motion Animation**: Smoothes window snaps using quadratic ease-out interpolation over a 150ms segment containing 12 linear intermediate frames:
  $$\text{easeProgress} = t \times (2 - t)$$
- **Hardened Dual-Anchor Protection & Countdown Latch**: When a stowed window is peek-revealed, a 50ms polling thread (`TrackUntuckedFocusLifecycle`) checks both focus state and mouse positioning. To prevent erratic retucking during transition phases, the system activates a countdown latch of `g_UntuckGraceTicks := 10` (500ms). The window is locked open while:
  1. The cursor is placed within the window boundary or its active native child controls (retrieved recursively via Win32 `GetAncestor` calls).
  2. The window remains the active Windows foreground window element.
  Definitive state switches (moving cursor away AND activating a separate typing window) trigger clean, interpolated sliding back to edge margins.
- **Damped Drag Resistance Curve**: Left-click dragging a peek-untucked window away from its docked screen margin is restricted using non-linear resistance. Standard mouse coordinates map to window coordinates using a steep 4x damping reduction multiplier:
  $$\text{resistedDeltaX} = \Delta X \times 0.25$$
  Parallel dragging coordinates are restricted with a 2x damping parameter ($0.5$).
- **Hysteresis Pop-off Threshold**: If the resisted drag direction displacement exceeds 120 absolute screen pixels ($\text{pullDistance} > 120\text{px}$), the docking structural anchor snaps. The window is removed from the stowed registry maps, plays an audio pitch, and is restored as a free-floating window.
- **Ctrl-Hold Dock-Seeking Indicator Overlay**: Holding the `Ctrl` key during a stowed window drag disables physical resistance and enters a Dock-Seeking Mode. The engine calculates the closest screen margin of the mouse's active monitor layout, instantiates a click-through translucent cyan GUI indicator overlay bar (60px thickness) aligned to that edge, and immediately snaps and docks the stowed window to the selected monitor margin upon left-clicked mouse release.

## 🛰️ 4. Commands, Keybindings & Context Flags
Every action from simple moves to grid mapping is indexed inside the INI command table:
- **Modifier Bindings**: Standardized representation allows compound keys to be written easily (e.g. `Win+Ctrl+Shift+Left`).
- **Contextual Execution Filters**: Hotkey combinations can be targeted to run conditionally using key filters:
  - `wintitleis=text` / `wintitlehas=text` (checks window title matching)
  - `winclassis=text` / `winclasshas=text` (targeted to matching application classes)
  - `winexeis=text` (direct executable restriction filters)

## 🔧 5. Workspace Build & Configuration
- **Script Customization**: Changes to standard profiles are added to the `HotWinAHK.ini` table.
- **On-The-Fly Compilation**: Hotkeys automatically re-compile and reboot on save or whenever `ReloadConfig` triggers.
- **Distribution Compile Step**: Source directories can be bundled into standard Windows binaries using compiler commands (`Ahk2Exe.exe /in HotWinAHK.ahk /out HotWinAHK.exe`).


---
## Go Back to...
- [AGENTS.md](AGENTS.md)
- [AILOG.md](AILOG.md)
- [AITASKS.md](AITASKS.md)
- [BUILD.md](BUILD.md)
- [CODE.md](CODE.md)
- [FEATURES.md](FEATURES.md)
- [MANUAL.md](MANUAL.md)
- [README.md](README.md)
- [SPEC.md](SPEC.md)
- [TESTING.md](TESTING.md)

---
## Go back to...
- ▪️[AGENTS.md](AGENTS.md)
- ▪️[AILOG.md](AILOG.md)
- ▪️[AITASKS.md](AITASKS.md)
- ▪️[BUILD.md](BUILD.md)
- ▪️[CODE.md](CODE.md)
- ▪️[FEATURES.md](FEATURES.md)
- 🔸[MANUAL.md](MANUAL.md)
- ▪️[README.md](README.md)
- ▪️[SPEC.md](SPEC.md)
- ▪️[TESTING.md](TESTING.md)

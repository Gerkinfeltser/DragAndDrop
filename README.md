# Drag And Drop

A Skyrim SE SKSE plugin for grabbing, dragging, and throwing NPCs using Havok physics springs.

**[Nexus Mods Page](https://www.nexusmods.com/skyrimspecialedition/mods/178446)** — download releases, screenshots, and discussion.

## Features

- **Grab** NPCs with a configurable action key (default: G)
- **Drag** them around with physics-based spring attachment
- **Throw** them with ramping force by holding the action key
- **Swing impact** — knock nearby actors while dragging
- **Impact tracking** — thrown NPCs knock back actors they pass near
- **Drop momentum** — NPCs inherit camera swing velocity on drop
- Highly configurable via INI

## Requirements

- Skyrim Special Edition / Anniversary Edition
- [SKSE64](https://skse.skyrimracing.com/)
- [Address Library for SKSE Plugins](https://www.nexusmods.com/skyrimspecialedition/mods/32444)

## Building

### Prerequisites

- Visual Studio 2022 with C++ Clang tools
- [vcpkg](https://vcpkg.io/) with `x64-windows-static` triplet
- [CMake](https://cmake.org/) 3.21+

### Steps

```bash
cd SKSE
cmake -B build -S . -DCMAKE_TOOLCHAIN_FILE=[vcpkg root]/scripts/buildsystems/vcpkg.cmake
cmake --build build --config Release
```

The built DLL lands in `SKSE/Plugins/DragAndDrop.dll`.

## Installation

Copy the following into your Skyrim Data folder:

```
SKSE/Plugins/DragAndDrop.dll
SKSE/Plugins/DragAndDrop.ini
Scripts/DragDrop.pex
Scripts/DragDropImpactScript.pex
DragAndDrop.esp
```

Or use a mod manager (MO2/Vortex).

## Configuration

All settings are in `SKSE/Plugins/DragAndDrop.ini`. See the file for full documentation of each setting.

### Key Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `iActionKey` | 34 (G) | Scancode for grab/drop/throw |
| `fGrabRange` | 120.0 | Max grab distance |
| `fThrowImpulseMax` | 20.0 | Max throw force |
| `fImpactRadius` | 120 | Knockback detection radius |
| `bGrabAnyone` | false | Allow grabbing any NPC |

Full scancode reference: [Nexus Mods Article](https://www.nexusmods.com/skyrimspecialedition/articles/7704)

## Papyrus API

```papyrus
DragDrop.ReleaseNPC()              ; Drop NPC (no impulse)
DragDrop.ThrowNPC(float force)     ; Throw NPC with force
DragDrop.GetGrabbedNPC()           ; Returns current grabbed Actor
DragDrop.IsDragging()              ; Returns bool
```

## Credits

- [powerof3's GrabAndThrow](https://www.nexusmods.com/skyrimspecialedition/mods/120460) — Havok spring access and throw impulse patterns
- [CommonLibSSE-NG](https://github.com/CharmedBaryon/CommonLibSSE-NG) — SKSE plugin library

## License

MIT

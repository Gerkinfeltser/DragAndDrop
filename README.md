# Drag & Drop

A Skyrim SE mod that lets you grab, drag, and throw NPCs using Havok physics. Self-contained — no other mods required (but pairs well with [Fableforge](https://www.nexusmods.com/profile/FableForge)'s [Knockout and Surrender](https://www.nexusmods.com/skyrimspecialedition/mods/40556) and [Knock and Surrender or Execute Patch](https://www.nexusmods.com/skyrimspecialedition/mods/173676) by me).

**Source code:** [GitHub](https://github.com/Gerkinfeltser/DragAndDrop)

## Requirements

- Skyrim Special Edition with matching SKSE and Address Library. Core gameplay of the `0.1.99-alpha` migration candidate was maintainer-tested on `1.6.1170`; queued-transition safety and other runtimes remain unverified. See [per-runtime evidence](docs/MINLL-MIGRATION.md#runtime-acceptance-matrix).
- SKSE64
- Address Library for SKSE Plugins

## Install

Drop the contents of the zip into your `Data` folder, or install via MO2/Vortex. That's it.

## Uninstall

Remove whenever you like. The mod is save-game safe — no quests, aliases, or permanent cell edits. The grab spell stays in your spell list after removal but does nothing. If removed mid-drag, your speed may not restore (fix with `player.setav speedmult 100` in console).

## How to Use

The mod adds a Lesser Power to your character. It works automatically — no need to equip or activate anything from the power menu.

### Controls (G key by default)

**To grab:**
- Look at an NPC and hold **G** — the grab starts on keydown

**To drop:**
- While dragging, **tap G** — the NPC drops with momentum from your camera swing

**To throw:**
- While dragging, **hold G** — a throw charges up (you'll see a notification)
- **Release G** — the NPC launches with ramping force. Longer hold = bigger throw

**Grab-hold-drop:**
- Hold G on an NPC to grab
- Release G quickly (within the hold timeout) — NPC stays grabbed
- Release after the timeout — charged release with shipped `bChargeThrowOnHold=true`; set it false for hold-drop.

### What Can You Grab?

By default you can grab:
- **Dead NPCs** — always
- **Paralyzed NPCs** — always
- **Followers** — enabled by default
- **Hostile NPCs** — disabled by default, see INI

You cannot grab ghosts, children (by default), or NPCs with paralysis immunity keywords.

### Swing Impact

While dragging, the NPC acts as a battering ram. Swing them into nearby actors and they'll get knocked back. Clutter and dynamic objects also get pushed around.

### Throw Impact

After throwing, the NPC is tracked for a few seconds. Any actors it passes near get knocked back. Damage can also be configured in the INI.

## INI Settings

All settings are in `SKSE/Plugins/DragAndDrop.ini`. Edit with any text editor. Changes require a game restart.

**Important:** Do not add inline comments with semicolons — they break the parser. Keep values clean:

```ini
bEnableMod = true
```

See [INI settings](docs/INI-SETTINGS.md) for current sections, shipped values, missing-key fallbacks, and descriptions. These values differ; deleting the INI does not reproduce the shipped configuration.

## Build and package

Use a C++23-capable MSVC developer shell with CMake, Ninja, and vcpkg. Configure a fresh repository-local tree; ordinary builds never install the DLL:

```powershell
cmake -S SKSE -B SKSE/build-minll/release -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE="$env:VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake" -DVCPKG_TARGET_TRIPLET=x64-windows-static -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
cmake --build SKSE/build-minll/release --parallel 6
powershell -NoProfile -NonInteractive -File make_zip.ps1 -Dll SKSE/build-minll/release/DragAndDrop.dll
```

The public source repository contains build inputs, not the development workspace's packaging scripts or ESP/SEQ/compiled assets. Run the first two commands there; run packaging and deployment commands only in the development workspace with the matching assets.

CMake source-builds the pinned MinLL SDK with SE+AE enabled and VR disabled, verifying revision/license and the shared non-VR ABI overlay. Keep the plugin and vcpkg compiler/CRT profiles coherent. Version and candidate suffix are owned by `SKSE/CMakeLists.txt`; change them there, not through a label cache override.

Packaging requires an explicit DLL, prints its SHA-256, verifies the archived member against it, and includes the SDK notice. Compare the printed hash to the accepted build hash. Packaging is not installation or runtime acceptance.
Candidate archives are stored and committed in the private repository as `_releases/DragAndDrop_v0-1-99-alpha.zip`; only archive filenames replace version dots with hyphens. Native metadata and logs keep `0.1.99` / `0.1.99-alpha`.

Only after installation approval, back up the current DLL/INI and save/co-save checkpoint, then explicitly deploy:

```powershell
Copy-Item SKSE/build-minll/release/DragAndDrop.dll SKSE/Plugins/DragAndDrop.dll -Force
```

`SKSE/Plugins` is MO2-linked in the development workspace. Installation, game launch, `publish_source.ps1` (copies and commits public source), commits, and pushes require separate approval. See [migration evidence and tester checklist](docs/MINLL-MIGRATION.md) for ABI audit, prerequisites, and exact-runtime acceptance.


## Known Issues

- **Power menu bypasses filters.** Casting the grab spell from the power menu ignores target restrictions (dev/debug mode).

## Early Access

This mod is in early development. Back up your saves. While the mod is designed to be save-safe and shouldn't cause issues, removing it mid-drag could leave your speed altered (see Uninstall above).

## Credits

- [GrabAndThrow](https://www.nexusmods.com/skyrimspecialedition/mods/120460) by powerof3 — Havok spring access and throw impulse patterns
- [Seize NPCs](https://www.nexusmods.com/skyrimspecialedition/mods/135703) — Mod inspiration & reference

## License

Drag & Drop is MIT-licensed, copyright 2026 Gerkinfeltser. The canonical `LICENSE` is in the public-source checkout (`_releases/DragAndDrop/LICENSE` in the development workspace); packaging includes it at the archive root. Dependency copyrights and terms are retained separately in the CommonLibSSE license and third-party notices.

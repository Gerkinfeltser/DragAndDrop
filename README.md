# Drag & Drop

A Skyrim SE mod that lets you grab, drag, and throw NPCs using Havok physics. No external gameplay-mod dependencies. Pairs well with [Knockout and Surrender](https://www.nexusmods.com/skyrimspecialedition/mods/40556) and [Knock and Surrender or Execute Patch](https://www.nexusmods.com/skyrimspecialedition/mods/173676).

**Source code:** [GitHub](https://github.com/Gerkinfeltser/DragAndDrop/tree/compat/skyrim-1.7.x)

## Requirements

- Skyrim Special Edition with matching SKSE64 and Address Library.
- Core gameplay of `0.1.99-alpha` was maintainer-tested on `1.6.1170`; queued-transition safety and full existing-save reload acceptance remain unverified.
- Other runtimes, including `1.7.99` and `1.7.104`, still need tester reports. This is not a verified 1.7 compatibility release. See [per-runtime evidence](https://github.com/Gerkinfeltser/DragAndDrop/blob/compat/skyrim-1.7.x/docs/MINLL-MIGRATION.md#runtime-acceptance-matrix).

## Install

Install the ZIP with MO2/Vortex, or copy its contents into Skyrim's `Data` folder. Launch through SKSE. Close Skyrim before replacing mod files, and keep your existing INI settings when upgrading.

## Uninstall

The mod adds no permanent quests, aliases or cell edits. The grab spell may remain in your spell list but does nothing after removal. If removed mid-drag, your speed may not restore; fix with `player.setav speedmult 100` in the console. Back up your saves before changing mods.

## How to Use

The mod adds a Lesser Power. With action-key grab enabled, you do not need to equip or cast it through the power menu.

### Controls (G by default)

- **Grab:** look at an eligible NPC and hold G; grab starts on keydown.
- **Drop:** while dragging, tap G; the NPC keeps momentum from your camera swing.
- **Throw:** while dragging, hold G to charge, then release. Longer holds increase force up to the configured limit. HUD notifications are configurable.
- **Tap-grab:** release the initial grab before `fGrabHoldTimeout` and the NPC stays grabbed.
- **Initial hold-release:** release after that timeout for a charged release with shipped `bChargeThrowOnHold=true`; set it false for hold-drop.

### Eligible targets

Dead and paralyzed NPCs are eligible. Followers are enabled by default; hostile NPCs are disabled by default. Settings can change those restrictions. Ghosts, children by default, and actors with paralysis-immunity keywords are excluded.

### Impacts

Swinging a dragged NPC can knock back nearby actors and push dynamic clutter. After a throw, the NPC is tracked briefly for further knockback. Impact damage can be configured in the INI.

## INI Settings

Edit `SKSE/Plugins/DragAndDrop.ini` inside this mod. Restart Skyrim after changing settings.

**Do not add inline semicolon comments:** keep values clean, especially booleans.

```ini
bEnableMod = true
```

The supplied INI and missing-key fallbacks differ; deleting the INI does not reproduce the shipped configuration. Detailed sections, values and behavior are in [INI settings](https://github.com/Gerkinfeltser/DragAndDrop/blob/compat/skyrim-1.7.x/docs/INI-SETTINGS.md); release packages also include `docs/INI-SETTINGS.md`.

## Known Issues

- Casting the grab power through the power menu bypasses target filters (dev/debug behavior).
- Reloaded knocked-out NPCs may be stiff or frozen; freshly knocked-out NPCs can drag normally. This pre-existing issue remains unresolved.

## Early Access

This mod is in early development. Back up your saves and report the exact game version, effective INI, observed behavior and `DragAndDrop.log` when reporting a bug. Build success alone does not establish runtime compatibility.

## Credits

- [GrabAndThrow](https://www.nexusmods.com/skyrimspecialedition/mods/120460) by powerof3 — Havok spring access and throw impulse patterns.
- [Seize NPCs](https://www.nexusmods.com/skyrimspecialedition/mods/135703) — inspiration and reference.

## License

MIT-licensed, copyright 2026 Gerkinfeltser. Release packages and the public source repository include `LICENSE`. Dependency copyrights and terms are retained separately in the CommonLibSSE license and third-party notices.

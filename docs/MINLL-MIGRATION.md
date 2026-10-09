# MinLL migration evidence — 0.1.99-alpha

## Scope and evidence boundaries

Candidate migration to MinLL/CommonLibVR `550cc4fb9114649dcf526d1f3d73d710c5d7003b`, SE+AE enabled, VR disabled. Numeric native version: `0.1.99`; package label: `v0.1.99-alpha`. No deployment, game launch, commit, or publication is part of build verification. Source and offline checks do not certify game behavior.

Current private-workspace path context: installable assets now live under `DragAndDrop/` (DLL/INI, ESP, SEQ, compiled and source scripts). Paths in the frozen/pre-migration observations below retain their original provenance; installed Data-relative paths and public-source paths are unchanged. Native source/build paths remain rooted at `SKSE/`.


## Frozen consumer contracts

- `Source/Scripts/DragDrop.psc` and `SKSE/src/main.cpp`: global native `bool ReleaseNPC()`, `bool ThrowNPC(float force)`, `Actor GetGrabbedNPC()`, `bool IsDragging()`, registered on `DragDrop`. No binding or serialization change.
- INI resolved beside the loaded DLL using its module path, read at SKSE post-load. Shipped values and missing-key fallbacks are separate contracts; see [INI settings](INI-SETTINGS.md).
- Native form lookups remain `DragAndDrop.esp:000800` (grab) and `000808` (impact). ESP, SEQ, Papyrus source and compiled scripts remain unchanged.
- Input sink walks the existing button event chain and forwards numeric `idCode`/`userEvent` down/up without device filtering. `Shout` is the exact release token. No speculative 1.7 input model added.
- Initial action-key release below `fGrabHoldTimeout` leaves a tap-grab active. At/above timeout, `bChargeThrowOnHold` selects charged release versus drop. Later action-key/Shout holds use the existing drop window and linear capped throw ramp. Preserve all timing/forces.
- Filtering rejects player/null, configured drawn-weapon classes, ghosts, children unless enabled, and immunity keywords before `bGrabAnyone`; otherwise dead/paralyzed actors pass, then configured teammates/hostiles. Range is checked by the G-key spell path. Power-menu behavior and reloaded-KO stiffness are pre-existing issues, not migration fixes.

### Frozen artifact hashes (SHA-256 before migration build)

| File | SHA-256 |
|---|---|
| Canonical `SKSE/Plugins/DragAndDrop.dll` | `A3B3F77F587D2EE7CB026789FF1129D157C4599146515926352E5EDCA3E7401E` |
| `SKSE/Plugins/DragAndDrop.ini` | `D65401DA957EB50FCE412A6D02697C43D0ABB28B01B9E875E58A36E86306F876` |
| `DragAndDrop.esp` | `AD2AEC9586AD1F79593A75418B7E43E8381BC43AB708D3DC901FE7058BC5960F` |
| `scripts/DragDrop.pex` | `560680C83E9433D9E2A1844FB0BCB4A132964E94728CD7B128557A850A224422` |
| `scripts/DragDropImpactScript.pex` | `87FD7D499ED4C18584FFC72F7A22C7A4569A6A45C193D180F13B49F768D40D25` |
| `Source/Scripts/DragDrop.psc` | `681BF5BC8ABA1886B6CFB5D439DD94044210CC919DAC4CD7C8DFD9CD53FCE734` |
| `Source/Scripts/DragDropImpactScript.psc` | `E9D63DDFBB77FCE80B364708C469A6888FB3D89D5890C8282626C25FDEBB0475` |
| `SEQ/DragAndDrop.seq` | `F2B8574C4989EACED6178DD1889D91AC7F759F921C48705B55CD7696D5D1E9CF` |

## Bounded engine-access audit

Locations refer to function names because edits shift line numbers. Reviewed against the pinned SDK, not inferred from loader flags.

| Consumer / source | Disposition and rationale |
|---|---|
| `DragHandler::IsValidTarget`, swing/throw impact `IsDead()` calls | Required shared header correction. Sole called virtual after `Unk_8C`; corrected slot `0x99`, byte offset `0x4C8`. No called Actor-declared virtual in the shifted range. `IsChild` is earlier slot `0x5E`. |
| `GetSpringAction` / `GetSpringEntity`, spring-settings writes in `HandleNewGrab` | Replaced raw `+0x30/+0x60/+0x64/+0x68` with pinned `hkpUnaryAction::entity` and `hkpMouseSpringAction` members. Keep the entity type rather than an unchecked rigid-body downcast. Typed headers alone do not prove live spring/body identity. |
| Player grab spring/object/weight | Pinned runtime aggregate nests these fields under `GetPlayerRuntimeData().grabData`; actual compile diagnosed the former flat accesses. Retain SDK runtime-aware accessor; no plugin-owned fixed player offset. |
| `CollectAllRigidBodies`, swing-static traversal | Scenegraph collision and Havok bridge casts require actual collision/body identity. Immediate paths remain synchronous; deferred paths re-collect under actor-cell world lock. |
| Throw, hit-drop, ordinary drop, Papyrus release tasks | Replace captured body pointers with actor handles; resolve actor/3D/cell/world at execution. Hit-drop cleanup remains independent from optional physics and checks ownership. |
| `TryGrabWithSpell`, impact tracking lookups | Null-check forms before actor cast. Deferred spell setup has no body writes. |
| `Hooks::InputEventSink` | Preserve chain/type/down/up/idCode/userEvent contract; pinned non-VR declarations expose existing fields. 1.7 event production remains runtime-unverified. |
| Immediate cell/world locks and Havok motion/mass fields | Typed pinned SDK layouts, no plugin-owned byte offsets; preserve current synchronous force/clamping paths. Runtime locking/object behavior still requires game acceptance. |
| Camera root transform / collision transforms | Typed SDK members; no evidenced migration-specific layout correction. No unrelated nullability refactor. |
| Actor ghost/teammate/hostility/ragdoll methods | Ordinary non-virtual pinned APIs; no shifted Actor slot consumer. |
| `main.cpp`, event-sink overrides, SDK lookups/singletons | Preserve SKSE registration/lifecycle; no plugin-owned relocation IDs or instruction patches. `GetParentCell()` is not `GetSaveParentCell()`; no saved-parent repair claimed. |
| Sound / notifications | `GetSoundHandle` accepts the descriptor interface and unchanged `0x1A` flags; failure skips playback. Old sound IDs `66404/67666` and notification IDs `52050/52933` match pinned replacements. `SendHUDMessage::ShowHUDMessage` has the same text/sound/queue defaults as old `DebugNotification`; game observations remain unexecuted. |
| Stamina / impact damage | Pinned `ModActorValue(kDamage, AV, negativeDelta)` preserves the explicit signed modifier/delta. Two-argument `RestoreActorValue` takes the absolute magnitude and would reverse damage intent. |
| Grab-effect cleanup | Fork's archetype helper is VR-only. `DispelGrabEffects` collects matching base archetypes from the non-VR active-effect list before forced `ActiveEffect::Dispel(true)`, preserving selection/force and avoiding iterator invalidation. |
| Spatial / actor iteration | Pinned callbacks accept pointers; null-check then bind references without copying. Filtering, radius checks, impulses, and damage calculations retained. |

Final audited declaration: Independent + Address Library, no active runtime whitelist, minimum SKSE unset. After typed spring conversion, no plugin-owned raw engine offset without an SDK accessor was identified. Linked metadata matches independence `0x1`, extended `0x3`, minimum SKSE `0`.

## Configuration evidence

- Pinned effective checkout and MIT license verified at configure; source target non-imported. Generated LF-normalized overlay fingerprints match the approved original `98a8a1cb24f51b893b4343c4c552e363d9bc90ee7a611c05f7719c2479bec2b6` and corrected `6ff59cd37693e301cc3282099a11b785f4a386334f0947adc37511afeabf690f`.
- SDK `cmake/CommonLibSSE.cmake` has no deployment environment-variable handling or post-build deployment. Repository post-build copy removed. SDK prebuilt resolver exits for the static CRT/profile; prebuilt selection is explicitly off.
- First configure exposed mixed compiler selection (VS2022 plugin versus VS2026 vcpkg). No native build accepted from that tree. Fresh candidate configuration uses `SKSE/build-minll/release`, VS2026 Build Tools 18.10.12224.181, MSVC 19.51.36260.0 / toolset 14.51.36231, Windows SDK 10.0.26100.0, CMake 4.2.0-rc2, Ninja 1.13.0.git.kitware.jobserver-pipe-1 from the developer environment.
- vcpkg toolchain: VS2026 `VC/vcpkg/scripts/buildsystems/vcpkg.cmake`; registry baseline `ee12231b20c95013c6638d845d04c91559a1d1ff`, target `x64-windows-static`, host `x64-windows`, static CRT. Candidate configuration rebuilt all seven ports (zero binary-cache restores): directxtk 2025-10-27, directxmath 2025-04-03, rapidcsv 8.90, spdlog 1.16.0, fmt 12.1.0, vcpkg-cmake 2024-04-23, vcpkg-cmake-config 2024-05-23.
- Fixed `/std:c++23preview` detected; optimization retained. VS2026 is this candidate's observed toolchain, not a universal migration prerequisite.
- Ordinary native compile succeeded after the demonstrated SDK API substitutions above. Removed obsolete manually quoted PCH `/FI`; CMake's generated forced-PCH path remains active. Metadata generation's `sv` suffix uses the standard string-view literal namespace imported by the plugin PCH.

Accepted developer-shell commands (VS2026 `vcvars64.bat` initialized first):

```powershell
cmake -S SKSE -B SKSE/build-minll/release -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE="C:/Program Files (x86)/Microsoft Visual Studio/18/BuildTools/VC/vcpkg/scripts/buildsystems/vcpkg.cmake" -DVCPKG_TARGET_TRIPLET=x64-windows-static -DVCPKG_HOST_TRIPLET=x64-windows -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -DFETCHCONTENT_SOURCE_DIR_COMMONLIBSSE="D:/gerkgit/Skyrim_Drag-n-Drop/SKSE/build-minll/_deps/commonlibsse-src"
cmake --build SKSE/build-minll/release --parallel 6
```

The source override is the verified clean pin, not a different SDK. Initial configuration also passed `VCPKG_VISUAL_STUDIO_PATH`, which CMake reported unused; actual compiler selection, not that option, establishes coherence.

## Offline artifact verification

- Built DLL: `SKSE/build-minll/release/DragAndDrop.dll`; SHA-256 `000FE7C1E15062C9A08FDC014F23885C9D2BA0434C4349DCFAE6132177380933`.
- All 492 compile commands: SE/AE defined, VR undefined, fixed `/std:c++23preview`, static `-MT`, shared ABI overlay first; SDK and plugin PCHs included.
- Linked `SKSEPlugin_Version`: one non-forwarded export, schema `1`, numeric version `0.1.99.0`, independence `0x1`, extended capability `0x3`, minimum SKSE `0`. No active whitelist with Independent declaration. Exports also include `SKSEPlugin_Load` and `SKSEPlugin_Query`. This is file inspection, not DLL loading.
- `dumpbin /disasm`: six native `IsDead` calls at `+0x4C8` in the native object and linked DLL; no `+0x4D0` calls. No additional shifted Actor-declared consumers were identified.
- Temporarily changed expected original-header fingerprint: configure rejected with `Header changed; re-audit the non-VR ABI correction`. Restored approved fingerprint; configure/generated profile succeeded.
- Independent native review identified and resolved missing grab-time/speed initialization, empty-engine-handle hit-drop cleanup, and null-entity spring search regressions. Final source review has no outstanding findings; it does not establish live physics behavior.

## Package verification

- Executed `powershell -NoProfile -NonInteractive -File make_zip.ps1 -Dll SKSE/build-minll/release/DragAndDrop.dll`. Input and archived DLL hashes both match the accepted hash above.
- Current archive: `_releases/DragAndDrop_v0-1-99-alpha.zip`, following the private repo's hyphenated release naming. It contains 14 files: candidate DLL, shipped INI, ESP/SEQ, two compiled scripts, two source scripts, README, INI/migration docs, project MIT `LICENSE`, CommonLib MIT notice, and combined third-party notices. No build tree, debug symbols, logs, or private state. Asset/license/notice bytes and unchanged candidate DLL identity are checked on rebuild.
- Notices retain pinned CommonLib MIT plus installed DirectXTK/DirectXMath/spdlog/fmt MIT and rapidcsv BSD-3-Clause text. Dependency versions are listed in `SKSE/THIRD-PARTY-NOTICES.txt`.
- Canonical project license: `_releases/DragAndDrop/LICENSE`; there is no private-root duplicate. Packaging copies it into the archive root; source publishing preserves the public checkout's existing license and copies dependency notices. Release ZIPs belong to the private repo; source publication/commit/push remains separate.
- Omitted DLL argument, nonexistent file, and directory input each fail. A throwaway fault-injection command corrupted the actual staged ZIP's DLL member; hash validation rejected it. All failure cases preserved the previous verified archive. Only a verified temporary archive replaces the destination.
- During build/package verification, canonical MO2-linked DLL/INI hashes remained the frozen pre-migration hashes. ESP/SEQ/Papyrus assets were unchanged. Source publisher was not executed; no installation or game launch was performed by the build verification. Subsequent maintainer installation/testing is recorded below.



## Runtime acceptance matrix

| Game version | Status | SKSE / Address Library | Installed DLL SHA-256 | Registration / input / gameplay / reload |
|---|---|---|---|---|
| 1.6.1170.0 | maintainer-tested, partial | Exact SKSE / Address Library versions not recorded | `000FE7C1E15062C9A08FDC014F23885C9D2BA0434C4349DCFAE6132177380933` | Core gameplay reported working; queued-transition and full existing-save reload acceptance untested |
| 1.5.97.0 | unexecuted | Not reported | Not reported | Community load report needed |
| 1.6.640.0 | unexecuted | Not reported | Not reported | Community load report needed |
| 1.7.99.0 | unexecuted | Not reported | Not reported | Legacy action-key and Shout delivery unknown |
| 1.7.104.0 | unexecuted | Not reported | Not reported | Legacy action-key and Shout delivery unknown |

Load reports are `reported-load`, never gameplay passes. Missing action-key logging requires diagnosis of installation, settings, state, key matching, and delivery; it does not by itself prove missing legacy events. Every verified support claim requires identified installed bytes and full feature/lifecycle acceptance on that exact runtime. No blanket future 1.7.x claim.

### Maintainer test evidence — 2026-10-08

- Separate `C:/Users/vector/Documents/My Games/Skyrim Special Edition/SKSE/DragAndDrop.log` identifies `0.1.99-alpha` on `1.6.1170.0`, successful input-sink installation and Papyrus registration, out-of-range refusal, grabs, a 0.15-second tap-drop, and a 2.10-second charged throw with deferred impulse on 20 bodies.
- Current MO2-linked DLL at `D:/Modlists/ADT/mods/Skyrim_Drag-n-Drop_symlink/SKSE/Plugins/DragAndDrop.dll` was hashed and matches the accepted candidate above. That check identifies current installed bytes, not an independent timestamped hash from the running session.
- Maintainer separately reports hold-drop, Shout release, impact knockback, hit-drop, and speed restoration working, and accepts the candidate for current use. Shout was disabled in the supplied log; its successful test is a maintainer report, not evidence from that log.
- Queued drop/throw/hit-drop across load doors or fast travel, missing-physics/stale-task edge cases, and full existing-save reload acceptance remain untested. No blanket runtime certification or 1.7 support claim.
- Supplied `SkyrimNetOutput_ADT_2026-10-08_22-53-08-732_11771f05.zip` contains SkyrimNet/Papyrus logs, not `DragAndDrop.log`; it is not a second independent copy of the plugin test evidence. Backup/checkpoint details and exact SKSE/Address Library versions were not recorded.


## Community tester checklist

1. Install the candidate package with matching SKSE and Address Library for the exact game version; record installed DLL SHA-256.
2. Enable `bEnableLogging` and record effective `bEnableMod`, `bEnableGKeyGrab`, `iActionKey`, and `bUseShoutKeyForRelease`. Enable the mod and G-key grab for the action-key check.
3. Start with no active drag, look at an NPC, then press and release the configured action key (G only for `iActionKey = 34`). Record down/up behavior; a down log alone does not verify key-up. Test Shout release separately with its option enabled.
4. Send the active instance's `DragAndDrop.log`, effective INI, exact game/SKSE/Address Library versions, installed DLL hash, initial state, and observed result. Installation evidence is `Input event sink installed`, not the unconditional `Hooks installed`; include `Game version: …` and `Papyrus functions registered`. `G-key down: calling TryGrabWithSpell` records the eligible down path; absence requires diagnosis rather than proving missing legacy delivery.

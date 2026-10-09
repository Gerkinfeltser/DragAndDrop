# Drag & Drop — development workspace

Private development repository for the Skyrim SKSE mod. User documentation lives in `DragAndDrop/README-DragAndDrop.md`, beside the installable contents.

The [public source repository](https://github.com/Gerkinfeltser/DragAndDrop/tree/compat/skyrim-1.7.x) has a different layout: the publisher maps the user guide to `README.md` and this development guide to `README-DEVELOPMENT.md`. The private mod-folder prefix is not used for public INI/Papyrus sources.

## Build and package

Use a C++23-capable MSVC developer shell with CMake, Ninja, and vcpkg. Configure a fresh repository-local tree; ordinary builds never install the DLL:

```powershell
cmake -S SKSE -B SKSE/build-minll/release -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE="$env:VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake" -DVCPKG_TARGET_TRIPLET=x64-windows-static -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
cmake --build SKSE/build-minll/release --parallel 6
powershell -NoProfile -NonInteractive -File make_zip.ps1 -Dll SKSE/build-minll/release/DragAndDrop.dll
```

The public source repository contains build inputs, not the development workspace's packaging scripts or ESP/SEQ/compiled assets. Run the first two commands there; run packaging and deployment commands only in the development workspace with the matching assets.

In the public checkout, the user guide is the root `README.md`, not the private mod-folder path above. Use the documented native build commands from the repository root; private packaging/deployment and modlist paths below do not apply to that checkout.


### Private workspace layout

| Path | Purpose |
|---|---|
| `DragAndDrop/` | Installable Data-relative mod contents; link this folder into MO2 |
| `SKSE/` | Native source, CMake/dependencies and isolated build trees |
| `DragAndDrop_spriggit/` | Editable ESP source |
| `_releases/DragAndDrop/` | Public-source submodule; its source layout is unchanged |
| Root scripts/docs and `_releases/*.zip` | Development tooling, documentation and candidate packages |

The existing ADT and ASSOS-1.1.1 `Skyrim_Drag-n-Drop_symlink` entries point to the private `DragAndDrop/` folder, not the repository root. Editing files there updates what MO2 sees directly. Close Skyrim before replacing runtime files or changing links; preserve mod entry names to retain profile selection/order.

ESP rebuild output is `DragAndDrop/DragAndDrop.esp`. Pass `DragAndDrop/Source/Scripts/<name>.psc` to the existing portable Papyrus compiler; its output directory derives from the parent of `Source`, so compiled scripts go to `DragAndDrop/scripts/`. These are explicit asset writes, not part of native build/package verification.


CMake source-builds the pinned MinLL SDK with SE+AE enabled and VR disabled, verifying revision/license and the shared non-VR ABI overlay. Keep the plugin and vcpkg compiler/CRT profiles coherent. Version and candidate suffix are owned by `SKSE/CMakeLists.txt`; change them there, not through a label cache override.

Packaging requires an explicit DLL, prints its SHA-256, verifies the archived member against it, and includes the SDK notice. Compare the printed hash to the accepted build hash. Packaging is not installation or runtime acceptance.
Candidate archives are stored and committed in the private repository as `_releases/DragAndDrop_v0-1-99-alpha.zip`; only archive filenames replace version dots with hyphens. Native metadata and logs keep `0.1.99` / `0.1.99-alpha`.

Only after installation approval, back up the current DLL/INI and save/co-save checkpoint, then explicitly deploy:

```powershell
Copy-Item SKSE/build-minll/release/DragAndDrop.dll DragAndDrop/SKSE/Plugins/DragAndDrop.dll -Force
```

`DragAndDrop/` is MO2-linked in the development workspace. Installation, game launch, `publish_source.ps1` (copies and commits public source), commits, and pushes require separate approval. The publisher reads private INI/PSC files from the mod root but retains the public repository's flat source layout. See [migration evidence and tester checklist](docs/MINLL-MIGRATION.md) for ABI audit, prerequisites, and exact-runtime acceptance.



## License

Drag & Drop is MIT-licensed, copyright 2026 Gerkinfeltser. The canonical `LICENSE` is in the public-source checkout (`_releases/DragAndDrop/LICENSE` in the development workspace); packaging includes it at the archive root. Dependency copyrights and terms are retained separately in the CommonLibSSE license and third-party notices.

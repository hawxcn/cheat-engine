# AGENTS.md

## Project Shape
- Main desktop app is the Lazarus/Free Pascal project `Cheat Engine/cheatengine.lpi`; entrypoint is `Cheat Engine/cheatengine.lpr`.
- `Cheat Engine/cecore.lpi` is a separate Android-oriented core project, not the desktop GUI.
- Major optional components are separate builds: DBK kernel driver under `DBKKernel/`, DBVM hypervisor under `dbvm/`, Linux/Android `ceserver` under `Cheat Engine/ceserver/`, DirectX hooks under `Cheat Engine/Direct x mess/`, .NET/Mono collectors under their own `.sln` files, and TCC under `Cheat Engine/tcclib/`.

## Build Commands
- Local GUI build documented in `README.md`: install Lazarus 2.2.2 with FPC 3.2.2 and the cross-i386 addon, open `Cheat Engine/cheatengine.lpi`, then `Run -> Build` or `Shift+F9`.
- Headless Lazarus build equivalent for the main app, when `lazbuild` is on PATH: `lazbuild "Cheat Engine/cheatengine.lpi" --build-all`.
- CI (`appveyor.yml`) uses a legacy Visual Studio 2017 image, installs Lazarus 1.6.4/FPC 3.0.2, then builds every `.lpi` with `c:\lazarus\lazbuild "<project>.lpi" --build-all`; tests are disabled (`test: off`).
- README lists secondary projects that must be built separately for full feature coverage: `speedhack.lpr`, `luaclient.lpr`, `DirectXMess.sln`, `DotNetCompiler.sln`, `MonoDataCollector.sln`, `DotNetDataCollector.sln`, `DotNetInvasiveDataCollector.sln`, `CEJVMTI.sln`, `tcclib.sln`, `vehdebug.lpr`, and `DBKKernel.sln`.
- `.sln` projects generally require Visual Studio 2017 per `README.md`; do not assume Lazarus builds those native/C# helpers.

## Focused Component Commands
- Linux `ceserver`: run `make` from `Cheat Engine/ceserver/gcc/`; output target is `./ceserver`.
- Android `ceserver`: run `ndk-build` from `Cheat Engine/ceserver/ndk-build/EXECUTABLE/`; configured ABIs are `x86`, `armeabi-v7a`, and `arm64-v8a`.
- `ceserver` extension for Linux: run `make` from `Cheat Engine/ceserver/extension/gcc/`; output target is `./libceserver-extension.so`.
- TCC subtree: run `./configure` before `make` in `Cheat Engine/tcclib/`; `make test` runs all TCC tests, `make tests2.37` runs one tests2 case, and `make tests2.37+` updates that expected output.
- Lua 5.3 subtree under `Cheat Engine/lua53/lua53/` requires an explicit platform target such as `make mingw` or `make linux`; plain `make` prints the supported platforms.

## Verification Reality
- There is no repo-wide unit test command; AppVeyor explicitly disables tests.
- For Pascal changes, the most relevant verification is building the touched `.lpi` with `lazbuild ... --build-all` or via Lazarus.
- For `ceserver` changes, use the matching `gcc/` makefile or Android `ndk-build` directory instead of the desktop Lazarus project.
- For TCC changes, use the TCC-local commands in `Cheat Engine/tcclib/`; they depend on generated `config.mak` from `./configure`.

## Repo-Specific Gotchas
- Lazarus may rewrite project/session files when opened; keep diffs tight and avoid committing incidental `.lpi`/`.lps` churn unless it is required.
- Root `.gitignore` contains broad patterns like `*.lpi` and `*.vcxproj` even though many such files are present and important; do not delete or ignore existing project files just because the ignore file mentions them.
- Build outputs are expected under ignored paths such as `Cheat Engine/bin` and `Cheat Engine/lib`.
- Running/debugging the Windows GUI from the Lazarus IDE may require launching Lazarus as administrator.
- DBVM make targets include destructive device writes (`dbvm/Makefile` has `usb` writing to `/dev/sdb` and `disk` writing to `/dev/fd0`); do not run those targets without explicit user approval.

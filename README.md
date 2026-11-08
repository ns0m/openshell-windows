# openshell-windows

[![Build Windows binaries](https://github.com/ns0m/openshell-windows/actions/workflows/build-windows.yml/badge.svg)](https://github.com/ns0m/openshell-windows/actions/workflows/build-windows.yml)

Unofficial Windows x64 builds of [NVIDIA OpenShell](https://github.com/NVIDIA/OpenShell).

NVIDIA ships no native Windows binaries: the Windows implementation lives on the [`windows` branch](https://github.com/NVIDIA/OpenShell/blob/windows/WINDOWS.md), their CI builds it on every push to that branch but never uploads the output, and no release carries a Windows asset.
This repo rebuilds that branch on GitHub-hosted runners and publishes one portable zip per upstream commit.

## Releases

`build-windows.yml` runs daily (and on demand) and:

1. resolves the head of `NVIDIA/OpenShell@windows` and derives its version from it: highest stable tag merged into the commit, patch bump, then the commit date and short sha — e.g. `0.1.5-pre.20261102.5b9e5b91a` (base `v0.1.4`).
   The patch bump keeps it from claiming an existing upstream tag, and `pre` marks it as a prerelease of the line it leads to;
2. skips the build if that version is already released;
3. stamps it into `Cargo.toml` and builds with `mise run --skip-tools windows:build:x64` on `windows-2025`, so `--version` reports it verbatim;
4. smoke-tests `openshell.exe --version` against the tag and publishes `openshell-windows-x64.zip` containing `openshell.exe`, `openshell-gateway.exe`, `openshell-supervisor.exe`, `openshell-windows-sandbox.exe`, `libz3.dll`, `LICENSE` and `THIRD-PARTY-NOTICES`.

Only x64 binaries are published.

## Using the binaries

Download `openshell-windows-x64.zip` from the latest release and keep the four executables and `libz3.dll` in one directory: the DLL must sit next to the executables, and the gateway launches the supervisor and sandbox binaries from there.
Add that directory to `PATH` and check the CLI:

```powershell
.\openshell.exe --version
```

Then follow NVIDIA's [Windows guide](https://github.com/NVIDIA/OpenShell/blob/windows/WINDOWS.md) for the gateway config and MXC setup.

## Caveats

- Not built, qualified, or signed by NVIDIA.
  NVIDIA marks the native Windows backend as under development and not production-qualified.
- Native sandboxing needs [MXC](https://github.com/microsoft/mxc) 1.0.0+ and a host that passes `wxc-exec --probe`, plus `compute_driver = "mxc"` in the gateway config.
- Binaries follow an unstable branch, so a build can regress at any time.
  `0.1.5-pre.<date>.<sha>` is a build of the line leading to `0.1.5`, not that release: the branch also carries later `main` commits and Windows-specific patches.
- Artifacts are Apache-2.0, © NVIDIA Corporation and affiliates.
  This repo is only a build wrapper.

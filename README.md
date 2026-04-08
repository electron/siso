# electron/siso

Patched builds of [siso](https://chromium.googlesource.com/build/+/HEAD/siso/)
for Electron's Windows CI. We carry a small patch set on top of the exact
revision Chromium pins via `siso_version` in `src/DEPS`, so the resulting
binary differs from the CIPD one only by our fixes.

Currently carried:

- `0001` — retry `ERROR_INVALID_PARAMETER` when opening subninja files on
  Windows. Works around intermittent `CreateFileW` failures on container
  bind-mounted volumes (`bindflt`/`wcifs`) that otherwise abort the manifest
  load with `failed to load build.ninja: ... The parameter is incorrect.`
  Tracking upstream at `chromium-review` (CL pending).

## Layout

```
vendor/build/   git submodule -> chromium.googlesource.com/build (pinned)
patches/        numbered git-format-patch files applied on top
script/         apply-patches, build
out/            build output (gitignored)
```

## Build locally

```sh
git submodule update --init
script/apply-patches
script/build                          # -> out/siso-windows-amd64.exe
GOOS=windows GOARCH=arm64 script/build
```

## Releasing

Push a tag (`vX.Y.Z-electron.N`). The `release` workflow cross-compiles
Windows amd64 and arm64 binaries and attaches them, with sha256 sums, to the
GitHub release.

## Consuming in Electron CI

`depot_tools/siso.py` honors `SISO_PATH`. In the Windows build job, download
the release asset and export `SISO_PATH` pointing at it before invoking
`e build` / `autoninja`.

## Bumping upstream

```sh
git -C vendor/build fetch origin
git -C vendor/build checkout <new-sha-from-src/DEPS-siso_version>
git add vendor/build
script/apply-patches   # resolve any conflicts, then `git format-patch` to refresh patches/
```

## Adding a patch

Make the change inside `vendor/build`, commit it there, then:

```sh
git -C vendor/build format-patch -1 HEAD -o patches/
```

Re-run `script/apply-patches` from a clean submodule to confirm it applies.

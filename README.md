# charly-lib

The shared multi-call plugin host for [charly](https://github.com/opencharly/charly).

`charly-lib` is **one** statically-linked Go binary that hosts every "welded" charly
plugin and dispatches it by the base name it was invoked as (`argv[0]`). The distro
packages install it once at `/usr/lib/charly/charly-lib` and symlink
`/usr/lib/charly/plugins/plugin-<word>` → `charly-lib`, so the shared SDK/spec
closure is stored **once** instead of re-linked into each per-plugin binary
(opencharly/sdk#256; ~87% smaller plugin payload).

## Design

- **The plugin set is DATA** — `scripts/host-command-plugins.txt` in
  `opencharly/charly` (fetched at the `charly_ref` input). No plugin is named in
  hand-written code.
- **The host source is GENERATED** — `charly-lib-gen`
  (`opencharly/sdk/cmd/charly-lib-gen`) emits `main.go` + `go.mod` that register
  each pinned plugin into `charlylib.Run` (`opencharly/sdk/charlylib`), then the
  binary is built pure-Go (`CGO_ENABLED=0`) for amd64 + arm64.
- **The loader contract is unchanged** — a `.providers` word manifest beside an
  executable named `plugin-<word>`; charly's `bakedPluginDirs` works as-is.

## Release assets

| asset | what |
|---|---|
| `charly-lib-linux-amd64` / `charly-lib-linux-arm64` | the host binary |
| `<plugin>.providers` | each plugin's word manifest (maps `class:word` → the `plugin-<word>` symlink) |

## Building

**Actions → build → Run workflow.** Inputs:

- `charly_ref` — the charly ref holding `scripts/host-command-plugins.txt` (the pins).
- `sdk_ref` — the opencharly/sdk ref carrying `cmd/charly-lib-gen`.
- `sdk_version` — the `github.com/opencharly/sdk` Go module version the host requires
  (must be compatible with every pinned plugin — see opencharly/charly#605).
- `version` — a CalVer to tag + publish; empty = build-only (no release).

The build clones the SDK (generator) and every pinned plugin, generates the host,
builds both arches, emits the `.providers` manifests, verifies every word dispatches
by running each `plugin-<word>` symlink in go-plugin serve mode, and (when `version`
is set) publishes the release.

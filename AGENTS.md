# AGENTS.md — charly-lib

Standalone repo for the shared multi-call plugin host: **one** statically-linked
Go binary that hosts every "welded" charly plugin and dispatches it by the base
name it was invoked as (`argv[0]`). The distro packages install it once at
`/usr/lib/charly/charly-lib` and symlink `/usr/lib/charly/plugins/plugin-<word>`
→ `charly-lib`, so the shared SDK/spec closure is stored once instead of
re-linked into each per-plugin binary. This repo has **no in-repo Go module** —
the host source (`main.go` + `go.mod`) is generated at build time.

Canonical files:

- `.github/workflows/build.yml` — the `workflow_dispatch` build: clones the SDK
  generator + every pinned plugin, generates the host via `charly-lib-gen`,
  builds amd64 + arm64 + armv7 pure-Go, emits the `.providers` manifests, verifies
  every word dispatches in go-plugin serve mode, and (when `version` is set)
  publishes the release.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — history (the PR body IS the changelog).
- `README.md` — user overview only; never agent guidance.

There is no root `charly.yml` and no candy in this repo; it carries no `skill:`
entity, so no owning `/charly-<family>:<name>` skill is projected into the
marketplace corpus.

## Load these skills first (R0)

- `/charly-internals:plugin` — the plugin authoring reference: the unified
  Provider model, the plugin SDK (`github.com/opencharly/sdk`), and the
  `charlylib` host this repo builds. Load before touching the host or its
  plugin set.
- `/charly-internals:git-workflow` — before any git/PR action.

There is no dedicated `/charly-*:charly-lib` owning skill — the gap (no owning
skill projected) is recorded against `opencharly/opencharly#291`.

## Build / validate / test

- The build is the `.github/workflows/build.yml` `workflow_dispatch` (Actions →
  build → Run workflow). Its inputs: `charly_ref` (the ref holding
  `scripts/host-command-plugins.txt`, the pins), `sdk_ref` (the ref carrying
  `cmd/charly-lib-gen`), `sdk_version` (the `github.com/opencharly/sdk` module
  version the host requires), and `version` (a CalVer to tag + publish; empty =
  build-only). There is no in-repo Go module to `go build`/`go test`.
- The build itself is the gate: it generates the host, builds all three arches,
  emits the `.providers` manifests, and verifies every `plugin-<word>` symlink
  dispatches in go-plugin serve mode.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate.

## Modify this repo

- The plugin set is **DATA** (`scripts/host-command-plugins.txt` in
  `opencharly/charly`, fetched at the `charly_ref` input) — no plugin is named in
  hand-written code. Do not hard-code a plugin into this repo.
- `armv7` is the 32-bit appliance target and builds as `GOARCH=arm GOARM=7`
  (`armv7` is not a valid `GOARCH`).
- Keep the loader contract unchanged: a `.providers` word manifest beside an
  executable named `plugin-<word>`, which charly's `bakedPluginDirs` reads as-is.

## Landing

Every change lands through a pull request gated by the org-required
`charly/pr-validator`. The landing mechanics — the `feat/` branch, the PR-only
rule, `CHANGELOG/` history, and the tag-on-merge CalVer — are owned by
`/charly-internals:git-workflow` and the umbrella `AGENTS.md` /
`charly/AGENTS.md`; this signpost points at them and does not restate them.

---
spec_id: "01"
spec_name: "go_project_skeleton"
title: "Minimal Go Project Skeleton"
status: "active"
created_at: "2026-10-15T15:37:01.594088Z"
updated_at: "2026-10-15T15:37:01.594088Z"
intent_hash: "cfc6ba790daaa182358c2d548ff1604765a6b548617f9517b93ee5d8e5c0d89f"
schema_version: 2
source: "docs/prd/01_initial_prd.md"
---
## Intent

Give the repository the smallest conventional Go project layout (`cmd/`, `internal/`, `Makefile`, `go.mod`) so that later specs can add code to a working, buildable foundation. The skeleton carries no application logic.

## Goals

- `go.mod` exists at the repository root with a valid module path and Go version.
- `cmd/` and `internal/` directories exist and are tracked by git.
- A `Makefile` at the root provides the basic developer targets: `build`, `test`, `fmt`, `vet`, `check`, `doc`, `clean`, and a default `help`/`all` target.
- `make check` succeeds on the freshly created skeleton, on Linux and macOS, with only the Go toolchain installed.
- The documentation files that the project steering requires exist with accurate, minimal content.

## Non-goals

- No application code, no `main` package, no sample binary, no tests. Later specs add those.
- No CI configuration, linter configuration (e.g. golangci-lint), Dockerfile, release tooling, or pre-commit hooks.
- No third-party dependencies; `go.mod` has no `require` block.
- No other top-level directories (`pkg/`, `api/`, `scripts/`, `vendor/`, and so on).
- No changes to `LICENSE`. Changes to `.gitignore` are out of scope except where required by Requirement 2.
- No REST API or CLI design. The docs say none exists yet.

## Background

The repository today holds only `LICENSE`, `README.md` ("upstream-repo — An 'upstream' repo used for testing"), `.gitignore`, `docs/prd/01_initial_prd.md`, and `.specs/steering.md`. There is no Go code and no `go.mod`. The `.gitignore` is the generic Python template. It has no Go-specific entries, and it ignores `build/`, `dist/` and `lib/`, so build output must not go into those directories.

The git remote is `https://github.com/agent-fox-dev/forked-repo`, which gives the module path. The input asks only for the four skeleton items and says to keep the Makefile to "the basics".

`.specs/steering.md` is the project directive. Several of its rules assume a Makefile and a docs set:
- `make doc` writes `go doc -all` output for every public package (excluding `internal/` and `examples/`) to `docs/generated/`, one text file per package.
- `make check` is run after documentation updates.
- Docs to maintain: `docs/api.md`, `docs/cli.md`, `docs/configuration.md`, `docs/architecture.md`, `README.md`. A missing doc must be created, and a repository without a REST API or CLI keeps the file and says in one line that it has none.
- Prefer the standard library and avoid new dependencies.
- The target platforms are Linux and macOS.

"Basics" is therefore read to include `check` and `doc`, because the steering depends on them.

## Requirements

1. **Module file.** The repository root has a `go.mod` with `module github.com/agent-fox-dev/forked-repo` and a `go` directive of `1.22`. It has no `require` entries.

2. **Directories.** `cmd/` and `internal/` exist at the root. Git does not track empty directories, so each contains a placeholder file named `.gitkeep`. Specs that add real code to a directory may delete its placeholder. The `.gitignore` must not ignore the placeholders, `cmd/`, `internal/`, or the Makefile output directory, except for build artefacts as in Requirement 4.

3. **Makefile targets.** The root `Makefile` declares all its non-file targets as `.PHONY` and provides:
   - `help`: the default goal. It lists the targets with a one-line description each.
   - `build`: runs `go build ./...`. Once `cmd/` contains `main` packages, binaries are written to `bin/`. Until then the target must still succeed.
   - `test`: runs `go test ./...`.
   - `fmt`: runs `gofmt -w` over the Go sources, or `go fmt ./...`.
   - `vet`: runs `go vet ./...`.
   - `check`: runs `fmt`, `vet`, `build` and `test` in that order, and fails if any step fails.
   - `doc`: regenerates `docs/generated/` by running `go doc -all` for each public package, as in Requirement 6.
   - `clean`: removes `bin/` and does nothing else. It must not fail when `bin/` is absent.

4. **Build artefacts.** `.gitignore` gains a `bin/` entry. `docs/generated/` stays tracked, as the steering requires.

5. **Empty-tree behaviour.** With no Go source files, `make build`, `make test`, `make vet`, `make fmt`, `make check` and `make doc` exit with status 0. Go's "matched no packages" warning is acceptable. `make doc` on an empty tree creates `docs/generated/` (with a `.gitkeep` if it is otherwise empty) and writes no package files.

6. **Doc generation.** `make doc` enumerates packages with `go list ./...`. It drops any whose import path contains `/internal/` or `/examples/`, or ends in `/internal` or `/examples`. For each remaining package it writes `go doc -all <importpath>` output to a file under `docs/generated/`. The file name is the import path relative to the module, with `/` replaced by `_` (the module root package is `root.txt`). Stale files from removed packages are deleted before regeneration. The target uses only the Go toolchain and POSIX shell tools available on Linux and macOS.

7. **Portability.** The Makefile works with both GNU make and the macOS system make (3.81). It uses no GNU-only extensions and no bash-only syntax. It is overridable through a `GO` variable (default `go`).

8. **Required docs.** The following files exist with real content, not placeholders:
   - `docs/architecture.md`: describes the layout (`cmd/` for entry points, `internal/` for private packages, `docs/generated/` for generated reference) and the module path.
   - `docs/api.md`: one line saying there is no REST API or public Go API yet.
   - `docs/cli.md`: one line saying there is no CLI yet, plus the Makefile targets as developer commands.
   - `docs/configuration.md`: one line saying there are no config keys or environment variables yet, other than the Makefile `GO` variable.
   - `README.md` gains a short section with the layout, the Go version and the `make` targets. Its existing title and description line are kept.
   - There are no tests for the presence or content of docs.

9. **No behaviour beyond the skeleton.** No Go source files with declarations are added. `gofmt -l .` produces no output.

## Design Decisions

1. **Module path is `github.com/agent-fox-dev/forked-repo`.** The input gives none; the git remote is the only evidence, and it is the conventional choice.
2. **Go version directive is `1.22`.** The input names no version. 1.22 is old enough to be widely installed and new enough to be supported, and it avoids toolchain auto-download directives.
3. **Placeholder `.gitkeep` files in `cmd/` and `internal/`.** The input wants the directories present and git cannot track empty ones. This is the least invasive way to do it, and it adds no Go code.
4. **No `main.go` and no sample package.** The input says "nothing else for now". An empty tree is valid for Go tooling, so `make check` still passes.
5. **Makefile includes `check` and `doc` beyond build/test.** The steering requires `make doc` and `make check`. "The basics" is read to include them. `fmt`, `vet`, `clean` and `help` are the other standard basics.
6. **Docs files are created despite "nothing else".** The steering says docs are regressions if missing, and it names the one-line "none yet" form for repositories without an API or CLI. The input's "nothing else" is read as limiting code and directory layout, not documentation. This is the decision most likely to be second-guessed.
7. **Binaries go to `bin/`.** The existing `.gitignore` already ignores `build/` and `dist/`, but `bin/` is the conventional Go location and is explicit. A `.gitignore` entry for `bin/` is added.
8. **Generated doc naming is flat (`_` for `/`).** The steering asks for "one text file per package" in `docs/generated/`. A flat layout is simple to produce in portable shell and to clean.
9. **No linter and no `tools.go`.** The steering prefers the standard library and no new dependencies. `go vet` and `gofmt` cover the basics.
10. **`fmt` runs in `check`.** It is kept as a rewriting step to keep the Makefile simple, which means `check` may modify files. Failing on unformatted code would be stricter but is not needed for a skeleton.

## Dependencies

None. No specs exist yet, and `.specs/` contains only `steering.md`.

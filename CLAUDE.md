# llm-forge

## Project Overview

llm-forge is a **universal translation hub** for AI developer tooling. It solves the fragmentation problem across AI coding tools (Claude Code, opencode, Codex, Copilot, Hermes, Pi): each tool invents its own format for settings, memories, skills, agents, and project context. llm-forge defines one canonical Go type for each asset category and provides per-tool adapters that translate in both directions. Define your configuration once; keep every tool in sync.

**Single Go binary. No Python. No IPC boundary.**

## Codebase Map

| Path | Purpose |
|------|---------|
| `cmd/forge-cli/main.go` | Binary entry point — CLI flag parsing, adapter registration |
| `internal/canonical/` | ForgeAsset Go struct definitions — start here |
| `internal/uil/uil.go` | Universal Interface Layer — mandatory gateway for all file I/O and shell ops |
| `internal/adapters/` | Per-tool adapters (`claudecode/`, `opencode/`, `codex/`, `copilot/`) + `Adapter` interface + `Registry` |
| `internal/engine/translation.go` | TranslationEngine — orchestrates import/export/sync flows |
| `internal/engine/skills.go` | SkillExecutor — loads and dispatches YAML skill catalog |
| `internal/memory/db.go` | ForgeDB — SQLite persistence for ForgeRecord and ForgeAsset |
| `internal/tui/tui.go` | TUI — bubbletea (Phase II) |
| `schemas/forge_skill_catalog.yaml` | Skill definitions loaded at runtime |
| `.opencode/specs/` | Canonical design specs — read before implementing any component |

## Build & Run Commands

```bash
# Install dependencies
go mod tidy

# Build binary
go build -o forge ./cmd/forge-cli/

# Run
./forge import --from claude_code --asset memory
./forge export --to opencode --asset memory
./forge sync --from claude_code --to opencode

# Test (all packages)
go test ./...

# Test (single package)
go test ./internal/canonical/...

# Test (by name filter)
go test ./... -run TestForgeMemory
```

## Architecture

```
User
  │
  ▼
[forge-cli]                          ← cmd/forge-cli/main.go
  │  (single Go binary — no IPC)
  │
  ├──► [TUI]                         ← internal/tui/  (Phase II: bubbletea)
  │
  ├──► [TranslationEngine]           ← internal/engine/translation.go
  │        │
  │        ├──► [AdapterRegistry]    ← internal/adapters/registry.go
  │        │        │
  │        │        └──► [Adapters]  ← internal/adapters/<toolname>/adapter.go
  │        │                 ImportAsset() ↔ ExportAsset()
  │        │                 all I/O via UIL only
  │        │
  │        └──► [UIL]                ← internal/uil/uil.go
  │                 cross-platform file I/O, path resolution, shell exec
  │
  ├──► [SkillExecutor]               ← internal/engine/skills.go
  │
  └──► [MemoryManager / ForgeDB]     ← internal/memory/db.go
           SQLite: ForgeRecord (audit) + ForgeAsset tables (content)
```

**Key boundaries:**
- UIL is the only package that imports `os`, `path/filepath`, or `os/exec`. Nothing else does.
- Adapters own all format knowledge. Nothing outside an adapter package reads or writes a tool's files.
- TranslationEngine orchestrates flows but holds zero format knowledge.

## Key Patterns

**ForgeAsset** — canonical Go struct for any translatable config. Five types: `ForgeSettings`, `ForgeMemory`, `ForgeSkill`, `ForgeAgent`, `ForgeContext`. All embed `ForgeAsset` base. Defined in `.opencode/specs/ForgeAsset_Schema.md`.

**ForgeRecord** — execution audit log entry. One per operation. Separate SQLite table from ForgeAsset. Defined in `.opencode/specs/ForgeAsset_Schema.md`.

**Adapter interface** — `ToolID()`, `SupportedAssets()`, `ImportAsset()`, `ExportAsset()`. One package per tool under `internal/adapters/<toolname>/`. Adding a tool = one new package. Defined in `.opencode/specs/AdapterLayer_Contract.md`.

**UIL contract** — `ResolvePaths`, `ReadFile`, `ListDir`, `WriteFile`, `ExecCommand`, `FileExists`, `DirExists`. All return typed errors. Defined in `.opencode/specs/PlatformAbstractionLayer_Contract.md`.

**Phase gating** — 4 phases. Phase I (single adapter, ForgeMemory, full round-trip) must pass 4 criteria before Phase II. Defined in `.opencode/specs/FORGE_SYSTEM_PLANNING_SPEC-V3.md`.

## Key Go Dependencies

| Module | Purpose |
|---|---|
| `gopkg.in/yaml.v3` | YAML parsing (skill catalog, opencode config) |
| `zombiezen.com/go/sqlite` | SQLite — no CGO required |
| `github.com/go-playground/validator/v10` | Struct field validation |
| `github.com/charmbracelet/bubbletea` | TUI (Phase II) |

## Session Status

- [ ] Phase I not started — Go directory structure scaffolded, no implementation yet
- [x] go.mod created (Go 1.26, module `llm-forge`)
- [x] Go-first architecture — single binary, no Python/IPC boundary
- [x] All spec files updated: V3 spec, ForgeAsset schema, Adapter contract, UIL contract
- [x] Full directory structure scaffolded under `internal/`

**First file to implement:** `internal/canonical/assets.go`
See `.opencode/specs/ForgeAsset_Schema.md` for the exact struct definitions.

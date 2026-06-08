# llm-forge — Project Rules for OpenCode

> opencode project context. For full architecture and build commands, also read `CLAUDE.md`.

## Project

llm-forge is a **universal translation hub** for AI developer tooling. It translates settings, memories, skills/agents, and project context between AI coding tools (Claude Code, opencode, Codex, Copilot, Hermes, Pi). Single Go binary — no Python, no IPC boundary.

## Collaboration Model

**User writes all code.** The LLM role is limited to:
- Product management (planning, task breakdown, roadmap)
- Documentation (specs, AGENTS.md, CLAUDE.md, architecture docs)
- Creating boilerplate folder/file scaffolding when asked

Never write implementation code unprompted. Respond to feature requests with planning artifacts — not code.

## Key Rules

- **UIL-first:** All file I/O and shell ops must go through `internal/uil/`. No package imports `os`, `filepath`, or `exec` directly.
- **ForgeAsset:** Every translated config object is a canonical Go struct. Five types: Settings, Memory, Skill, Agent, ProjectContext.
- **ForgeRecord:** Every operation produces an audit log entry. Separate SQLite table from ForgeAsset.
- **Adapter boundary:** Only adapter packages know tool-specific file formats. Nothing outside `internal/adapters/<toolname>/` reads or writes a tool's files.
- **Phase-gated:** Do not implement Phase II until Phase I passes all 4 test criteria.
- **Docker-native:** All code must run inside Docker for cross-platform portability.
- **Single binary:** No Python, no subprocess IPC. One `go build` produces the deliverable.

## Language & Build

```bash
go build -o forge ./cmd/forge-cli/
go test ./...
```

## Config

- PM config: `.opencode/pm-config.md`
- Project board: `.opencode/project_board.md`
- Memory: `.opencode/memory/`
- Specs: `.opencode/specs/`

Agents read `.opencode/pm-config.md` at Step 0. Fall back to `.claude/pm-config.md` if absent.

## Specs

| Spec | Purpose |
|---|---|
| `.opencode/specs/FORGE_SYSTEM_PLANNING_SPEC-V3.md` | Canonical system spec — read this first |
| `.opencode/specs/ForgeAsset_Schema.md` | Go struct definitions for all canonical types |
| `.opencode/specs/AdapterLayer_Contract.md` | Go Adapter interface + per-tool format reference |
| `.opencode/specs/ForgeRecord_Schema.md` | Execution audit log schema |
| `.opencode/specs/PlatformAbstractionLayer_Contract.md` | UIL Go function contracts |

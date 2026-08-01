# Changelog

## [1.1.0] - 2026-07-31

### Changed
- Removed all hardcoded command references (refine/bugfix). Everything is now config-driven via `lifecycle-config.yml`.
- `check` command: reads phase from state file, blocks based on config phase definition (no hardcoded lists).
- `lock` command: accepts `--phase` to transition to any named phase (default: `"locked"`). Descriptions read from config.
- `unlock` command: accepts `--phase` and `--spec-dir` for cross-plugin use. Generic interface, any plugin can call it.
- `status` command: builds command matrix dynamically from config phases, not hardcoded SDD categories.
- `write-agents` command: reads blocked/allowed patterns from config, writes to agent context with phase metadata.
- `extension.yml` hooks: generic descriptions. `after_lifecycle_lock` and `after_lifecycle_unlock` fire for any caller.
- `lifecycle-config.yml`: restructured with `lock_target`, `unlock_target`, and `phases` map for multi-phase support.

### Added
- Multi-phase support: `phases` map in config allows defining any number of lifecycle phases.
- `--phase` parameter on `lock` and `unlock` for custom phase transitions.
- `--reason` parameter on `lock` and `unlock` for audit trail.
- `--yes` / `-y` flag for automated/unattended transitions.
- `reason` field in `.lifecycle.json` state file.
- Phase-to-config matching in `check` command (glob pattern support).

### Removed
- All hardcoded mentions of refine/bugfix in command descriptions and documentation.
- Binary active/locked assumption — now multi-phase capable.

## [1.0.0] - 2026-07-31

### Added
- Initial release
- `speckit.lifecycle.check` — guard against locked specs
- `speckit.lifecycle.lock` — lock spec artifacts
- `speckit.lifecycle.unlock` — unlock spec (supports `--spec-dir`)
- `speckit.lifecycle.status` — show current lifecycle phase
- `speckit.lifecycle.write-agents` — sync lock status to AGENTS.md
- Per-spec lifecycle state (`specs/NNN-feature/.lifecycle.json`)
- Configurable blocked/allowed commands via `lifecycle-config.yml`
- Mandatory `before_*` hooks for core SDD commands
- Cross-plugin interface: `unlock --spec-dir <path>` for external plugins

# Changelog

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

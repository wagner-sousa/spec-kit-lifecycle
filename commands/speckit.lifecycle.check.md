---
description: "Check lifecycle phase — blocks execution based on phase config"
---

# Lifecycle Check

Guard command. Runs as a mandatory `before_*` hook for any command you want
to protect. Reads `specs/{feature}/.lifecycle.json`, looks up the current
phase in `lifecycle-config.yml`, and either passes or blocks based on the
phase's command restrictions.

## Prerequisites

1. Load `lifecycle-config.yml` from the extension directory.
2. Determine active spec directory from `.specify/feature.json` → `feature_directory`.
3. Read `{feature_directory}/.lifecycle.json` — extract `phase` field.

## Decision

### No `.lifecycle.json` found → PASS

```
✅ Lifecycle: active (default — no lock file)
```

### Phase matches `unlock_target` (default: `active`) → PASS

```
✅ Lifecycle: active
All commands available.
```

### Phase has restrictions in config → CHECK

For each command about to run, check if it matches any pattern in the phase's
`blocked` list. If it does AND does NOT match any pattern in the `allowed` list → BLOCK.

```
⛔ SPEC IS [phase_name]

This spec is in phase "[phase_name]" since [locked_at].
Reason: [reason if set]

The following commands are blocked in this phase:
  - [list of blocked commands from config]

To proceed:
- /speckit.lifecycle.unlock — transition back to active phase
- /speckit.lifecycle.status — see full command matrix
```

**Do NOT proceed with the parent command. Stop execution here.**

## Rules

- Read-only — never modifies any file
- If `.lifecycle.json` is malformed or missing `phase`, treat as `active` (fail-open)
- If `lifecycle-config.yml` is missing, treat as `active` (fail-open)
- Pattern matching: exact match first, then glob (`*` for any, `prefix.*` for subcommands)
- `allowed` overrides `blocked` when a command matches both
- After outputting BLOCK, do not continue the parent command execution

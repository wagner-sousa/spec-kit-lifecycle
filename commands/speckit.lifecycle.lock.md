---
description: "Transition spec to locked phase — blocks commands per lifecycle-config.yml"
---

# Lock Lifecycle

Transition the active feature's spec to a restricted phase. Which commands
are blocked and which are allowed is defined entirely in `lifecycle-config.yml`
— no command names are hardcoded.

## Parameters

| Param | Required | Description |
|-------|----------|-------------|
| `--phase` | No | Phase name to transition to. Defaults to `lock_target` from config (default: `"locked"`) |
| `--spec-dir` | No | Target a specific spec directory. Defaults to active spec from `feature.json` |
| `--reason` | No | Optional description of why the phase changed |
| `--yes` / `-y` | No | Skip confirmation prompt |

## Action

1. **Determine target**: If `--spec-dir` provided, use it. Otherwise read `.specify/feature.json` for active spec.

2. **Load config**: Read `lifecycle-config.yml`. If missing, use defaults (locked → `"locked"`).

3. **Read current state**: Check `{spec_dir}/.lifecycle.json`. If already at target phase, output:
   ```
   ℹ️ Spec is already in phase "[phase]". Nothing to change.
   ```

4. **Confirm** (unless `--yes`):
   ```
   Lock spec lifecycle to phase "[phase]"?
   
   Blocked commands: [list from config]
   Allowed commands: [list from config]
   
   Proceed? (yes/no)
   ```

5. **If confirmed**, write `.lifecycle.json`:
   ```json
   {
     "phase": "[phase]",
     "locked_at": "[ISO_DATE]",
     "locked_by": "[caller or 'user']",
     "reason": "[reason if provided]"
   }
   ```

6. **Output confirmation**:
   ```
   🔒 Spec lifecycle: [spec_name]
   Phase: [phase]
   
   Blocked:  [blocked commands from config]
   Allowed:  [allowed commands from config]
   
   Use /speckit.lifecycle.status to verify.
   To unlock: /speckit.lifecycle.unlock
   ```

## Rules

- Requires explicit user confirmation unless `--yes` is passed
- Writes valid JSON to `{spec_dir}/.lifecycle.json`
- Creates the spec directory if it does not exist
- The `after_lifecycle_lock` hook fires after this command completes

---
description: "Transition spec back to active phase — re-enables unrestricted commands"
---

# Unlock Lifecycle

Transition a spec back to the active (unrestricted) phase. After unlocking,
all commands return to normal operation as defined by the active phase in
`lifecycle-config.yml`.

## Parameters

| Param | Required | Description |
|-------|----------|-------------|
| `--phase` | No | Phase to transition to. Defaults to `unlock_target` from config (default: `"active"`) |
| `--spec-dir` | No | Target a specific spec directory. Defaults to active spec from `feature.json` |
| `--reason` | No | Optional description of why the phase changed |
| `--yes` / `-y` | No | Skip confirmation prompt |

## Action

1. **Determine target**: If `--spec-dir` provided, use it. Otherwise read `.specify/feature.json`.

2. **Read current state**: Check `{spec_dir}/.lifecycle.json`. If not found or already at target phase:
   ```
   ℹ️ Spec is already in phase "[phase]". Nothing to unlock.
   ```

3. **Confirm** (unless `--yes`):
   ```
   Unlock spec lifecycle from "[current_phase]" to "[target_phase]"?
   All commands will be re-enabled.
   Proceed? (yes/no)
   ```

4. **If confirmed**, update `.lifecycle.json`:
   ```json
   {
     "phase": "[target_phase]",
     "locked_at": null,
     "locked_by": null,
     "unlocked_at": "[ISO_DATE]",
     "reason": "[reason if provided]"
   }
   ```

5. **Output confirmation**:
   ```
   🔓 Spec lifecycle: [spec_name]
   Phase: [target_phase]
   
   All commands are now available.
   
   To lock again: /speckit.lifecycle.lock
   ```

## Cross-Plugin Usage

Any extension or workflow can unlock a specific spec:
```
/speckit.lifecycle.unlock --spec-dir specs/013-fix-auth --yes
```

The `--yes` flag skips confirmation for automated callers. The
`after_lifecycle_unlock` hook fires after this command completes,
syncing the change to the agent context.

## Rules

- Requires explicit user confirmation unless `--yes` is passed
- `--spec-dir` allows external callers (plugins, workflows) to target any spec
- After unlock, the current phase's command restrictions (usually none for `active`) apply
- The `after_lifecycle_unlock` hook fires after this command completes

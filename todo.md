# Lifecycle — Planned Improvements

## 1. Transition Validation

Add `transitions` map to `lifecycle-config.yml` to define valid phase transitions.

```yaml
# New field in lifecycle-config.yml
transitions:
  active: [locked, review]
  locked: [active]
  review: [locked, active]
```

- `lock --phase X` rejects invalid transitions with an error message listing valid targets.
- If `transitions` is not defined, any transition is allowed (backward compatible).
- `unlock` also validates against transitions if the target phase is locked phase.

**State file**: unchanged, but `to` phase is checked against valid transitions before writing.

---

## 2. Phase History

Add `history` array to `.lifecycle.json` for full audit trail.

```json
{
  "phase": "locked",
  "locked_at": "2026-08-01T00:00:00",
  "locked_by": "user",
  "reason": "Spec finalized",
  "history": [
    { "from": "active", "to": "locked", "at": "2026-08-01T00:00:00", "by": "user", "reason": "Spec finalized" }
  ]
}
```

- Every `lock`/`unlock` appends to `history` in addition to updating current fields.
- `status` shows the last N entries (default: 5).
- New command: `lifecycle.history` — shows full timeline as table.
- `history` supports `--spec-dir` for cross-plugin access.

**Command template**: `commands/speckit.lifecycle.history.md`

---

## 3. Generic Hook: `after_lifecycle_phase_changed`

New hook that fires on **any** phase transition, not just lock/unlock.

```yaml
# In extension.yml, new hook entry:
after_lifecycle_phase_changed:
  command: speckit.lifecycle.write-agents
  optional: false
  description: "Generic post-transition hook — fires on any phase change. Includes from_phase, to_phase, triggered_by, reason in payload."
```

- Existing `after_lifecycle_lock` and `after_lifecycle_unlock` remain as convenience aliases.
- Payload: `{ from_phase, to_phase, triggered_by, reason, spec_dir }`.
- Other extensions can subscribe to this single hook instead of both lock/unlock.
- `write-agents` registered on both the old and new hooks for full coverage.

---

## 4. TTL / Auto-Expiry

Optional expiration on locked phases.

```json
{
  "phase": "locked",
  "locked_at": "2026-08-01T00:00:00",
  "locked_by": "user",
  "expires_at": "2026-08-02T00:00:00",
  "reason": "Lock during implementation — auto-expires in 24h"
}
```

- `lock --ttl 24h` or `lock --expires-at "2026-08-02T00:00:00"`.
- `check` treats expired locks as `active` (fail-open).
- `status` shows countdown if TTL is set.
- `unlock` clears `expires_at` explicitly.
- No background process needed — check on read only.

**Config**: optional `default_ttl` in `lifecycle-config.yml` for auto-apply.

---

## 5. Dry-Run Mode

Preview what a lock/unlock would do without applying.

```
/speckit.lifecycle.lock --dry-run
/speckit.lifecycle.lock --phase review --dry-run
```

Output:
```
## Dry Run: lock → review

Transition: active → review (valid)

Blocked commands:
  - specify, plan, tasks, implement

Currently allowed commands that would be blocked:
  - [list of commands that would change status]

No files will be modified.
```

- `--dry-run` skips all writes and hook triggers.
- Validates transition before showing preview.
- Works for lock, unlock, and --phase variants.

---

## 6. Wildcard Block/Allow Documentation & Testing

Already partially supported via glob patterns (`*`, `prefix.*`). Needs:

- Explicit documentation of precedence rules: `allowed` wins over `blocked`.
- Test coverage for edge cases: `["*"]` with overrides, nested patterns, exact vs glob conflict.
- Config examples for common patterns (block all except X, allow all except Y).

---

## 7. Phase Templates via `extends`

Reusable phase presets.

```yaml
# Built-in templates loaded by lifecycle extension
extends: "sdd-standard"

phases:
  locked:
    blocked:
      - "specify"
    # ... user overrides
```

Built-in templates:
- `sdd-standard`: active + locked (specify/clarify/plan/tasks/checklist/analyze blocked)
- `minimal`: active + locked (only specify blocked)
- `agile`: backlog → sprint → in-review → done

Users override specific fields without redefining everything. Template merging: deep merge, user values win.

---

## 8. `lifecycle.history` Command

Show full phase transition timeline.

```
/speckit.lifecycle.history
/speckit.lifecycle.history --spec-dir specs/010-fix-login
```

Output:
```
## Lifecycle History: 010-fix-login

| # | From | To | When | By | Reason |
|---|------|----|------|----|--------|
| 1 | active | locked | 2026-07-31 22:00 | user | Spec finalized |
| 2 | locked | active | 2026-08-01 00:15 | switch plugin | Auto-unlock on spec switch |
| 3 | active | locked | 2026-08-01 00:30 | CI pipeline | Re-locked after merge |
```

- Read-only command.
- `--limit N` to cap entries (default: all).
- `--json` for programmatic output.

---

## 9. Reason Requirement on Unlock

Configurable requirement for `reason` on destructive transitions.

```yaml
# In lifecycle-config.yml
require_reason_on:
  - unlock
  - lock
```

- If `require_reason_on` includes `unlock`, `unlock` without `--reason` asks for one (even with `--yes`).
- If `require_reason_on` includes `lock`, same for `lock`.
- Default: empty list (no requirement, backward compatible).
- Promotes audit trail discipline.

---

## Priority / Sequencing

| Priority | ID | Improvement | Reason |
|----------|----|-------------|--------|
| P0 | 1 | Transition Validation | Foundation for multi-phase model |
| P0 | 2 | Phase History | Enables #8 (history command), audit prerequisite |
| P0 | 3 | Generic phase_changed hook | Required for external integrations |
| P1 | 5 | Dry-Run | Safety net for config changes |
| P1 | 8 | history command | User-facing value, uses #2 |
| P2 | 4 | TTL / Expiry | Nice-to-have, independent |
| P2 | 9 | Reason requirement | Small UX improvement |
| P3 | 6 | Wildcard docs | Documentation debt |
| P3 | 7 | Phase templates | Complex, less immediate value |

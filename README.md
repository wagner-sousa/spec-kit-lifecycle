# SDD Lifecycle Manager

Generic phase-gate for Spec Kit. Define lifecycle phases. Block or allow
any command per phase. Any plugin or hook can trigger lock/unlock via
`--spec-dir <path>`.

Per-spec architecture: each feature's lifecycle state lives inside its own
directory at `specs/NNN-feature/.lifecycle.json`.

## Philosophy

Lifecycle Manager does **not** hardcode which commands are blocked.
Everything is driven by `lifecycle-config.yml`. You define phases, and
for each phase you specify which command patterns are blocked or allowed.

The `lock` and `unlock` hooks (`after_lifecycle_lock` / `after_lifecycle_unlock`)
are generic — any extension, workflow, or external tool can trigger them.
The lifecycle extension provides the mechanism; you define the policy.

## Commands

| Command | Description |
|---------|-------------|
| `/speckit.lifecycle.check` | Guard hook — reads current phase and blocks if config says so |
| `/speckit.lifecycle.lock` | Transition to locked phase — freezes commands per config |
| `/speckit.lifecycle.unlock` | Transition back to active phase — re-enables restricted commands |
| `/speckit.lifecycle.status` | Show current phase, metadata, and command matrix |
| `/speckit.lifecycle.write-agents` | Sync phase restrictions to agent context file |

## State File

Per spec: `specs/NNN-feature/.lifecycle.json`

```json
{
  "phase": "locked",
  "locked_at": "2026-07-31T22:00:00",
  "locked_by": "workflow",
  "reason": "Spec finalized — awaiting implementation"
}
```

Fields:

| Field | Type | Description |
|-------|------|-------------|
| `phase` | string | Current phase label. Default: `"active"` |
| `locked_at` | ISO string | When phase transitioned (if applicable) |
| `locked_by` | string | Who or what triggered the transition |
| `reason` | string | Optional — why the phase changed |

Discovered via `.specify/feature.json` → `feature_directory`.

## Cross-Plugin Interface

**Lock and unlock from any extension or workflow:**

```
/speckit.lifecycle.unlock --spec-dir specs/013-fix-auth
/speckit.lifecycle.lock --phase review --spec-dir specs/013-fix-auth
```

Any plugin can call `lock` or `unlock` with `--spec-dir` to target a specific
spec. This is the standard interface — lifecycle does not own the policy,
it only enforces it.

Used by `switch.set` for auto-unlock on spec checkout, for example.

## Agent Context Sync

When phase changes (lock or unlock), the `after_lifecycle_lock` and
`after_lifecycle_unlock` hooks fire automatically. The `write-agents`
command syncs current restrictions to the agent context file using
unique markers:

```
<!-- LIFECYCLE START --> ... <!-- LIFECYCLE END -->
```

## Configuration

`lifecycle-config.yml` defines the policy:

```yaml
# Which phase label to set on lock
lock_target: "locked"

# Which phase label to restore on unlock
unlock_target: "active"

# Define phases and their command restrictions
phases:
  active:
    blocked: []
    allowed: ["*"]

  locked:
    blocked:
      - "specify"
      - "clarify"
      - "plan"
      - "tasks"
      - "checklist"
      - "analyze"
      - "sync.apply"
      - "sync.backfill"
    allowed:
      - "lifecycle.*"
      - "review.*"
      - "verify.*"
      - "sync.*"
      - "git.*"
      - "doctor.*"
      - "agent-context.*"
      - "switch.*"
      - "converge"
      - "refine.*"
      - "bugfix.*"
      - "implement"
      - "taskstoissues"
      - "checklist"
      - "analyze"
      - "sync.apply"
      - "sync.backfill"

  review:
    blocked:
      - "specify"
      - "plan"
      - "tasks"
      - "implement"
    allowed:
      - "review.*"
      - "lifecycle.*"
      - "git.*"
```

- **`lock_target`**: phase name set by the `lock` command (default: `"locked"`)
- **`unlock_target`**: phase name set by the `unlock` command (default: `"active"`)
- **`phases`**: map of phase labels to blocked/allowed command patterns
- **`blocked`**: list of commands or glob patterns (`*` for all) that are blocked
- **`allowed`**: list of commands or glob patterns that are always allowed (overrides blocked)
- Wildcards: `*` matches all, `refine.*` matches all refine subcommands

**To add a new phase**, add it to `phases` and reference it by name:
```
/speckit.lifecycle.lock --phase review
```

## Hooks

Lifecycle exposes two **generic hooks** that any workflow can leverage:

| Hook | Description |
|------|-------------|
| `after_lifecycle_lock` | Fires after ANY lock transition. Syncs restrictions to agent context. |
| `after_lifecycle_unlock` | Fires after ANY unlock transition. Removes restrictions from agent context. |

These hooks are not tied to a specific phase or workflow — they fire
whenever lock or unlock completes, regardless of who triggered it.

To use the `check` guard in your own hooks, add lifecycle as a mandatory
`before_*` hook for any command you want to protect:

```yaml
before_specify:
  - extension: lifecycle
    command: speckit.lifecycle.check
    enabled: true
    optional: false
    priority: 100
```

## Behavior

| Phase | Commands blocked by config are ⛔ | Commands allowed by config are ✅ |
|-------|-----------------------------------|-------------------------------------|
| `active` | none (default) | all |
| `locked` | as defined in `lifecycle-config.yml` | as defined in `lifecycle-config.yml` |
| custom | as defined in config entry | as defined in config entry |

The `check` command reads the current phase from `.lifecycle.json`, looks
up the config, and blocks or passes based on the phase definition. If a
command matches both `blocked` and `allowed`, `allowed` wins.

## Rules

- **Config-driven**: no command names hardcoded in logic
- **Read-only guards**: check never modifies files
- **Fail-open**: malformed JSON or missing config defaults to `active`
- **Generic hooks**: usable by any extension, not lifecycle-specific
- **Per-spec granularity**: each spec has its own `.lifecycle.json`

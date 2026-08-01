---
description: "Write lifecycle phase restrictions into agent context file"
---

# Lifecycle: Write Agent Context

Synchronizes lifecycle phase restrictions to the agent context file
(AGENTS.md by default). Runs automatically as `after_lifecycle_lock`
and `after_lifecycle_unlock` hooks — generic hooks that fire regardless
of who or what triggered the lock/unlock.

## Behavior

1. **Find context file target** (first match wins):
   a. `lifecycle-config.yml` → `context_file` field (optional)
   b. `.specify/init-options.json` → detect integration → map to file
      (opencode → AGENTS.md, claude → CLAUDE.md, copilot → .github/copilot-instructions.md, etc.)
   c. Fallback: `AGENTS.md`

2. **Read `.specify/feature.json`** → extract `feature_directory` and `spec_name`.

3. **Read `{feature_directory}/.lifecycle.json`** → determine phase.

4. **Read `lifecycle-config.yml`** → get phase's blocked/allowed patterns.

5. **If phase is not the active/default** → WRITE mode:
   Build block:
   ```
   <!-- LIFECYCLE START -->
   ## 🔒 Lifecycle: [spec_name] — [phase] since [locked_at]

   **Phase**: [phase]
   **Reason**: [reason or "unspecified"]

   **Restricted commands:** [blocked patterns from config]

   ## Allowed commands
   [allowed patterns from config]

   To change phase: /speckit.lifecycle.unlock
   <!-- LIFECYCLE END -->
   ```

   Insert/replace between `<!-- LIFECYCLE START -->` and `<!-- LIFECYCLE END -->` in context file.
   Output:
   ```
   🔒 Lifecycle phase "[phase]" written to [context_file]
   ```

6. **If phase is the default/active** → REMOVE mode:
   Find and remove `<!-- LIFECYCLE START -->` ... `<!-- LIFECYCLE END -->` block.
   Output:
   ```
   ✅ Lifecycle restrictions removed from [context_file]
   ```

## Generic Hook Behavior

This command is registered as both `after_lifecycle_lock` and
`after_lifecycle_unlock` hooks in `extension.yml`. These hooks are
**generic** — they fire whenever any lock or unlock transition completes,
whether triggered by the user, a workflow, or another extension.

Other extensions can rely on these hooks to react to lifecycle changes
without coupling to lifecycle internals. For example, a CI extension
could watch for `after_lifecycle_lock` to trigger validation.

## Rules

- Creates context file if it does not exist
- Uses unique markers (`<!-- LIFECYCLE START -->` / `<!-- LIFECYCLE END -->`) separate from agent-context extension
- Reads blocked/allowed patterns from `lifecycle-config.yml` — never hardcodes command names
- If context file cannot be written, log warning and continue (non-fatal)

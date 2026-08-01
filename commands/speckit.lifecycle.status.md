---
description: "Show current lifecycle phase, metadata, and per-phase command matrix"
---

# Lifecycle Status

Display the current lifecycle phase, lock metadata, and which commands are
affected — all driven by `lifecycle-config.yml`.

## Action

1. **Read state**: Load `lifecycle-config.yml`, then read `{spec_dir}/.lifecycle.json`.

2. **If `.lifecycle.json` not found**:
   ```
   # Lifecycle Status

   **Spec**: [spec_name]
   **Phase**: active (default)
   **Lock file**: not found

   All commands are available.
   ```

3. **If found**, parse and display:
   ```
   # Lifecycle Status

   **Spec**: [spec_name]
   **Phase**: [phase]
   **Since**: [locked_at or "—"]
   **By**: [locked_by or "—"]
   **Reason**: [reason or "—"]
   ```

4. **Show command matrix** from config:

   Build a table from the phase's `blocked` and `allowed` lists.
   Group by command prefix for readability.

   ```
   ## Command Availability in phase "[phase]"

   | Command Pattern | Status |
   |----------------|--------|
   | [pattern 1]    | ⛔ Blocked |
   | [pattern 2]    | ✅ Allowed |
   | ...            | ...    |
   ```

5. **Show available transitions**:
   ```
   ## Available Phases

   | Phase | Description |
   |-------|-------------|
   | active | Unrestricted — all commands available |
   | locked | Restricted — see blocked list above |
   ```

## Rules

- Read-only — never modifies any file
- Phase list and command matrix come entirely from config
- If config is missing, show defaults (active=all, locked=none blocked)
- If JSON is malformed, report the parsing error and default to `active`

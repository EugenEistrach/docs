# Claude Code Hook Patterns

## Stop Hook: Enforce Linting Before Finish

Block agent from finishing until linting passes.

**`.claude/settings.json`:**
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/check-on-agent-finish.sh"
          }
        ]
      }
    ]
  }
}
```

**`.claude/hooks/check-on-agent-finish.sh`:**
```bash
#!/bin/bash

output=$(bun run check 2>&1)
exit_code=$?

if [ $exit_code -ne 0 ]; then
  jq -n --arg reason "⚠️  Linting errors found:

$output" '{
    decision: "block",
    reason: $reason
  }'
  exit 0
fi

exit 0
```

## Hook Types

- **Stop** - Main agent finishing
- **SubagentStop** - Subagent (Task tool) finishing
- **PostToolUse** - After tool runs (use `"matcher"` to filter: `"Write"`, `"Edit"`, `"Bash"`)
- **PreToolUse** - Before tool runs
- **UserPromptSubmit** - User submits prompt
- **SessionStart** - Session starts

## Blocking Patterns

**Exit code 2 (stderr shown to agent):**
```bash
echo "Error message" >&2
exit 2
```

**JSON decision (more control):**
```bash
jq -n --arg reason "Error: $output" '{
  decision: "block",
  reason: $reason
}'
exit 0
```

## Debugging

```bash
claude --debug
```

Look for: `[DEBUG] Getting matching hook commands`, `[DEBUG] Successfully parsed`, `[DEBUG] Failed to parse`

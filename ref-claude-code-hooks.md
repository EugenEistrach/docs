# Claude Code Hook Patterns

Reference for useful Claude Code hook configurations and patterns.

## Stop Hook: Enforce Linting Before Agent Finishes

**Use case:** Force the main agent to fix linting errors before completing their task.

**Hook configuration** (`.claude/settings.json`):
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

**Script** (`.claude/hooks/check-on-agent-finish.sh`):
```bash
#!/bin/bash

# Run linting check and capture output
output=$(bun run check 2>&1)
exit_code=$?

# If check failed, use JSON to block with detailed reason
if [ $exit_code -ne 0 ]; then
  # Properly escape output for JSON using jq
  jq -n --arg reason "⚠️  Linting errors found. Please fix these issues before completing the task:

$output" '{
    decision: "block",
    reason: $reason
  }'
  exit 0
fi

# Success - allow agent to finish (no output needed)
exit 0
```

**How it works:**
1. When agent tries to finish, `Stop` hook runs
2. Script runs `bun run check` (or your lint command)
3. If errors found, returns JSON with `"decision": "block"`
4. Agent is blocked from stopping and receives error details
5. Agent must fix errors before successfully finishing
6. Hook runs again after fixes - passes if clean

**Important notes:**
- Only works for main agent, not subagents (Task tool)
- Must return valid JSON with proper escaping (use `jq`)
- Exit code 0 required when blocking (JSON decision controls blocking)
- `"reason"` field is shown to agent to guide them

## Hook Types Reference

**Stop:** Main agent finishing (can block with `"decision": "block"`)
**SubagentStop:** Subagent finishing (can block but enforcement varies)
**PostToolUse:** After tool executes - use `"matcher"` to filter (e.g., `"Write"`, `"Edit"`, `"Bash"`)
**PreToolUse:** Before tool executes - can block with `"permissionDecision": "deny"`
**UserPromptSubmit:** User submits prompt - can block with `"decision": "block"`
**SessionStart:** Session starts - can add context via `"additionalContext"`

## Common Patterns

### Only run hook for specific operations

```json
{
  "PostToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [
        {
          "type": "command",
          "command": ".claude/hooks/check-style.sh"
        }
      ]
    }
  ]
}
```

### Block with error message (exit code 2)

```bash
#!/bin/bash
# Simple blocking - exit code 2 shows stderr to Claude
if [ some_condition ]; then
  echo "Error message here" >&2
  exit 2
fi
exit 0
```

### Block with JSON (more control)

```bash
#!/bin/bash
output=$(some_command 2>&1)
if [ $? -ne 0 ]; then
  jq -n --arg reason "Error: $output" '{
    decision: "block",
    reason: $reason
  }'
  exit 0
fi
exit 0
```

## Debugging Hooks

Run Claude Code with debug flag to see hook execution:
```bash
claude --debug
```

Look for these log lines:
- `[DEBUG] Getting matching hook commands for {HookType}`
- `[DEBUG] Matched X unique hooks`
- `[DEBUG] Successfully parsed and validated hook JSON output`
- `[ERROR]` or `[DEBUG] Failed to parse` indicates JSON issues

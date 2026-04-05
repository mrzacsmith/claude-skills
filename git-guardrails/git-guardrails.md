# Setup Git Guardrails

Set up a Claude Code PreToolUse hook that intercepts and blocks dangerous git commands before they execute.

## What Gets Blocked

| Command | Why |
|---------|-----|
| `git push` (all variants including `--force`) | Prevents accidental pushes |
| `git reset --hard` | Prevents loss of uncommitted work |
| `git clean -f` / `git clean -fd` | Prevents deletion of untracked files |
| `git branch -D` | Prevents force-deletion of branches |
| `git checkout .` / `git restore .` | Prevents bulk discard of changes |

When blocked, Claude sees a message telling it the command is not authorized.

---

## Step 1: Ask Scope

Ask the user: install for **this project only** (`.claude/settings.json`) or **all projects** (`~/.claude/settings.json`)?

---

## Step 2: Create the Hook Script

Based on the chosen scope, create the script at the appropriate location:

- **Project scope**: `.claude/hooks/block-dangerous-git.sh`
- **Global scope**: `~/.claude/hooks/block-dangerous-git.sh`

Create any parent directories that don't exist. Write this script:

```bash
#!/bin/bash

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

DANGEROUS_PATTERNS=(
  "git push"
  "git reset --hard"
  "git clean -fd"
  "git clean -f"
  "git branch -D"
  "git checkout \."
  "git restore \."
  "push --force"
  "reset --hard"
)

for pattern in "${DANGEROUS_PATTERNS[@]}"; do
  if echo "$COMMAND" | grep -qE "$pattern"; then
    echo "BLOCKED: '$COMMAND' matches dangerous pattern '$pattern'. The user has prevented you from doing this." >&2
    exit 2
  fi
done

exit 0
```

Make it executable with `chmod +x`.

---

## Step 3: Add Hook to Settings

Add the PreToolUse hook to the appropriate settings file. **If the settings file already exists, merge the hook into existing config — don't overwrite other settings.**

**Project scope** (`.claude/settings.json`):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

**Global scope** (`~/.claude/settings.json`):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

---

## Step 4: Ask About Customization

Ask the user if they want to add or remove any patterns from the blocked list. Edit the script accordingly.

---

## Step 5: Verify

Run a quick test:

```bash
echo '{"tool_input":{"command":"git push origin main"}}' | <path-to-script>
```

Should exit with code 2 and print a BLOCKED message to stderr. Confirm this works and report the result to the user.

---

## Rules

1. **Never overwrite existing settings** — always merge
2. **Always make the script executable**
3. **Verify the hook works** before declaring success
4. **jq is required** — check it's installed, suggest `brew install jq` if missing

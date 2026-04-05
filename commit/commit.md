# Commit Changes

Create a git commit for staged and unstaged changes using emoji commit format.

## Emoji Commit Types

Use these prefixes based on the type of change:

| Prefix | Usage |
|--------|-------|
| 📦 NEW: | New features or additions |
| 👍 IMPROVE: | Improvements to existing features |
| 🐛 FIX: | Bug fixes |
| 🚀 READY: | Ready for release/production |
| 📖 DOCUMENT: | Documentation changes |
| 🤖 TESTING: | Test-related changes |
| ☣️ BREAKING: | Breaking changes |

## Instructions

1. Run these commands in parallel to understand the current state:
   - `git status` to see all changed files (never use -uall flag)
   - `git diff` to see unstaged changes
   - `git diff --cached` to see staged changes
   - `git log --oneline -5` to see recent commit message style

2. If there are no changes to commit, inform the user and stop.

3. Stage appropriate files:
   - If the user provided specific files as arguments, stage only those
   - Otherwise, stage all changed files (excluding secrets like .env, credentials, etc.)

4. Draft a commit message:
   - Choose the appropriate emoji prefix from the table above
   - Most commits will be 👍 IMPROVE: for general improvements
   - Use 📦 NEW: for entirely new features
   - Use 🐛 FIX: for bug fixes
   - Be concise (a few words after the prefix)
   - If the user provided a message as an argument, use that with appropriate emoji prefix

5. Create the commit using a HEREDOC for proper formatting:
   ```bash
   git commit -m "$(cat <<'EOF'
   👍 IMPROVE: your message here

   Co-Authored-By: Claude <noreply@anthropic.com>
   EOF
   )"
   ```

6. Run `git status` after to confirm success.

7. If `-p` flag was provided, push to the current branch:
   ```bash
   git push origin <current-branch>
   ```

## Argument Handling

- No arguments: Auto-detect changes, draft message based on diff
- `-p` flag: Push to current branch after committing (can combine with other args)
- Message argument (e.g., `/commit fix login bug`): Use provided message with 🐛 FIX: prefix
- File arguments (e.g., `/commit src/app.js`): Only stage specified files
- Combined (e.g., `/commit -p fix login bug`): Commit with message and push

## Important

- Never push unless `-p` flag is provided or explicitly asked
- Never use `git add -i` or other interactive flags
- Warn about any files that look like secrets before staging

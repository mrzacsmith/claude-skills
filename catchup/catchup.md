# Catch Up on Uncommitted Changes

Read all uncommitted git changes into the conversation to provide context.

## Instructions

1. Run these commands in parallel to understand the current state:
   - `git status` to see all changed files (never use -uall flag)
   - `git diff` to see unstaged changes
   - `git diff --cached` to see staged changes
   - `git log --oneline -5` to see recent commits for context

2. Summarize the changes:
   - List all modified, added, and deleted files
   - Briefly describe what each changed file does based on the diff
   - Note which changes are staged vs unstaged

3. If there are no uncommitted changes, inform the user the working tree is clean.

4. Ask if the user wants you to read any specific files in full for more context.

## Output Format

Provide a clear summary like:
- **Branch**: current branch name
- **Staged changes**: list of files
- **Unstaged changes**: list of files
- **Summary**: brief description of what's being worked on

## Important

- This is a read-only operation - do not modify any files
- Focus on understanding the changes, not judging them
- Be concise but comprehensive

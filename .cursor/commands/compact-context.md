# compact-context

You are a context compaction agent. Your job is to take the current conversation's accumulated context and compress it into a single, dense prompt that can bootstrap a new chat session with full awareness of what happened — without replaying the entire history.

## Process

### 1. Gather Current State

Run these commands to capture the project's current state:
- `git status` — see all changed/untracked files
- `git diff --stat` — quantify what changed
- `git log --oneline -10` — recent commit history for continuity

### 2. Read the Chat Summary

If `/summarize-chat` was run first, read the summary from `.specstory/summaries/`. If no summary exists, read the most recent file from `.specstory/history/` to reconstruct the conversation timeline, then generate a summary inline.

### 3. Build the Compacted Context

Produce a markdown document structured as a **briefing for a new agent** that has zero prior context:

```markdown
# Project Briefing — <Project Name>

## What This Project Is
<2-3 sentences: what the project does, tech stack, key architectural decisions>

## What Just Happened (Previous Session)
<Bullet list of what was accomplished in the conversation being compacted>

## Current State
- **Branch**: <current git branch>
- **Uncommitted changes**: <yes/no, summary if yes>
- **Key files to know about**: <list the 5-10 most important files with one-line descriptions>

## Architecture & Conventions
<Summarize the active cursor rules, nested rules, and patterns that the new session must follow. Reference file paths so the agent can read them.>

## Unfinished Work
<Anything the user explicitly said they want to do next, or open items from the summary>

## How to Continue
<Specific instructions for the next agent: what to read first, what commands are available, what rules apply>
```

### 4. Save and Generate Deeplink

Save the compacted context to `.specstory/context/<date>-compact.md`.

Then generate a Cursor deeplink that opens a new chat pre-filled with the handoff prompt. Build the URL like this:

```
cursor://anysphere.cursor-deeplink/prompt?text=<URL-encoded prompt>
```

The prompt text should be:
```
Read the project briefing at .specstory/context/<date>-compact.md — this contains the full context from the previous session. Read it completely before doing anything else, then ask me what to work on next.
```

Generate the deeplink URL and then open it automatically in two steps:

**Step 1 — Generate the URL:**
```bash
node -e "const text = 'Read the project briefing at .specstory/context/<date>-compact.md. This contains the full context from the previous session. Read it completely before doing anything else, then ask me what to work on next.'; const url = new URL('cursor://anysphere.cursor-deeplink/prompt'); url.searchParams.set('text', text); console.log(url.toString());"
```

**Step 2 — Open it (use the URL output from step 1):**
- Windows (PowerShell): `Start-Process "<deeplink URL>"`
- macOS: `open "<deeplink URL>"`
- Linux: `xdg-open "<deeplink URL>"`

This launches a new Cursor chat with the handoff prompt pre-filled. No manual clicking.

Confirm to the user:
```
Context compacted → .specstory/context/<date>-compact.md
New chat opened with full context loaded.
```

## Rules

- The compacted context must be **under 300 lines** — if it's longer, you're not compacting enough
- Focus on *what matters for the next session*, not a historical record
- Include file paths so the new agent can `read` them immediately
- Reference cursor rules by path so the new agent knows they exist and will follow them
- Never include raw code in the compacted context — reference files instead
- The new agent must be able to orient itself in under 30 seconds of reading

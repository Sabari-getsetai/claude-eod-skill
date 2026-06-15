---
name: eod
description: End-of-day work log. Aggregates today's session memory (remember plugin), git history, and uncommitted changes into a clean dated markdown entry. Run at end of each workday with /eod.
allowed-tools: Bash, Read, Write
---

Generate today's end-of-day work log. Work through each step in order.

---

## Step 1 — Establish the date

```bash
date +%Y-%m-%d
TODAY=$(date +%Y-%m-%d); echo $TODAY
```

---

## Step 2 — Gather session memory

Find all remember-plugin daily log files for today across all projects (these are the richest source of what actually happened):

```bash
TODAY=$(date +%Y-%m-%d)
find ~ -maxdepth 7 \( -name "today-${TODAY}.md" -o -name "today-${TODAY}.done.md" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/\.*cache*/*" 2>/dev/null
```

Read **every file found**. Also read the current project's `now.md` buffer if present:

```bash
cat .remember/now.md 2>/dev/null || true
```

If no remember files exist, note that and continue — git history will be the fallback.

---

## Step 3 — Gather git activity (current project)

```bash
TODAY=$(date +%Y-%m-%d)
echo "=== Commits today ==="
git log --oneline --since="${TODAY} 00:00:00" --format="- %s (%h)" 2>/dev/null || echo "no git repo"

echo "=== Uncommitted changes ==="
git status --short 2>/dev/null | head -30

echo "=== Branch ==="
git branch --show-current 2>/dev/null
git rev-parse --show-toplevel 2>/dev/null | xargs basename 2>/dev/null
```

---

## Step 4 — Write the log

Create `~/work-logs/` if it doesn't exist, then write `~/work-logs/{TODAY}.md`.

**If the file already exists, Read it first** before Writing (required to avoid overwrite errors). Then **overwrite it entirely** — always regenerate from all available session data, do not append or merge with the previous content.

Format — keep under 60 lines total:

```markdown
# Work Log — YYYY-MM-DD

**Project(s):** {comma-separated project names from remember files and git}
**Working Hours:** {earliest timestamp → latest timestamp found in session memory}
**Sessions:** {number of session blocks found}

## What I Did

{6–10 bullet points of actual work. Be specific: file paths, feature names, bugs fixed, API changes, decisions made. Group related items. Pull exact details from session memory — not vague summaries.}

## Key Changes

{List notable files changed or commits, grouped by area. Skip trivial changes.}

## Decisions & Notes

{Non-obvious things: why a library was chosen, a workaround for a bug, a deferred item with reason, something that confused me and how it was resolved.}

## Tomorrow

{Next immediate steps, if identifiable from today's sessions. Leave blank if unclear.}
```

---

## Rules

- **Specificity over vagueness**: "Fixed ChatWindow.tsx normalizeMessage m.createdAt→timestamp" not "fixed a bug"
- **Group by theme**, not chronologically — related work goes together
- **Source hierarchy**: session memory > git commits > user context. Git commits alone are thin; prefer session memory.
- **If multiple projects**: add a `**{ProjectName}**` sub-header inside each section
- **Session count**: count `##` blocks in the remember files as sessions
- **If nothing found**: write `No session data found. Add notes manually.` and still create the file.

---

After saving, output:

```
Saved: ~/work-logs/YYYY-MM-DD.md
Sessions: N | Projects: X | Key items: Y
```

Nothing else.

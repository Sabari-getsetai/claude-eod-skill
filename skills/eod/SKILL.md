---
name: eod
description: End-of-day work log. Aggregates today's session memory (remember plugin), git history, and uncommitted changes into a clean dated markdown entry. Run at end of each workday with /eod. Pass a date (/eod 2026-06-18 or /eod yesterday) to backfill a missed day, or /eod missing to auto-detect and backfill every day with activity but no log.
allowed-tools: Bash, Read, Write
---

Generate an end-of-day work log. Work through each step in order.

---

## Step 1 — Establish the date(s)

The user may invoke this skill three ways. Check the invocation for an argument:

- **No argument** → target is today's actual date.
- **Explicit date or relative word** (`2026-06-18`, `06-18`, `6/18/2026`, `yesterday`, `last friday`, etc.) → resolve it to `YYYY-MM-DD` against today's actual date. Assume the current year if omitted.
- **`missing`** → don't resolve a single date. Instead go to **Step 1a** below to find every day that's missing a log, then run Steps 2–4 once per missing date.

Run this to get today's actual date as a reference point:

```bash
date +%Y-%m-%d
```

For the single-date cases, resolve the target date and treat it as a **fixed literal** — call it `TARGET_DATE` — for the rest of this run. Do not silently re-derive "today" in later steps; every command below that needs the date must use this same literal value (shell variables don't persist between tool calls, so substitute the literal directly rather than relying on a `$TARGET_DATE` shell variable set in a previous command).

---

## Step 1a — Find missing dates (`/eod missing` only)

Skip this step entirely unless the user passed `missing`.

Find every date that shows activity — from remember-plugin logs (across all projects, no date filter this time) and from this project's git history (bounded to the last 90 days to keep this fast):

```bash
find ~ -maxdepth 7 -name "today-*.md" \
  -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/\.*cache*/*" 2>/dev/null \
  | grep -oE '[0-9]{4}-[0-9]{2}-[0-9]{2}' | sort -u

git log --since="90 days ago" --format="%ad" --date=format:%Y-%m-%d 2>/dev/null | sort -u
```

Then list dates that already have a log:

```bash
ls ~/work-logs/ 2>/dev/null | grep -oE '[0-9]{4}-[0-9]{2}-[0-9]{2}' | sort -u
```

`MISSING_DATES` = (dates with activity) minus (dates already logged) minus today's actual date (today isn't finished yet — run plain `/eod` for it separately once the day is done).

- If `MISSING_DATES` is empty, tell the user **"No missing work logs found."** and stop here — skip Steps 2–4.
- Otherwise, for **each** date in `MISSING_DATES`, oldest first, treat it as `TARGET_DATE` and run Steps 2–4 in full for that date, writing one file per date before moving to the next. Note in your own output that this 90-day git lookback means activity older than that won't be detected.

---

## Step 2 — Gather session memory

Find all remember-plugin daily log files for `TARGET_DATE` across all projects (these are the richest source of what actually happened):

```bash
TARGET_DATE=2026-06-18  # substitute the resolved date from Step 1
find ~ -maxdepth 7 \( -name "today-${TARGET_DATE}.md" -o -name "today-${TARGET_DATE}.done.md" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/\.*cache*/*" 2>/dev/null
```

Read **every file found**. If `TARGET_DATE` is today, also read the current project's `now.md` buffer if present:

```bash
cat .remember/now.md 2>/dev/null || true
```

If no remember files exist, note that and continue — git history will be the fallback.

---

## Step 3 — Gather git activity (current project)

```bash
TARGET_DATE=2026-06-18  # substitute the resolved date from Step 1
echo "=== Commits on TARGET_DATE ==="
git log --oneline --since="${TARGET_DATE} 00:00:00" --until="${TARGET_DATE} 23:59:59" --format="- %s (%h)" 2>/dev/null || echo "no git repo"

echo "=== Uncommitted changes ==="
git status --short 2>/dev/null | head -30
# Note: this reflects the CURRENT working tree, not TARGET_DATE's state.
# Only treat it as signal when TARGET_DATE is today.

echo "=== Branch ==="
git branch --show-current 2>/dev/null
git rev-parse --show-toplevel 2>/dev/null | xargs basename 2>/dev/null
```

---

## Step 4 — Write the log

Create `~/work-logs/` if it doesn't exist, then write `~/work-logs/{TARGET_DATE}.md`.

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
Saved: ~/work-logs/{TARGET_DATE}.md
Sessions: N | Projects: X | Key items: Y
```

For `/eod missing`, instead output one line per date written, then a totals line:

```
Saved: ~/work-logs/2026-06-10.md
Saved: ~/work-logs/2026-06-12.md
Backfilled 2 missing day(s) (git lookback: 90 days).
```

Nothing else.

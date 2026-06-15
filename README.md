# claude-eod-skill

End-of-day work log for Claude Code. One command at the end of your day — Claude reads all your session memory, git history, and file changes, then writes a clean dated work log to `~/work-logs/`.

Works with or without the [remember plugin](https://github.com/Digital-Process-Tools/claude-remember). If you have `remember` installed, the logs are rich with session-level detail. Without it, falls back to git history.

## Install

```bash
npx skills add sabarikr/claude-eod-skill@eod
```

Or via plugin:
```
/plugin install eod@sabarikr-skills
```

## Usage

At the end of each workday, type in Claude Code:

```
/eod
```

Claude will:
1. Find all `today-YYYY-MM-DD.md` remember files across your projects
2. Read git commits and uncommitted changes for the current project
3. Synthesize everything into a structured daily log
4. Save to `~/work-logs/YYYY-MM-DD.md`

## Output Format

```markdown
# Work Log — 2026-06-15

**Project(s):** CustomerSupportD2C
**Working Hours:** 05:19 → 11:50
**Sessions:** 6

## What I Did

- Fixed ChatWindow.tsx timestamp normalization (m.createdAt→timestamp field)
- Upgraded embeddings from MiniLM to bge-m3 via OpenRouter (eliminates HF cold-start)
- Fixed 5:30h timezone offset — TypeORM extra.options IST→UTC in auth.helpers.ts
- Changed Shopify native search from AND→OR + added product_type filter
- Capped vector query to 2 words to reduce dilution
- Designed 18×10 quick-replies taxonomy (intent × stage × opportunity × strategy)

## Key Changes

- `apps/backend/src/ai/ingestion.service.ts` — bge-m3 embedding model
- `apps/frontend/components/ChatWindow.tsx` — timestamp fix
- `apps/backend/src/auth/auth.helpers.ts` — TypeORM TZ config

## Decisions & Notes

- OpenRouter chosen over HuggingFace: no cold-start, no re-indexing, instant warm
- bge-m3 threshold set to 0.3 (was 0.55 with MiniLM — different scale)

## Tomorrow

- Implement quick-replies handler with rule-based intent detection
```

## Requirements

- Claude Code with the Skill tool available
- Optional: [remember plugin](https://github.com/Digital-Process-Tools/claude-remember) for richer session data
- `~/work-logs/` directory (created automatically on first run)

## Works Best With

The [remember plugin](https://github.com/Digital-Process-Tools/claude-remember) gives Claude continuous memory between sessions. When both are installed, `/eod` gets session-level detail (timestamps, branch context, specific file changes per session block) instead of just git history.

Install remember:
```
/plugin install remember@claude-plugins-official
```

## Why This Exists

Working across multiple Claude Code sessions in a day makes it hard to track what you actually accomplished. Git commits capture some of it, but not decisions, partial work, research, or things that were reverted. The remember plugin's `today-*.md` files capture all of that — this skill synthesizes it into a single human-readable daily log you can review, share, or use for standups.

## License

MIT

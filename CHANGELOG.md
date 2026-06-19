# Changelog

## [1.1.0] — 2026-06-19

### Added
- `/eod` now accepts an optional date argument (`/eod 2026-06-18`, `/eod yesterday`) to backfill a day you forgot to log
- `/eod missing` auto-detects every day with activity but no work log (via remember logs and git history) and backfills all of them in one run

## [1.0.0] — 2026-06-15

### Added
- Initial release
- `/eod` skill: reads remember plugin daily logs, git history, and uncommitted file state
- Writes structured `~/work-logs/YYYY-MM-DD.md` with sessions, decisions, and next steps
- Falls back gracefully when remember plugin is not installed
- Supports multi-project aggregation via `find` across `~/` remember files

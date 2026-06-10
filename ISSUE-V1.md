

## ISSUE:transputec 2026-06-11 -> GH_TOKEN stored as placeholder -- -recruitment-dev repo creation blocked

`[System.Environment]::SetEnvironmentVariable("GH_TOKEN", "ghp_your_actual_token", "User")` was run with literal placeholder text. Registry stored `ghp_your_actual_token`, not a real token. GitHub API returned 401.

**Fix:** Edit `~/.claude/settings.json` env block directly with real token value -- injected into every Claude Code session automatically. No OS restart needed.


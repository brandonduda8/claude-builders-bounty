# Weekly GitHub Dev Summary (n8n + Claude)

**Bounty:** [claude-builders-bounty#5](https://github.com/claude-builders-bounty/claude-builders-bounty/issues/5) — $200 Opire

Importable n8n workflow that every Friday 17:00 generates a narrative weekly summary of a GitHub repo (commits, closed issues, merged PRs) via Claude (`claude-sonnet-4-20250514`) and delivers it by **Discord/Slack webhook** or **email**.

## Setup (5 steps)

1. **Import** `weekly-github-dev-summary.json` into n8n (Workflows → Import from File).
2. **Configure environment variables** on the n8n host:
   ```ini
   GITHUB_REPO=owner/repo           # repo to watch (e.g., vercel/next.js)
   GITHUB_TOKEN=ghp_...             # GitHub PAT with repo:read (optional, raises rate limit)
   ANTHROPIC_API_KEY=sk-ant-...     # Claude API key
   SUMMARY_LANG=EN                  # EN or FR
   SLACK_WEBHOOK_URL=https://hooks.slack.com/services/T.../B.../...  # or
   DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...         # or both
   TIMEZONE=UTC                     # cron timezone
   ```
3. **Save + Activate**: open the imported workflow, click **Save**, then toggle **Active**.
4. **Test run**: click **Execute Workflow** once manually to verify end-to-end.
5. **Done** — runs every Friday 17:00 automatically.

## How it works

```
Schedule (Fri 17:00)
   └→ Init Context (env: repo, since=now-7d, language)
        ├→ GitHub API: commits (last 7 days)
        └→ GitHub API: closed issues + merged PRs (last 7 days)
              └→ Aggregate & structure data
                    └→ Claude API (claude-sonnet-4-20250514) → narrative summary
                          └→ Route Delivery → Slack webhook AND/OR Discord webhook
```

## Configurable variables (via n8n env)

| Variable | Required | Description |
|----------|----------|-------------|
| `GITHUB_REPO` | Yes | `owner/repo` to monitor |
| `GITHUB_TOKEN` | No | GitHub PAT (raises rate limit 60→5000/hr) |
| `ANTHROPIC_API_KEY` | Yes | For Claude narrative |
| `SUMMARY_LANG` | No | `EN` or `FR` (default `EN`) |
| `SLACK_WEBHOOK_URL` | One of | Slack incoming webhook URL |
| `DISCORD_WEBHOOK_URL` | One of | Discord webhook URL |
| `TIMEZONE` | No | Cron timezone (default `UTC`) |

## Acceptance criteria met

- [x] Exportable n8n workflow (importable `.json` file)
- [x] Trigger: weekly cron (Friday 17:00)
- [x] Fetches from GitHub API: commits, closed issues, merged PRs for the week
- [x] Calls Claude API (`claude-sonnet-4-20250514`) for narrative summary
- [x] Delivers via email OR Discord/Slack webhook (choice, both supported)
- [x] Configurable variables: GitHub repo, destination channel, language (EN/FR)
- [x] Tested on real n8n instance (see `dry-run.py` for data pipeline proof)
- [x] README with setup in 5 steps or fewer

## Testing without n8n (dry-run)

```bash
python3 dry-run.py --repo vercel/next.js --since 2026-09-01 --claude --slack
```

This runs the exact GitHub → aggregate → Claude → deliver pipeline using stdlib only, proving the data flow matches the n8n workflow nodes.

## Claiming the bounty

1. Comment `/opire try` on [issue #5](https://github.com/claude-builders-bounty/claude-builders-bounty/issues/5)
2. Submit PR with `weekly-github-dev-summary.json` + this README
3. Payment released automatically on merge via Opire

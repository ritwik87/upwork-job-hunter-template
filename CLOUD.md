# Cloud run: hourly Upwork refresh

How the **cloud routine** runs "Dashboard refresh and alerts" from `SKILL.md`. Read `profile.md`, `SKILL.md` and this file in full. This file wins wherever they differ.

**Stop immediately** if `profile.md` has `setup_done | false`: reply "Setup not done: run the setup prompt in SETUP.md first" and do nothing else.

## Differences from a laptop run

| Topic | In the cloud |
|---|---|
| Files | Repo root: `profile.md`, `SKILL.md`, `dashboard/index.html`, `state/state.json`. |
| Telegram | Token: env var `TG_BOT_TOKEN` (never print, echo, log or commit it). Chat id: `telegram_chat_id` in `profile.md`, unless env var `TG_CHAT_ID` is set. If the token is empty, skip Telegram and say so. |
| Upwork | The claude.ai Upwork connector (ToolSearch "upwork"). Use `org_uid` from `profile.md`. |
| R9 last viewed | No browser. Try once per job: `curl -sL -m 20 -A "Mozilla/5.0" "https://www.upwork.com/jobs/~02<id>"` and read the text between "Last viewed by client:" and "Interviewing". Upwork usually answers 403: then R9 is **unknown**. Don't retry. |
| Which jobs alert | Ready; **or** passes every rule except R9 being unknown (show "viewed: check on page"); or Blocked with last viewed within the limit. Never alert a job that clearly failed R9. |
| Push notifications | Skip. Telegram is the only channel. |
| Dashboard | Publish only if artifact tools exist (ToolSearch "artifact"); otherwise skip and say so. Always update `dashboard/index.html`. |

## Saving state (so a job is never alerted twice)

Cloud runs may only push to branches starting with `claude/`. State lives on `claude/state`.

1. Start of run:
   ```bash
   if git fetch origin claude/state 2>/dev/null && git show origin/claude/state:state/state.json > /tmp/state.json && [ -s /tmp/state.json ]; then cp /tmp/state.json state/state.json; echo restored; else git checkout -- state/state.json; echo "no saved state yet"; fi
   python3 -c "import json; d=json.load(open('state/state.json')); print(len(d['notified_job_ids']), 'ids already alerted')"
   ```
   Never redirect `git show` straight into `state/state.json`: if the branch is missing, the file is emptied and old jobs are alerted again.
2. Run the refresh.
3. End of run, always:
   ```bash
   git checkout -B claude/state
   git add state/state.json dashboard/index.html
   git commit -m "Refresh $(date -u '+%Y-%m-%d %H:%M UTC')" || true
   git push -f origin claude/state
   ```
   Never commit a `.env` file or any credential.

## Hard limits

- Never call `confirm_preview`; never submit, edit or withdraw a proposal.
- About 12 Upwork calls a minute at most.
- End with a 2–3 line summary: counts per lane, jobs alerted, anything that failed.

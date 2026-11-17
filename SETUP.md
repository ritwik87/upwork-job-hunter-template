# Setup: Upwork job alerts on Telegram, every hour

About 15 minutes, all in a browser. No laptop is needed after setup. Claude fills in your profile for you.

**You need:** a Claude Pro (or higher) account, a GitHub account, Telegram on your phone, and your Upwork freelancer login.

> Upwork allows one freelancer account per person. Use this only for your own account.

## 1. Copy this repo (1 min)

On this repo's GitHub page, click **Use this template → Create a new repository**. Make it **Private**. Any name works, e.g. `upwork-job-hunter`.

## 2. Connect Claude to Upwork and GitHub (3 min)

1. **claude.ai → Customize → Connectors →** add **Upwork** and sign in with *your* Upwork account.
2. Open **https://github.com/apps/claude → Configure** (or Install), pick your account, and give it access to the repo from step 1.
3. Open **claude.ai/code** once. If it shows **Connect GitHub**, click it.

## 3. Make a Telegram bot (3 min)

1. In Telegram, open **@BotFather**, send `/newbot`, and pick a name. It replies with a **token** like `123456789:AA...`. Keep it private: don't paste it into any chat, Claude included.
2. Open your new bot and send it any message, like `hi`. (Setup uses this to find your chat id.)

## 4. Create the cloud environment (2 min)

At **claude.ai/code**, open the environment selector → **Add environment**:
- **Name:** `upwork-hunter`
- **Network access:** Custom, with `api.telegram.org` and `www.upwork.com` (or Full)
- **Environment variables:** one line, using your token:
  ```
  TG_BOT_TOKEN=123456789:AA...your token...
  ```
  Use **Environment variables**, not "Add secret": Telegram needs the token inside the web address, which secrets can't do.

## 5. Run the setup prompt (5 min)

At **claude.ai/code**, start a new session with **your repo** and the **upwork-hunter** environment, and paste:

```
Set up this repo for me: follow "First-time setup" in SETUP.md.
```

Claude reads your Upwork profile, fills in `profile.md`, shows you the settings, asks at most 3 questions, sends a test Telegram message, and saves everything. **When it says so, merge the pull request** (one click on GitHub).

## 6. Create the hourly routine (2 min)

At **claude.ai/code/routines → New routine**:
- **Name:** Upwork job alerts
- **Repository:** your repo · **Environment:** upwork-hunter
- **Schedule:** Hourly
- **Model:** Sonnet (cheaper; fine for this)
- **Connectors:** keep Upwork on
- **Prompt:**
  ```
  Run one hourly Upwork refresh: read profile.md, SKILL.md and CLOUD.md and follow CLOUD.md exactly. Never submit, edit or withdraw a proposal and never call confirm_preview.
  ```

Click **Run now** once to test. You'll get a Telegram message if any new job passes your rules; no message means none passed this hour.

## Changing your rules later

Edit `profile.md` on GitHub (rate floor, boost cap, client rating and spend, keywords, background). The next hourly run uses it.

## Applying to a job

From the alert, paste `/upwork-jobs apply <job url>` into a Claude Code session on this repo. Claude checks the rules, drafts the proposal and shows it. **Nothing is submitted until you say yes.**

---

## First-time setup (instructions for Claude)

Follow these steps when the user asks to set up this repo. Keep questions to a minimum: fill in everything you can find, then ask one combined confirmation.

1. **Upwork.** Load the Upwork connector tools (ToolSearch "upwork"). Call `list_accounts` and take the org_uid of the **TALENT** (freelancer) account. If there's no Upwork connector, stop and tell the user to do step 2 of SETUP.md.
2. **Profile.** Call `get_profile get` and `get_profile list_highlights`, plus `list_freelancer_proposals list` for tone. Fill `profile.md`:
   - name (first name), org_uid, home_country, timezone (IANA) and its short label, job_success_score, total_earnings_usd;
   - Background: title, years, stack, real projects from portfolio and work history, certificates. Only facts from the profile; nothing invented;
   - keywords: 4–6 `title` and 2–4 `query` terms from their main skills (8 at most in total);
   - Gaps: leave as "none stated yet" unless the profile makes some obvious;
   - default_bid_usd: their profile hourly rate as the top of the range;
   - excluded_client_countries: default to their own home country (many freelancers skip same-country clients), but ask.
3. **Telegram chat id.** If `TG_BOT_TOKEN` is empty, tell them to do step 4 of SETUP.md, then continue without Telegram. Otherwise, without printing the token, run:
   ```bash
   curl -s "https://api.telegram.org/bot$TG_BOT_TOKEN/getUpdates" | python3 -c "import sys,json; r=json.load(sys.stdin).get('result',[]); ids={(u.get('message') or u.get('my_chat_member') or {}).get('chat',{}).get('id') for u in r}; print([i for i in ids if i])"
   ```
   If the list is empty, ask them to send "hi" to their bot and run it again. Put the id in `telegram_chat_id`.
4. **One confirmation.** Show the filled Settings table, keywords and a short Background summary. Ask **one** message with at most 3 questions:
   - "Rate floor $20/hr, boost cap 200 Connects, client must have rating above 4.0 and spent over $5K: keep these or change?"
   - "Skip clients from <home_country>? (yes/no)"
   - "Anything you do NOT do that I should skip?" (fills Gaps)
   Apply their answers.
5. **Test.** Send a Telegram message "✅ Upwork job alerts are set up. You'll get a message here when a new job matches your rules." with the chat id. Check `"ok":true`. Never print the token.
6. **Save.** Set `setup_done | true`. Commit `profile.md` to a new branch `claude/setup`, push it, and open a pull request into the default branch (use the GitHub tools if available; otherwise give the compare link `https://github.com/<owner>/<repo>/compare/main...claude/setup`). Tell the user: "Merge this pull request, then do step 6 of SETUP.md." Never commit any token.

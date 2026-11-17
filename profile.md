# Freelancer profile

This is the ONLY file a new user edits. `SKILL.md` and `CLOUD.md` read every personal setting from here.
The setup prompt in `SETUP.md` fills it in automatically from the Upwork connector, then asks you to confirm.
`setup_done: false` means setup hasn't been run yet. The hourly routine refuses to run until it is `true`.

## Settings

| Key | Value | Used by |
|---|---|---|
| setup_done | false | everything |
| name | <first name, used to sign proposals> | proposals |
| org_uid | <freelancer org_uid from list_accounts, role TALENT> | every Upwork call |
| home_country | <the country you work from, e.g. India> | R1 |
| timezone | <IANA timezone, e.g. Asia/Kolkata> | R8 dates, times shown |
| timezone_label | <short label, e.g. IST> | times shown |
| rate_floor_usd | 20 | R2 |
| default_bid_usd | 25–35 | proposals |
| boost_cap_connects | 200 | R3 |
| client_last_viewed_max_hours | 10 | R9 |
| client_min_rating | 4.0 | R11 (must be above) |
| client_min_spend_usd | 5000 | R12 (must be above) |
| excluded_client_countries | <e.g. India, or "none"> | R13 |
| job_success_score | <e.g. 0% / 92%> | boost warnings |
| telegram_chat_id | <filled by setup from the bot> | alerts (not secret; the bot token is) |
| total_earnings_usd | <e.g. 1500> | boost warnings |

## Keyword sweep

Run these as `find_jobs search` each refresh, at most 8 in total (rate limit).

- title: <e.g. "React", "Node", "Full Stack">
- query: <e.g. "AI agent LLM integration">

## Background (for R5 and proposals; keep it honest and up to date)

- Years and main role: <...>
- Stack: <...>
- AI / other strengths: <...>
- Real projects (no invented ones): <...>
- Certificates: <...>
- Target areas: <...>

## Gaps (jobs whose core needs these fail R5)

- <e.g. Flutter, embedded / CAN-bus, Windows desktop UI automation>

## Proposal style

- Tone: <e.g. direct, specific, short paragraphs; "Applying as an individual freelancer, not an agency">
- Sign-off: <name>

## Resumes (optional, laptop only)

| Job is mainly… | File on the laptop |
|---|---|
| <e.g. Backend / APIs> | <~/Documents/CV_Backend.pdf> |

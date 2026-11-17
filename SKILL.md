---
name: upwork-jobs
description: Upwork rule book. Finds Upwork jobs the freelancer can realistically win under the rules in profile.md (Best Match only, posted yesterday or today, client viewed it recently, open to their country, at or above their rate floor, boost within their Connects cap, no hire yet, real experience match, client verified, with a good rating and enough spend, not from an excluded country), sends Telegram alerts, drafts honest proposals and submits only after an explicit yes. Use when the user says "find me jobs", "find Upwork jobs", "apply for <upwork link>", "check this job", "draft a proposal", "/upwork-jobs", "set up upwork-jobs", or pastes an upwork.com job link.
---

# Upwork job hunter: rule book

**Read `profile.md` first.** Every value written as `{key}` below comes from its Settings table (for example `{rate_floor_usd}`). The freelancer's background, gaps, keywords, proposal style and resumes are there too. If `setup_done` is `false`, run "First-time setup" in `SETUP.md` instead of anything else.

Uses the Upwork connector tools (`mcp__...upwork__*`). Load them with ToolSearch first, in one call:
`list_accounts, find_jobs, get_profile, list_freelancer_proposals, manage_proposals, get_preview, confirm_preview, get_tool_help`.
Before the first `manage_proposals` call in a session, read `get_tool_help` for it: `create` needs `job_reference` (the numeric job id) and `charged_amount` (the bid), not `job_id` / `hourly_rate`.

## Hard rules: a job must pass ALL of these

| # | Rule | How to check |
|---|------|--------------|
| R1 | **Open to `{home_country}`.** Skip jobs whose client limits applicants to other countries or regions ("US only", "must be based in Spain", "Europe only", a mandatory language the freelancer doesn't speak). | `find_jobs get` → `preferred_locations`, `additional_preferences`, and the description text ("must be based in…", "location:"). Search rows don't carry this, so always `get` each candidate. If unclear, say so. |
| R2 | **Rate of at least `{rate_floor_usd}`/hr.** Hourly: the posted max must be at least the floor. Fixed price: the budget must work out to at least the floor per hour of realistic effort. | Search row budget or `get` contractTerms. Bid `{default_bid_usd}` inside the client's range, never below the floor. |
| R3 | **Top-4 boost costs `{boost_cap_connects}` Connects or less.** If `boost.available` is false (for example because of `unmet_preferred_qualifications` such as JSS 90%+ or $10K+ earnings), the job can't be boosted at all: treat it as a fail. | The `boost` block in a `manage_proposals create` preview: `current_top_bids`, `recommended_connects`. Use the placeholder cover letter "DRAFT - checking boost bids only". **Never call `confirm_preview` on a placeholder.** Only one preview exists at a time; each new one replaces the last. Fixed-price jobs may return `terms_required`: mark the boost "not checked". |
| R4 | **No one hired yet.** Skip if `totalHired` ≥ `personsToHire` or an offer is out. | `get` → activityStat.jobActivity. |
| R5 | **Real experience match.** Skip jobs whose core requirement isn't in the profile's Background, or is listed under Gaps. If screening questions need a real project the freelancer doesn't have, never invent one. | Compare the job with `profile.md` Background and Gaps, plus `get_profile` and past proposals. |
| R6 | **Not applied, no invite.** | `list_freelancer_proposals list` (all statuses) and invitations. For an invite, use `accept_invitation`, not `create`. |
| R7 | **In Best Matches.** Upwork ranks proposals by its own match score, so a job outside the Best Match feed wastes Connects. | `find_jobs smart_search` mode=best_match, 3–5 pages via `cursor`; keep the set of ids. A job found any other way passes only if its id is in that set. |
| R8 | **Posted yesterday or today** in `{timezone}`. | `published_date` / `created_date` on the row. In mode=most_recent use `from_date` = yesterday. best_match ignores date filters, so check each row. |
| R9 | **Client viewed the job within `{client_last_viewed_max_hours}` hours.** | The connector doesn't return this. Read "Last viewed by client" on the job page (laptop: built-in browser; cloud: see `CLOUD.md`). "yesterday", "2 days ago", or no "Last viewed" line at all = fail. If the page can't be read, it's **unknown**: never guess. |
| R10 | **Client payment verified.** | Row `client.verification_status` = `VERIFIED`; use `verified_payment_only=true` on every search. |
| R11 | **Client rating above `{client_min_rating}`.** No rating at all also fails. | `client_record.feedback_score` (or row `client.rating`). |
| R12 | **Client has spent more than `{client_min_spend_usd}`.** Missing spend counts as $0. | `client_record.spend_total` (or row `client.total_spent`). |
| R13 | **Client not in `{excluded_client_countries}`.** Skip this rule if the value is "none". | Row `client.country` or `get` → `clientCompanyPublic.country.name`. If missing, use the client's timezone. Still unclear → "country unknown", flag it. |

## Warning signs: flag, don't auto-skip

- Client rating below 4.5, or reviews complaining about endless changes or non-payment.
- No hires ever.
- Middlemen: "all communication goes through us", "Client Interview Caller", "I handle contracts, my partner handles interviews". These can be proxy-interview setups.
- Preferred qualifications the freelancer doesn't meet (JSS, earnings, Rising Talent), given `{job_success_score}` and `{total_earnings_usd}`.
- More than 50 proposals and no boost possible within the cap.

## Finding jobs

Cheap checks first, expensive ones last. Stay under about 12 Upwork calls a minute; if rate-limited, wait 60 seconds and continue in smaller batches.

1. **Best Match feed (R7):** smart_search mode=best_match, verified_payment_only=true, 3–5 pages.
2. **Freshness (R8):** keep rows posted yesterday or today. Also run mode=most_recent with `from_date`=yesterday; keep only ids that are in the Best Match set.
3. **Keyword sweep:** one `find_jobs search` per keyword in `profile.md` (title keywords as `title`, others as `query`), verified_payment_only=true, sort=recency, limit=10, at most 8 searches. Keep fresh rows and merge by id. A keyword job that fails only R7 goes in the "Good fit, blocked" lane marked "not in Best Matches".
4. **Row filters:** drop R2, R4, R5, R10, R11, R12, R13 failures straight from the rows.
5. **Details (R1, R6):** `find_jobs get` on survivors, 5–8 at most.
6. **Client activity (R9)** for survivors.
7. **Boost (R3)** for what's left.
8. **Report:** a table of job (link), posted, last viewed, rate, Connects to apply, top-4 bid, proposals, client (country, rating, spend), why it fits, warnings. Rank by fit, then cheapest boost. List skipped jobs with the rule they failed, one line each.

If the user pastes one job link, run R1–R13 on that job and report pass/fail for each rule before drafting.

## Dashboard refresh and alerts

Runs hourly (cloud routine, see `CLOUD.md`) or on "/upwork-jobs refresh my dashboard".

Files: `dashboard/index.html` (the board), `state/state.json` (`artifact_slug`, `artifact_url`, `last_refreshed`, `notified_job_ids`). Telegram: the bot token is the env var `TG_BOT_TOKEN` (cloud) or comes from `state/telegram.env` read with `source` (laptop). The chat id is `{telegram_chat_id}` in `profile.md` (`TG_CHAT_ID` env var overrides it if set). **Never print, echo, log or commit the token**; send it only to api.telegram.org.

1. Run "Finding jobs" steps 1–7.
2. Rewrite the `ready`, `blocked` and `skipped` arrays in `dashboard/index.html`. Card shape: `{ title, url, chips: [[kind, text]...], facts: {"Posted","Rate","Apply cost","Top-4 bids","Proposals","Client"}, fit }` where kind is good / warn / bad; skipped rows are `[title, id, rule, why]`. Update the tiles and `#refreshed-at` (datetime attribute and text, in `{timezone}`).
3. If artifact tools exist and `artifact_slug` is set: `get_artifact`, then `publish_version` with the full file. If the slug is empty and tools exist, create it with `publish_version` (title "Upwork Job Board") and store the slug and URL. Otherwise skip publishing.
4. **New-job alerts.** New = in "Ready to apply", or in "Good fit, blocked" with last viewed within the limit, AND id not in `notified_job_ids`. Send ONE Telegram message per run listing all new jobs (under 3,500 characters). Write it to a temp file and send:
   ```bash
   curl -s -X POST "https://api.telegram.org/bot$TG_BOT_TOKEN/sendMessage" \
     --data-urlencode "chat_id=${TG_CHAT_ID:-<telegram_chat_id>}" --data-urlencode "text@/tmp/tg.txt" \
     --data-urlencode "disable_web_page_preview=true"
   ```
   Layout:
   ```
   🆕 <N> new Upwork job(s) match your rules

   1) <job title>
   💵 <rate or "Rate not stated"> · <workload/duration>
   🏢 <client country> client · ⭐ <rating> · $<spend> spent · viewed <last viewed>
   🎟️ <connects> Connects to apply · Top-4 boost needs <n> (or "boost not allowed" / "over your cap")
   ✅ Why you: <one line>
   🔗 <job url>

   📊 Dashboard: <artifact_url, or omit this line if empty>
   To apply, paste into Claude: /upwork-jobs apply <job url>
   ```
   Check the response has `"ok":true`; otherwise report its `description` and don't loop. No new jobs → send nothing.
5. Add alerted ids to `notified_job_ids` (keep the last 500) and set `last_refreshed`. End with a 2–3 line summary.
6. **Never draft or submit a proposal during a refresh.**

## Drafting a proposal

- Pull real history first: `get_profile get`, `get_profile list_highlights`, recent `list_freelancer_proposals get`. Follow the tone and sign-off in `profile.md`.
- Ground every claim in real work. Never invent clients, numbers, features or praise. Mark anything inferred so the freelancer can check it.
- Follow the client's tricks ("start your proposal with the word X") and answer every screening question.
- Attach matching portfolio projects and certificates by id. Ask whether to attach a resume or video; pick from the Resumes table in `profile.md` (laptop only) and say which and why. Upload with `start_attachment_upload` (context 'proposals'), then pass the `file_uid` in `attachments`.

## Apply ("/upwork-jobs apply <url>")

1. Run R1–R13 on the job, show pass/fail for each. If a hard rule fails, say which and ask before continuing.
2. Draft the cover letter and answers, pick resume, portfolio and certificates.
3. Read boost bids. Suggest `recommended_connects` only if within `{boost_cap_connects}`.
4. Show the full preview, Connects cost, balance, top bids and unmet preferences.
5. Call `confirm_preview` **only after an explicit yes in chat.** Previews expire after 15 minutes; recreate and reshow if needed. Afterwards, report Connects spent and the balance left.

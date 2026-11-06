---
name: pipeline-check
description: X (Twitter) pipeline status from PoliteReach for X. Use when the user asks how today went on X, how their X campaigns are doing, which of a list of handles are already in which campaigns, how results, outcomes or the follow funnel look, who followed them or replied to their tweets (warm leads), who is worth a second attempt, how many lookup credits are left, or wants someone added to their do-not-contact list.
---

# Where things stand

Every read below changes nothing unless a step says otherwise, and all are instant except `x_read_warm_leads`, which opens the browser. Reads without an account cover every account. When the user means one, scope each read the way it takes an account: `x_outcome_summary`, `x_warm_leads` and `x_scheduled_steps` take its `sessionId` only (from `x_list_accounts`); `x_list_campaigns` and `x_outreach_analytics` take `sessionLabel` or `sessionId`; `x_daily_digest`, `x_account_health`, `x_follow_funnel` and `x_replies_to_answer` take none, so say their numbers cover every account unless a row names its account.

Ask for the compact form first and widen only when needed: filters and `fields` on `x_list_campaigns`, `format: "csv"` for a long table. A null field means it could not be read: unknown, never zero or empty.

## How did today go?

`x_daily_digest`: DMs sent, replies waiting, steps due in the next 24 hours, campaigns active and completed, failed messages. `blockedAccounts` leads the report, with the `x_login_link` URL; then `sessionsAtRisk` (sign those in before they die).

## How are my campaigns doing?

- `x_list_campaigns`: narrow with `status`, `namePrefix`, the ICP, strategy or account; keep only the columns needed with `fields`, or `format: "csv"`. Read `totalMatching` and walk on with `nextOffset` until it is null. Report each campaign's `state.label` and `detail`, never the bare status: `active` does not mean sending, and a never-launched campaign has sent nothing.
- `x_campaign_status` per campaign: counts by status, the strategy, the ICP and `priority`.
- `x_campaign_contacts` for people: paged (read `truncated`, walk with `offset`). A step with `deliveryIssue` showed only in part or in several messages: never resend it.

## Which of these people are already in a campaign?

`x_check_targets` with the handles. Each person lands in ONE bucket, most serious first: `suppressed`, `outcome_recorded`, `in_this_campaign`, `live_elsewhere`, `replied_or_handed_off`, `messaged_before`, `unreachable`, `fresh`. Each row's `campaigns` lists every campaign they are in. Only the first two are refused at import; say the rest to the user before they decide.

## How are results?

- `x_outcome_summary`: `won`, `meetingBooked` (a call on the calendar, not won), `interested`, `notInterested`, and `undecided`, which means nobody has judged those yet: report it as "not judged yet", never as a result. Suggest the `answer-replies` skill to close them out.
- `x_outreach_analytics` with `groupBy` `strategy`, `icp`, `campaign`, `account`, `source`, `sendHour` or `weekday`: reply rates, sends per step and which step people reply to. Say when a number rests on too few sends to trust.
- `x_follow_funnel` for strategies with a follow step: followed, followed back, messaged, replied. `followedBack` is a floor, and null until that account's follower list was read since the follows: unknown, never 0.
- Limits: each account's daily DM cap, warm-up and pacing are its own (`x_account_health`, and `x_set_sending_limits` with no fields to read them). Quote the account's own numbers, never one flat cap, and never suggest raising one.

## Who showed interest on their own?

`x_warm_leads`: new followers of the user's accounts and people who replied to their tweets. A list for the user to pick from, nothing more: nothing ever DMs a lead by itself, and a follow is not consent. `accounts[].readAt` null means that account's list was never read, so an empty list then means nothing. `x_read_warm_leads` reads them now (at most once a day, it opens the browser). `x_dismiss_warm_leads` hides the ones the user decides against. To write to one, use the `start-outreach` skill.

## Who is worth a second attempt?

`x_reengage_candidates`: sequences that ended in silence past the threshold, with no verdict. `timesSequenced` says how many sequences they already had; a fourth is rarely a new idea. Read-only: to start one, use the `start-outreach` skill with a fresh sequence.

## How many lookup credits are left?

`x_enrichment_status`: allowance, used, left and the reset date, and one lookup job's state with `jobId`. Credits, never dollars: never quote a price.

## Never contact these people

`x_suppress_handles` with the handles or profile links and a `reason`: `confirmSuppress: false` to preview (it shows the parsed handles, unreadable lines and who is already listed), then `confirmSuppress: true`. Listed people are refused at every import and before every send. It does not cancel steps already queued: `x_stop_contact` with `alsoSuppress: true` does both. Read the list with `x_list_suppressed`. `x_unsuppress_handle` removes ONE person, only when the user explicitly says so.

## What is the account doing right now?

- `x_account_activity`: what runs now, what waits and why, the sending window and today's counts. "Queued" usually means another job is ahead on the shared browser.
- `x_operations_health`: one verdict on whether outreach is moving, with its cause.
- For setup or sign-in problems, use the `get-started` skill.

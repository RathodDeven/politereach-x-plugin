---
name: using-politereach
description: X (Twitter) overview of PoliteReach for X, the X DM outreach connector. Says what must be connected for each job (the PoliteReach for X connector, an X account connected by cookie import, lookup credits, a calendar for bookings, scheduled tasks for routines), which skill handles which job, and what every x_* tool does with its safety rule. Use when the user asks what PoliteReach for X can do, how to use it, what they need to connect or set up, which tool or skill to use, why something needs a calendar or credits, or makes any PoliteReach X request that no other skill in this plugin covers.
---

# Using PoliteReach for X

PoliteReach for X sends DM outreach from the user's OWN X accounts. You (the assistant) research, write and decide. PoliteReach's server does the X work: DMs, opt-in follows and likes, reply checks, reading threads, scheduled tweets. It runs no model and writes no text. Its tools are named `x_*`.

## What must be connected, per job

| Job | Needs | If it is missing |
|---|---|---|
| Anything | The PoliteReach for X connector, signed in with the user's PoliteReach account | claude.ai, Desktop, Cowork: this plugin's **Connectors** tab, "Continue with your PoliteReach account". Claude Code: `/mcp`, pick `politereach-x`. ChatGPT or Codex: add `https://mcp-x.politereach.com/mcp`. No login yet: https://politereach.com |
| Anything that touches X | At least one X account, connected and checked with `x_verify_account` | Cookie import is the default: the user opens https://app-x.politereach.com/accounts/login, picks Import cookies and pastes the Cookie-Editor JSON export of x.com there. The live browser sign-in on that page is the fallback. Signed out later: `x_login_link` |
| Research and sourcing (`x_prospect_context`, `x_contact_dossier`, `x_enrich_handles`, `x_source_prospects`) | Lookup credits left this month (`x_enrichment_status`) | Out of credits: write from what is stored, or skip the person. Credits, never dollars |
| Booking check (who booked), and the nudges that depend on it | A calendar connector in the SAME app: Google Calendar, or Microsoft 365 / Outlook (a Calendly or Cal.com connector that lists bookings also works), its list-events and get-event tools allowed | Tell the user to connect their calendar in the app's connector settings. Until then, record who agreed but send no nudges. Never guess |
| Routines that run on their own | The app's scheduled-tasks feature (claude.ai and Cowork: **Scheduled**; Claude Code: `/schedule`; ChatGPT: tasks), with the tools set to always allow | Run the routine by hand when the user asks |
| Writing in the user's voice | The user's own outreach skill, "(company)-politereach-x-skill" | The `outreach-skill-builder` skill |

PoliteReach itself never connects to a calendar or a mailbox. You read those through the user's own connectors and send PoliteReach only what is needed (attendee name, email, event id, start time, description).

## Which skill for which job

- Set up, connect or reconnect, account health: `get-started`. Everything in one go (account, outreach skill, routines): its `references/setup-sequence.md`. A read-only morning routine: its `references/morning-check.md`.
- Build the user's voice, ICPs and offers: `outreach-skill-builder`.
- Reach out to people (handles, a source, warm leads, a second attempt): `start-outreach`.
- Keep a running campaign fed from the Prospects pool: `keep-outreach-going`.
- Replies waiting on the user, outcomes, a one-time follow-up for a warm thread: `answer-replies`.
- Who agreed to a call, who booked, nudges: `book-agreed-calls`.
- Tweets now or later, a content calendar: `schedule-posts`.
- How things are going, warm leads, credits, do-not-contact: `pipeline-check`.
- What goes out soon, fixing queued wording, pausing a campaign: `queued-messages`.

Load the matching skill and follow it. Use the tool map below only for jobs none of them covers. The connector also serves a prompt, `x_dm_outreach_workflow`, with the whole campaign flow in one text.

## Tool map

Tools name an account in different ways, and a parameter a tool does not take is dropped without an error: the call then covers every account, or a tweet goes out from the default one. `x_list_accounts` gives each account's label and id. Pass the account the way the tool takes it:

- `sessionId` only: `x_retry_held`, `x_scheduled_steps`, `x_outcome_summary`, `x_warm_leads`, `x_booking_due`, `x_bookings`, `x_schedule_tweet`, `x_session_cookies`.
- No account at all, so always every account: `x_daily_digest`, `x_account_health`, `x_operations_health`, `x_replies_to_answer`, `x_follow_funnel`, `x_due_messages`, `x_scheduled_tweets`, `x_list_conversations`, `x_reengage_candidates`, `x_check_replies`. Report one account from them only where a row names its account; otherwise say the number covers every account.
- Every other tool that takes an account: `sessionLabel` or `sessionId`.

"Preview until" a flag means the call is a dry run until that flag is true.

**Setup**
- `x_setup_guide`: the same setup and routine instructions these skills carry, served by the connector, so an assistant without the plugin can follow them too. No `step` = the whole sequence; `account`, `outreach_skill`, `start_outreach`, `replies`, `follow_up`, `outreach`, `bookings`, `posts`, `morning_check` = one part.

**Accounts and health**
- `x_list_accounts`: every account with label, status, default flag, passcode set, and `verified` (null = never read, not "no"). Start here.
- `x_account_health`: per account `canSend`, `blocked`, `paused`, `verified`, `lastCheck`, `sendHealth.degraded` (clears itself; signing in again does not help) and the warm-up (`state`, today's `cap`).
- `x_verify_account`: prove one account works now. Costs the browser slot for tens of seconds: once, never in a loop. `stillRunning` = read `x_account_health` later. A pass lifts a dead-session block, never an XChat passcode block.
- `x_session_status`: saved sessions; `verifyLive` runs the same check as `x_verify_account` and acts on it the same way.
- `x_login_link`: the link a person opens to sign an account back in. Give it as returned. Never ask for cookies.
- `x_account_activity`: what each account runs now, what waits and why, its sending window and today's counts. Read before calling anything stuck.
- `x_operations_health`: one verdict on whether outreach is moving, with its cause. `sending_held` = every account is on hold; report it, never release it.
- `x_daily_digest`: today's totals, `blockedAccounts` and `sessionsAtRisk` (sign those in before they die).
- `x_job_result`: what a queued send did, by `jobId` or `jobIds`. A timeout is not a failure: read here, never resend.
- `x_set_default_account`: the account that sends campaigns not pinned to one.
- `x_set_office_hours`: the account's sending window, `end` exclusive. Its zone is where the account's proxy exits, never the prospects' zone.
- `x_pause_account`: hold (`paused: true`) or resume all work on one account. For a known outage, not a blocked account.
- `x_set_sending_limits`: the account's daily DM cap and pacing. No fields only reads them. Change only on the user's explicit request, after saying the risk; never propose raising it.
- `x_set_warmup` / `x_restart_warmup`: the account's warm-up ramp. Same rule. `x_restart_warmup` previews until `confirmRestart`.
- `x_set_engagement_budgets`: daily follow and like budgets for the opt-in public steps. No fields only reads them.
- `x_retry_held`: ask held sends to try again after a fix. Each still meets every gate.

**Campaigns and import**
- `x_list_strategies` / `x_create_strategy`: sequence templates of at most five DM steps (`first`, `followup_1` to `followup_3`, `closer`), plus opt-in `like_recent` and `follow` before the opener. Create previews until `confirmCreate`.
- `x_list_icps` / `x_create_icp`: ICP labels for comparing results. Create previews until `confirmCreate`.
- `x_create_dm_campaign`: import people with every DM step's text. Preview until `confirmCreate`. With `campaignId` it adds to that campaign. Show `collisions`, `rejectedRows`, `templateRejections`, `repeatedOpeners` and `accountHold` first.
- `x_add_prospects_to_campaign`: the same import from the Prospects pool (`texts`, or `approvedDraftIds`). Preview until `confirmCreate`.
- `x_launch_campaign`: start a new campaign (`confirmLaunch`). Launched only when `openerQueued` is true. Several: `campaignIds`, or `namePrefix` with `expectCount`.
- `x_list_campaigns`: campaigns with ids and `state.label`; report that, never the bare status. Filters, `fields`, `format: "csv"`, `nextOffset`.
- `x_campaign_status`: one campaign's counts, strategy, ICP and priority.
- `x_campaign_contacts`: per person, with step ids, delivery issues and `retryable`. Paged.
- `x_set_campaign_status`: pause or resume (`confirmChange` for bulk). A pause is not instant; a bulk resume never launches.
- `x_edit_campaign`: name, ICP, `sendWindowMode`, `priority`. The sending account never changes.
- `x_move_contact`: move people between campaigns of the same account (`confirmMove`).
- `x_reset_contact`: revive one sequence that failed for a reason that may pass (`confirmReset`).
- `x_check_targets`: who on a list was already approached, in buckets. Read-only.
- `x_set_contact_timezone`: one person's own zone, for `prospect` or `both` campaigns.

**Queued messages**
- `x_scheduled_steps`: every queued step with its send time and hold reason, soonest first.
- `x_due_messages`: steps already due (a long list means the worker is behind).
- `x_edit_step`: rewrite unsent steps by step id (`confirmEdit`).
- `x_bulk_edit_steps`: replace one exact sentence across a campaign. `apply` false previews; not idempotent.
- `x_cancel_step`: hold named steps (`confirmCancel`). Cannot be undone; rewrite instead when the message is still wanted.
- `x_stop_contact`: end a person's live sequences (`confirmStop`); `alsoSuppress: true` puts them on the do-not-contact list too.

**Sending**
- `x_send_reply`: answer one or many people (`replies`), each from the account holding the thread. Preview until `confirmSend`. Returns job ids at once; a job id means queued, never send twice. Booking nudges and closes go only through here, with `bookingNudge` or `bookingClose` and `calendarChecked: true`.
- `x_send_dm_now`: one DM that holds the call open on a browser send. Only on the user's explicit request for one person; never in a routine. Preview until `confirmSend`.
- `x_check_dm_availability`: whether ONE handle can be DMed. Costs the browser slot: never loop it.

**Conversations and verdicts**
- `x_replies_to_answer`: people owed an answer, longest-waiting first; `includeThread: true` adds the thread. `intent` is a suggestion; `failedReply` means an answer that may still have arrived.
- `x_list_conversations`: every captured thread, inside a campaign or not.
- `x_conversation_history`: the stored thread, instant. Empty means never read, not "they said nothing".
- `x_read_conversation`: one thread live, tens of seconds. `x_read_conversations`: up to 25 in ONE job; never loop the single one.
- `x_check_replies`: re-read finished threads now. Only on a person's "check now": the hourly sweep is built in.
- `x_contact_dossier`: everything stored about one person, looked up when stale.
- `x_set_outcome`: the verdict: `won`, `meeting_booked`, `interested`, `not_interested`. A booked call or a demo is never `won`. A verdict cancels every pending step to that person.
- `x_mark_reply_handled`: close replies dealt with (`confirmMark`), with or without an outcome.
- `x_snooze_reply`: park a thread until a date. Nothing is sent when it ends.
- `x_mark_replied`: record a reply the check missed (`confirmMark`); it stops their sequence.
- `x_outcome_summary`: how conversations ended; `undecided` = not judged yet, never a result.

**Bookings** (you read the calendar; PoliteReach stores state and enforces the limits)
- `x_booking_due`: people recorded `interested`, by stage: `nudge_due`, `close_due`, `they_wrote`, `waiting`, `done`. Agreed, then ONE nudge, then a close, then done.
- `x_match_contacts`: calendar attendees to contacts. Mark only a `match`; skip `ignored`. Never pass `createdAt`.
- `x_ignore_calendar_event`: the user says a booking is not from X (`eventId` or `email`). Only on their word.
- `x_mark_booked`: a call is on the calendar (`calendarEventId`, `meetingStart`); same event, new time is a move.
- `x_mark_booking_cancelled`: the event is gone or cancelled.
- `x_bookings`: booked calls, to compare with the calendar.

**Tweets**
- `x_schedule_tweet`: one tweet at a chosen time on the exact account (`sessionId`), `confirmSchedule`. `accountHold` / `sendHold` in the reply = it would not post now: hand over `fixUrl`, never move it to another account.
- `x_post_tweet`: publish now (`confirmPost`). Only on an explicit "post now"; it leaves no record.
- `x_scheduled_tweets`: queued, posted, failed and cancelled tweets.
- `x_cancel_scheduled_tweet`: cancel one not yet posted.

**Finding people** (lookup credits, never dollars)
- `x_source_prospects`: fill the Prospects pool from followers, following, an X list, a tweet's repliers or a search. Dry run (`creditsNeeded`), then `confirm: true`; poll by `jobId`.
- `x_list_prospects`: the pool, with stored profiles and a `filter`. A list only.
- `x_prospect_context`: bio and recent original tweets per handle, looked up automatically when missing or stale.
- `x_enrich_handles`: bulk lookup for a whole list or campaign. Dry run, then `confirm: true`.
- `x_enrichment_status`: credits left, the ledger, one lookup job's state.
- `x_submit_opener_drafts`: store your written openers as drafts for the user to approve in the Drafts panel. Never sends.
- `x_warm_leads`: new followers and tweet repliers. A list only: nothing DMs a lead by itself. `x_read_warm_leads`: read the lists now (at most once a day). `x_dismiss_warm_leads`: hide ones the user decided against.
- `x_reengage_candidates`: silent past the threshold, no verdict. Read-only.

**Reporting**
- `x_outreach_analytics`: reply rates by strategy, ICP, campaign, account, source, send hour or weekday. Say when a sample is too small.
- `x_follow_funnel`: for follow strategies, followed, followed back, messaged, replied. A follow-back is unknown until that account's follower list was read.

**Do-not-contact**
- `x_suppress_handles`: add people. Preview until `confirmSuppress`. Refused at every import and send.
- `x_list_suppressed`: read the list. `x_unsuppress_handle`: remove ONE person, only on the user's explicit word (`confirmUnsuppress`).

Operator only, not for normal use: `x_host_health`, `x_selector_canary`, `x_session_cookies`, `x_hold_sending`, `x_set_workspace_billing`, `x_set_default_pricing_tier`, `x_set_workspace_complimentary`, `x_set_account_complimentary`, `x_set_user_default_proxy`.

## Ground rules, everywhere

- **Preview, then confirm.** Show the dry run, get the user's yes, then call again with the confirm flag. A scheduled task acts only in the mode the user chose when setting it up.
- **Prefilled text only.** Every DM and tweet is written now and approved. PoliteReach never writes text at send time and adds nothing to a booking link.
- **Limits are the account's own.** The daily DM cap, warm-up, pacing and sending hours are per account. Quote the account's own numbers, never one flat cap. Never raise one unasked.
- **Follows and likes stay public steps.** They spend the follow and like budgets, never DM, never reply to or quote anything, and are never "we messaged them".
- **Warm leads are a list only.** A follow is not consent: a person picks who to write to.
- **No cookies or passwords in chat.** Cookie import is the default way to connect, and it happens on https://app-x.politereach.com/accounts/login, never here. Never ask for, accept or repeat `auth_token`, `ct0`, cookie JSON, passwords or codes; if the user pastes one, tell them to delete that message and import the cookies on the PoliteReach page.
- **A sign-in or blocked error means stop.** Give the `x_login_link` URL. Never retry around it.
- **Never guess the account.** With more than one, every write names it.
- **Relay refusals.** A refusal says why (do-not-contact, a recorded verdict, a held account, credits). Relay it with any `fixUrl` it carries; never work around it.
- **Credits, never dollars.** Never quote a price for a lookup.
- **Your own accounts only.** At most five DMs per sequence.

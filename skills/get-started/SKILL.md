---
name: get-started
description: X (Twitter) setup for PoliteReach for X. Connects the connector to the user's AI assistant and an X account to PoliteReach (cookie import by default), then checks the account is ready to send and reads back its own cap, warm-up and hours. Use when the user is setting up PoliteReach for X or wants everything set up in one go, asks how to connect or reconnect an X account, an account says it needs signing in again, or they ask whether an account is healthy, stuck, blocked or what it is doing right now.
---

# Get started with PoliteReach for X

PoliteReach for X sends DMs from the user's OWN X accounts. You (the assistant) decide and write; PoliteReach's server does the X work. Its tools are named `x_*`.

The user wants everything set up in one go (account, outreach skill, then routines): follow `references/setup-sequence.md`, which starts with the steps below.

## 1. Check the connector

Call `x_list_accounts`.

- Tools missing: the PoliteReach for X connector is not connected yet.
  - claude.ai or Cowork: open this plugin's **Connectors** tab and connect it with "Continue with your PoliteReach account".
  - Claude Code: run `/mcp`, pick `politereach-x`, and sign in in the browser.
  - ChatGPT or Codex: add a connector with the URL `https://mcp-x.politereach.com/mcp`, then sign in with your PoliteReach account.
  - No PoliteReach login yet: start at https://politereach.com.
- Tools present: keep the account labels and ids it returns. Most account tools take the label (`sessionLabel`) or the id (`sessionId`); some take the id only, and some cover every account (the `using-politereach` tool map lists them). With more than one account, every write names one. Never guess.

## 2. Connect or reconnect an X account

Cookie import is the default way to connect. It happens on PoliteReach's own page, never in chat:

1. The user opens https://app-x.politereach.com/accounts/login and picks **Import cookies**.
2. On x.com, signed in to the account, they open the Cookie-Editor browser extension, press **Export**, then **JSON**, and paste that into the PoliteReach page. Without the extension: copy `auth_token` and `ct0` from the browser's DevTools (Application, Cookies, x.com) into the same page.
3. PoliteReach saves the account and names it after its handle.

The live browser sign-in on the same page is the fallback: X refuses a sign-in from a server for some accounts, and then cookie import is the way in.

- An account that needs signing in again: call `x_login_link` and give the user that link exactly as returned. Sending resumes by itself once they finish.
- Never ask for, accept or repeat `auth_token`, `ct0`, cookie JSON, passwords, passcodes or one-time codes in chat. If the user pastes one, tell them to delete that message and import the cookies on the PoliteReach page instead.

## 3. Confirm it really works

- `x_verify_account` once for the account. It costs the one browser slot for tens of seconds: never run it in a loop.
  - `stillRunning`: the check goes on and records its answer. Read `x_account_health` a little later; never run the verify again.
  - `isLoggedIn: null` means the check could not tell. It says nothing about the cookies.
  - A pass lifts a dead-session block and re-asks the held sends. It never clears an XChat passcode block: only a sign-in does.
- Read back from `x_account_health`, per account, in plain words, the account's own numbers and never one flat cap for everybody:
  - `canSend`, `blocked`, `paused`.
  - `verified`: true, false, or null (never read, which is not "no").
  - Warm-up: `warmup.state` (off, not_started, ramping, done) and today's `warmup.cap`.
  - The daily DM cap and pacing: `x_set_sending_limits` with no fields only reads them (`effective`, `capToday`, `pacing`).
  - Follow and like budgets: `x_set_engagement_budgets` with no fields only reads them.
  - Sending window and zone (`x_account_activity` shows `sendingWindow`).
  - `sendHealth.degraded`: several sends in a row failed for a reason X puts on the account. It clears itself; signing in again does not help.
  - `sessionsAtRisk` from `x_daily_digest`: sign those in before they die.
  - Lookup credits left: `x_enrichment_status`. Credits, never dollars.

## 4. Optional settings (only when the user asks)

- Sending hours: `x_set_office_hours`. `end` is exclusive (20 means up to 19:59). The `timezone` is **where the account's proxy exits**, because it is also the zone the account's browser reports. Never set it to the prospects' zone. Prospects in several markets: the campaign's `sendWindowMode` (`prospect` or `both`, with `x_edit_campaign`) plus `x_set_contact_timezone` per person.
- Default sender: `x_set_default_account`.
- Daily DM cap, pacing, warm-up (`x_set_sending_limits`, `x_set_warmup`, `x_restart_warmup`): only on the user's explicit request, after saying the risk out loud: faster sending is how X restricts an account. Never propose raising a limit. `x_restart_warmup` previews until `confirmRestart`.
- Follow and like budgets: `x_set_engagement_budgets`.
- Booking follow-up pace: `x_set_booking_settings` with `nudgeScheduleDays` (default [3, 4]: days after each message of ours, the last one the close, so one nudge after 3 days and the close 4 days after it).

## When something looks stuck

- `x_operations_health` first: one verdict with its cause. Then `x_account_activity` (what runs now, what waits and why) and the hold reasons in `x_scheduled_steps`. Every browser job shares the worker, so "queued" is usually normal.
- After a fix (a sign-in, a pause lifted): `x_retry_held` asks held sends to try again; each still meets every gate.
- A known outage (a dead proxy, X asking the account to verify): `x_pause_account` with `paused: true` holds all work without alerts; `paused: false` resumes. It does not fix a blocked account.
- A sign-in or blocked error: stop. Give the user the `x_login_link` URL. Never retry around it.
- `sending_held` (a send hold after a misdelivery, or an admin hold): report it. Never release it yourself.

## Next

1. Build the user's outreach skill: the `outreach-skill-builder` skill.
2. Start outreach to people: the `start-outreach` skill.
3. Keep conversations moving after a reply, a call or a proposal: the `keep-warm-conversations` skill.
4. What else PoliteReach for X does, and what to connect for it (a calendar for bookings, scheduled tasks for routines): the `using-politereach` skill.

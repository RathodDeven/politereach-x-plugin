---
name: keep-outreach-going
description: X (Twitter) daily top-up for PoliteReach for X. Adds new people from the Prospects pool to a running DM campaign on each account, sized to the account's DM cap today, with each person's sequence written in the user's voice, or submits the sequences as drafts for review. Use when the user asks to keep a campaign fed, add today's people, top up outreach from the pool, says the queue is running dry, or wants this on a daily schedule.
---

# Keep X outreach going

A launched campaign sends one person after another, inside each account's DM cap, warm-up, pacing and sending hours. When it runs out of people, it stops. This skill keeps it fed from the Prospects pool, a day's worth at a time, never faster than the account sends.

## First

- Load the user's outreach skill, "(company)-politereach-x-skill". None: offer the `outreach-skill-builder` skill first.
- `x_account_health`: today's `warmup.cap` per account, and `canSend`. Skip an account that cannot send and give its fix: the `x_login_link` URL for a blocked one, the PoliteReach Accounts page for a paused one.
- `x_list_campaigns` for the account's running campaign (one per account and offer). Report `state.label`; a campaign nobody launched sends nothing.

## How many

`x_campaign_status` on that campaign. When the people still waiting for their opener already cover two days of the account's cap today, add nobody. Otherwise add at most one day's cap. A paused or completed campaign: add nobody, and say so.

## Who

- `x_list_prospects` (status `new`, and the source list the user names) and judge each person against the skill's ICPs on the stored profile. `filter` can narrow by bio words, follower range or verified.
- The pool too small: say so. Find more only if the user agrees: `x_source_prospects` with `confirm` false (shows `creditsNeeded`), then `confirm` true on a yes; its people arrive in the pool for the next run.
- Research with `x_prospect_context`. It looks up anyone missing or stale by itself from the lookup credits; when `lookup.state` is `running`, call again in a minute. Out of credits: write from the bio, or skip the person.

## Write and add

- One text for every DM step of the campaign's strategy, the opener under 300 characters and tied to one recent tweet. Show every word.
- Add on the user's yes: `x_add_prospects_to_campaign` with the `campaignId` and `texts` per person, `confirmCreate` false first. Drop every row in `missingText`, `templateRejections` or `refusalSummary`, then the same call with `confirmCreate` true for `wouldImport` only. They join the running drip: report `start.startsWhen` and any `accountHold` with its `fixUrl`.
- Or as drafts: `x_submit_opener_drafts` and report the `batchId`, so the user approves them in the Drafts panel on the Campaigns page. Approved drafts go in later: `x_submit_opener_drafts` with only that `batchId` lists them, and the approved ones without `usedAt` go into `x_add_prospects_to_campaign` with `approvedDraftIds`. Never draft a person again while an earlier draft of theirs waits or is approved: only a person's newest draft counts, so a new one withdraws the approval.

Never make a new campaign or launch one from here: this skill only adds to a campaign that is already running.

## Put it on a schedule

Follow `references/scheduled-task.md`.

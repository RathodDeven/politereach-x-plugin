---
name: answer-replies
description: X (Twitter) DM replies through PoliteReach for X. Finds every reply waiting on the user across their X accounts, drafts answers in their voice, sends the approved ones from the account holding each thread, and records how each conversation went. Use when the user asks who replied on X, what needs an answer, to clear or triage their X DMs, to answer replies, to record outcomes, to plan the next touches for a warm thread, or to put reply handling on a schedule.
---

# Answer X DM replies

**Connect the calendar first.** Ask the user to connect their calendar in this app's connector settings (Google Calendar or Microsoft 365 / Outlook, or a Calendly or Cal.com connector that lists bookings) if it is not: it is how you see who already booked a call. Without it, anyone who agreed to a call is flagged for the user, never answered. PoliteReach reads no calendar.

Work the whole queue in batches. Every tool here takes a list, so a queue is a handful of calls, not one browser trip per person.

## 1. Find what is owed

- `x_replies_to_answer` with `includeThread: true`: one call covers every account, and each row names the account holding the thread. Rows carry their words, the last step we sent, their profile, an `intent` and `failedReply`.
- `x_list_conversations`: every thread a reply check captured, including people outside a campaign, with whether they wait on an answer.
- A row with no captured words: `x_conversation_history` for that handle (stored, instant; empty means never read, not "they said nothing"). Still nothing: group those rows by the account holding the thread (each row's `account.id`) and read each group in ONE `x_read_conversations` call with that account's `sessionId`, at most 25 handles a call, never one by one. One call reads every handle from one account, so a thread read from another account comes back empty or `noThread`. A row whose `account` is null has no known account: flag it, never read it from a guessed one. A `failed` row is unknown, never an empty thread.
- Before writing to someone, `x_contact_dossier` holds their profile, every sequence and the history.
- `x_check_replies` only when the user says "check now": the late-reply sweep already runs every hour inside sending hours. Its results land in `x_replies_to_answer`, not in its own reply.

## 2. Decide, one per thread

- **Flag for the user, never send:**
  - anything on their always-flag list (pricing beyond what their skill allows, contracts, NDAs, invoices, anything legal);
  - an `intent` of `stop_request`: a suggestion only; offer the do-not-contact list and add them only on the user's yes;
  - a row with `failedReply`: that answer may still have arrived, so read the thread before writing again;
  - anything senior or unclear.
  Give their last message, why, and a draft the user can send.
- **Reply:** you understand the ask and the answer needs no invented fact. Draft in the user's voice (their outreach skill). If their last message is over 14 days old, open with one short "sorry, this slipped past me."
- **They agreed to a call, or say they booked:** search the calendar for them first (name, then handle): people book minutes after saying yes.
  - Booked: send nothing. `x_mark_booked` with the event id and start. Unsure who it is: `x_match_contacts`, and mark only a `match`.
  - Not booked: send the plain booking link from their skill (PoliteReach does NOT add their name to it), then `x_set_outcome` with `interested` and their words as the note, so the booking follow-through picks them up.
  - No calendar connector: never send to them. Flag them.
- **Skip:** dead threads (a flat no, a thumbs up, a vendor pitching the user). Unsure between reply and skip: flag.

## 3. Send only what the user approved

Show every draft. On a yes: ONE `x_send_reply` call with `replies: [{handle, message}, ...]` and `confirmSend: true`. Each reply goes from the account holding that thread (`sendsFrom`). One refused row queues nothing: drop or fix it and call again. A job id means queued, not failed: never call it again for the same person. A refusal naming the duplicate-send guard means those words already went or are queued: relay it, never resend or force it. Outcomes only when asked, with ONE `x_job_result` call passing `jobIds`. Never `x_send_dm_now` here: it holds the call open on a browser send.

## 4. Record a verdict for every thread read

One `x_set_outcome` per verdict, with `handles` and a short `note`:

- `not_interested`: a clear no.
- `interested`: agreed to talk, nothing booked.
- `meeting_booked`: a call actually on the calendar. Never `won`.
- `won`: a deal actually agreed. A demo is not won.

A verdict cancels every pending step to that PERSON, in every campaign. A sent answer leaves the queue by itself.

Other marks:
- A thread dealt with and no verdict yet: `x_mark_reply_handled` with `confirmMark: false` to preview, then `confirmMark: true`.
- "Try again in (month)": `x_snooze_reply` with `until`. Nothing is sent when it ends.
- A reply the automatic check missed (the user saw it on x.com): `x_mark_replied`, preview first, then `confirmMark: true`. It stops their sequence.
- "Remove me": on the user's yes, `x_suppress_handles` (preview, then `confirmSuppress: true`), or `x_stop_contact` with `alsoSuppress: true` to stop and suppress at once.
- Where the deal now stands (a call done, a proposal out, a trial started): `x_set_stage` with `handles` and the stage; `call_done` takes `note`, `nextStep`, `owner` and `dueDate`. Then offer that stage's chain.
- The conversation moved to email, WhatsApp or a call: `x_set_contact_email` when they give an address, and `x_log_touch` for each touch off X (`direction: "in"` for theirs stops their chains and campaign follow-ups).
- Flagged threads: mark nothing.

A closed conversation comes back by itself if they write again.

## Warm threads: book the next touch

A thread with no sequence left (they replied, a call happened, a proposal went out, you wait on them) has nothing scheduled, so it gets forgotten. When the next move is to wait, offer a follow-up chain for that person: the next 1-4 touches, written now, sent or drafted when due, stopped by itself the moment they write. The `keep-warm-conversations` skill has the rules (`x_create_followup_chain`, preview, then the user's yes). Only when the user wants the follow-up written on the day, from the thread as it stands then, offer a one-time scheduled task instead: `references/follow-up-task.md`.

## Message rules

At most three short sentences, one idea per line. No em dashes, exclamation marks, emojis or filler ("just checking in", "circling back"). End on a concrete ask. Never invent a price, client, metric or date.

## Stop conditions

A sign-in or blocked error: stop and give the user the `x_login_link` URL. Never retry around it.

## Put it on a schedule

To run this unattended, follow `references/scheduled-task.md`.

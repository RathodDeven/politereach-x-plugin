# Scheduled task: X replies

Create nothing until the user has answered part 1.

The task writes with the user's outreach skill, named "(company)-politereach-x-skill". Find it among their skills first. If theirs has another name, ask which one it is and use that name. If they have none, offer to build it first (the `outreach-skill-builder` skill, or `x_setup_guide` step `outreach_skill`) and stop.

Also needed: the PoliteReach for X tools (`x_*`), and the app's scheduled-tasks feature (claude.ai and Cowork: Scheduled; Claude Code: `/schedule`; ChatGPT: tasks). Without scheduling, offer to run the TASK PROMPT below once, by hand, in this conversation.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets, so the user can answer "defaults". If their outreach skill already answers one, say what you took from it. Ask their timezone if unknown.

1. Which PoliteReach X accounts should it cover? Use the labels from `x_list_accounts`; the task keeps each one's id with its label. [all of them]
2. Send or draft? Send: it answers the clear replies itself, records how each conversation went and flags the rest. Draft: it writes every reply and the verdict it would record, and sends and records nothing. [draft]
3. What must always come to you instead of being answered? [pricing beyond what your skill allows, contracts, NDAs, invoices, anything legal, anyone asking to be left alone]
4. Your booking link, if your skill does not have it. And which calendar the bookings land on (connect it in this app: Google Calendar or Microsoft 365 / Outlook, with its tools allowed to run on their own), so it can see who already booked. Without one, anyone who agreed to a call is flagged for you instead of answered.
5. When should it run, in your timezone? [every 2 hours from 10:00 to 20:00, every day]

## Part 2: create the task

Create a recurring scheduled task named "X replies" at the chosen times, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's (company)-politereach-x-skill to the task and put its exact name in CONFIG, so every run loads it.

Then tell the user the task name and next run, and remind them to set every tool it uses to always allow: nobody is there to approve a tool while it runs. Choosing send mode here is the user's standing approval to send. Offer one run now so they can watch it.

## TASK PROMPT

```text
Answer my X DM replies via PoliteReach. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my company)-politereach-x-skill
- Accounts: (label and id of each, from x_list_accounts)
- MODE: (send | draft). In draft mode never call x_send_reply, x_set_outcome, x_mark_booked, x_mark_reply_handled or x_snooze_reply: put each reply you would have sent, with the verdict you would have recorded, under DRAFTS in the report. A verdict takes the person off my reply queue and cancels their pending steps, so only I record one while in draft mode.
- Always flag, never answer: (my list)
- Booking link: (my link)
- Calendar connector: (Google Calendar | Microsoft 365 | none)

SETUP
1. Load my outreach skill (named in CONFIG) and follow its voice and rules for every word. If it cannot be loaded, notify me "(task name) failed: skill not found" and stop.
2. Make sure the PoliteReach X tools (x_*) are loaded (if this app defers tools, search for them first). If they are missing, notify me "(task name) failed: PoliteReach X not connected" and stop.
3. x_account_health. Match the CONFIG accounts by id, never by label: a label changes when an account is renamed. An id no longer listed: skip that account and report "account not found: re-run setup"; never pick another account. Skip and report any account whose canSend is false, with its fix: the x_login_link URL for a blocked account, the PoliteReach Accounts page for a paused one.

FIND WHAT IS OWED
4. x_replies_to_answer with includeThread true: one call covers every account, and each row names the account holding the thread (account.id). Keep the rows whose account.id is a CONFIG account. A row whose account is null: flag it.
5. A row with no captured words: x_conversation_history for that handle. If nothing is stored either, group those rows by account.id and make one x_read_conversations call per account (never one by one) with that account's sessionId, at most 25 handles a call: one call reads every handle from one account. Still unreadable: flag it.

DECIDE, one per thread
A. FLAG, never send: anything on the always-flag list, a row whose intent is stop_request (suggest the do-not-contact list; never add them yourself), a row with failedReply (that answer may still have arrived), anything senior or unclear. Give their last message, why, and a draft I can send myself.
B. SEND when confident: you understand the ask and the answer needs no fact you would have to invent. x_send_reply with replies[] for the whole batch in ONE call, confirmSend true. Each reply goes from the account holding that thread. A job id means queued, not failed: never call it again for the same person. A refusal naming the duplicate-send guard means those words already went or are queued: never resend, never force.
If they agreed to a call or say they booked, search the CONFIG calendar for them first: people book minutes after saying yes. Booked: send nothing and record it with x_mark_booked (x_match_contacts when unsure; mark only a match); in draft mode list it under BOOKED, unrecorded. Not booked: send my plain booking link, then x_set_outcome with outcome interested and their words as the note, so the booking follow-through picks them up. No calendar connector: never send to them; FLAG them instead.
If their last message is over 14 days old, open with one short "sorry, this slipped past me."
C. SKIP dead threads (a flat no, a thumbs up, a vendor pitching us). When unsure between send and skip, flag.

RECORD A VERDICT (send mode only; in draft mode record nothing) for every thread read, one x_set_outcome call per verdict with a short note: not_interested (a clear no), meeting_booked (a call actually booked), won (a deal actually agreed; a demo is not), interested (agreed to talk, nothing booked). A sent answer leaves the queue by itself. A skipped thread with no verdict: x_mark_reply_handled with confirmMark false to check the row, then confirmMark true. "Try again in (month)": x_snooze_reply. Flagged: mark nothing. Someone who gave an email address in the thread: x_set_contact_email with it.

MESSAGE RULES (hard)
Max 3 short sentences, one idea per line. No em dashes, no exclamation marks, no emojis unless my skill uses them. No filler: no "just checking in", "circling back", "hope this finds you well". Match the tone of the thread and my skill's voice. End on a concrete ask, never "let me know". No link except my booking link or one my skill allows. Never invent a price, client, metric or date.

If any tool returns a sign-in or blocked error, stop work on that account and report the x_login_link URL; never retry around it.

REPORT: FLAGGED first, then BOOKED (no reply sent), then SENT (or DRAFTS), each message in a code block, then one "Skipped: N" line per account. Nothing at all: "Nothing owed."
Notify me once only when something was sent, drafted or flagged, first line the counts. Notify too if the run could not work (connector failing, every account blocked).
```

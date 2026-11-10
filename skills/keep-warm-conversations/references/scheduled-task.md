# Scheduled task: X warm follow-ups

Create nothing until the user has answered part 1.

The task writes with the user's outreach skill, named "(company)-politereach-x-skill". Find it among their skills first. If theirs has another name, ask which one it is and use that name. If they have none, offer to build it first (the `outreach-skill-builder` skill, or `x_setup_guide` step `outreach_skill`) and stop. Its `references/stage-templates.md` holds the default chain per stage; if it has none, the task proposes chains without a template.

Also needed: the PoliteReach for X tools (`x_*`), and the app's scheduled-tasks feature (claude.ai and Cowork: Scheduled; Claude Code: `/schedule`; ChatGPT: tasks). Without scheduling, offer to run the TASK PROMPT below once, by hand, in this conversation.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets, so the user can answer "defaults". If their outreach skill already answers one, say what you took from it. Ask their timezone if unknown.

1. Which PoliteReach X accounts should it cover? Use the labels from `x_list_accounts`; the task keeps each one's id with its label. [all of them]
2. Send or draft? Send: it sends the due X steps of chains you already approved, steps you saved as draft included, word for word, only while nothing new happened in the thread; everything it writes itself comes to you as a draft. Draft: it sends nothing and shows you every due touch. [draft]
3. Which email do your email follow-ups go out from? Only that connector is used, to put email steps in your drafts and to notice which ones you sent. [none: email steps come to you as text]
4. Which calendar do your bookings land on? It is used to make sure nobody is chased after booking a call. [the connected one; none = anyone with your booking link is drafted for you, never sent]
5. When should it run, in your timezone? [every day at 09:30]

## Part 2: create the task

Create a recurring scheduled task named "X warm follow-ups" at the chosen time, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's (company)-politereach-x-skill to the task and put its exact name in CONFIG, so every run loads it.

Then tell the user the task name and next run, and remind them to set every tool it uses to always allow, the email and calendar tools included: nobody is there to approve a tool while it runs. Choosing send mode here is the user's standing approval to send the chain steps they already approved. Offer one run now so they can watch it.

## TASK PROMPT

```text
Work my warm X conversations via PoliteReach: whoever's next touch is due today. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my company)-politereach-x-skill
- Accounts: (label and id of each, from x_list_accounts)
- MODE: (send | draft). In draft mode never call x_send_reply: put each message you would have sent under DRAFTS in the report.
- Email connector: (Gmail | Outlook | other | none)
- Calendar: (which connector and calendar, or none)

SETUP
1. Load my outreach skill (named in CONFIG) and follow its voice and rules for every word. If it cannot be loaded, notify me "(task name) failed: skill not found" and stop. Read its references/stage-templates.md if it has one.
2. Make sure the PoliteReach X tools (x_*) are loaded (if this app defers tools, search for them first). If they are missing, notify me "(task name) failed: PoliteReach X not connected" and stop.
3. x_account_health. Match the CONFIG accounts by id, never by label: a label changes when an account is renamed. An id no longer listed: skip that account and report "account not found: re-run setup"; never pick another account. Skip and report any account whose canSend is false, with its fix: the x_login_link URL for a blocked account, the PoliteReach Accounts page for a paused one.
4. Load the CONFIG email and calendar tools, if any. One that is missing is not a stop: email steps then come to me as text, and anyone with my booking link in the thread is drafted, never sent. Say which one was missing in the report.

FIND WHAT IS DUE
5. x_warm_due with the CONFIG account's sessionId and limit 100, once per CONFIG account. There is no paging: if truncated or scanTruncated is true, work the rows you got and say in the report "list incomplete: N more due, not shown" (N = total minus rows); never call the list complete. If the read is unreadable or fails, notify me "(task name) failed: warm list unreadable" and stop; never report "nothing due" from a failed read.
6. Rows come most overdue first. Before writing to anyone, read their thread with x_conversation_history (stored, instant); never a live read per person.

WORK EACH ROW BY ITS REASONS
A. chain_step_due with a draftedStep:
   - Read the thread first. Anything new from them since the chain started: send nothing, list under NEEDS CHECK (the chain should have stopped).
   - My booking link is in the thread, or a call was agreed or booked: search my calendar for them first (name, then handle). Booked: send nothing, x_mark_booked with the event id and start, list under BOOKED. No calendar: draft only.
   - draftedStep.sendsItself true: PoliteReach sends this step itself in sending hours. Never send it: list it under LATER TODAY.
   - Other X step (a draft step): in send mode, x_send_reply with that handle and draftedStep.text word for word, confirmSend true, once; it goes from the account holding their thread. Only when the text still fits the thread; if it no longer does, rewrite it and put both under DRAFTS instead of sending. In draft mode, list it under DRAFTS.
   - Email step: never send email. If the email connector can make drafts, create one to the row's email with the step text and a short subject, and list it under EMAIL DRAFTS. No connector or no address: list the text under EMAIL DRAFTS.
B. Email steps I already sent: if the email connector can search sent mail, look for a message to the row's email since the step's dueAt. Found: x_log_touch with channel email, direction out, at = its sent time, note "chain step sent". The step then completes by itself.
C. they_wrote_last: never send. Write a suggested answer in my voice under OWED, naming the channel they wrote on. An X reply is also handled by my replies routine: say so instead of drafting twice if one exists.
D. next_step_date: the stage's nextStep is due. Draft the touch that does it under DUE, with the stage note, owner and date. If the next step is not a message (send a proposal, set up an account), list it as a task for me.
E. quiet: a live stage, no chain, no touch either way for 7 days. Write the chain from the stage's template in my skill (steps, delays, channel, mode, every word written for this person) under PROPOSED CHAINS. Never create a chain in this run: I approve each one in chat, and it is created there with x_create_followup_chain.
F. Every row: unansweredXRun 2 or more means no answer to our last X messages (X has no read receipts, so never call them unread or ignored). suggestedChannel email: make the next touch an email draft. emailMissing true: write "email (no address stored)" next to the person.

DUPLICATES: a send refused because the identical words went or were queued minutes ago already went. Check x_job_result or the thread, list it under NEEDS CHECK, and never resend; never use force. A refusal that says the duplicate check could not be made was NOT sent: try it once more after a minute, never with force.

MESSAGE RULES (hard)
Max 3 short sentences, one idea per line. No em dashes, no exclamation marks, no emojis. No filler: no "just checking in", "circling back", "bumping this". Each touch brings one new reason or one clear ask. Never repeat what the thread already covered. Never invent a price, client, metric or date.

If any tool returns a sign-in or blocked error, stop work on that account and report the x_login_link URL; never retry around it.

REPORT: OWED first, then DUE, then SENT (or DRAFTS), EMAIL DRAFTS, BOOKED, PROPOSED CHAINS, LATER TODAY, NEEDS CHECK. Each message in a code block, each person with their handle, stage and the account holding the thread. Then one line "Warm: N due, N sent, N drafted, N chains proposed".
Notify me once only when something was sent, drafted, proposed or needs me: first line the counts, then one short line per person who wrote last. Notify too if the run could not work (connector failing, every account blocked).
EMPTY RUN = SILENCE: nothing due means no notification and no message. End with exactly "Nothing due."
```

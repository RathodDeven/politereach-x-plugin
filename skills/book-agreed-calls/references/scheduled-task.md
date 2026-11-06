# Scheduled task: X bookings

## Before anything: the calendar connector

This task cannot work without a calendar connector in the same app as PoliteReach for X:

- Google Calendar, or Microsoft 365 / Outlook calendar (or a Calendly or Cal.com connector that lists bookings).
- Its list-events and get-event (or search-events) tools must be loaded and set to always allow, because nobody is there to approve them during a run.

Load those tools now. If none are there, tell the user: "This task needs your calendar. Connect Google Calendar or Microsoft 365 / Outlook in this app's connector settings, set its tools to always allow, then ask me again." Then stop. PoliteReach never connects to the calendar; the assistant reads it and passes the bookings over.

The task writes with the user's outreach skill, named "(company)-politereach-x-skill". Find it among their skills first. If theirs has another name, ask which one it is and use that name. If they have none, offer to build it first (the `outreach-skill-builder` skill, or `x_setup_guide` step `outreach_skill`) and stop.

Also needed: the PoliteReach for X tools (`x_*`), and the app's scheduled-tasks feature (claude.ai and Cowork: Scheduled; Claude Code: `/schedule`; ChatGPT: tasks). Without scheduling, offer to run the TASK PROMPT below once, by hand, in this conversation.

Create nothing until the user has answered part 1.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets, so the user can answer "defaults". If their outreach skill already answers one, say what you took from it. Ask their timezone if unknown.

1. Which PoliteReach X accounts should it cover? Use the labels from `x_list_accounts`; the task keeps each one's id with its label. [all of them]
2. Send the nudge and the close itself, or draft them for you? Send: one nudge, then one polite close, each only when PoliteReach says it is due. [draft]
3. Which calendar connector and calendar do your bookings land on? [the connected one, your primary calendar]
4. Your booking link, if your skill does not have it.
5. Anything a nudge must never say. [never propose a time, never pitch]
6. The email domain of your own team, so internal meetings are ignored.
7. Anyone who books on your calendar but never through X (their email). Mark each with `x_ignore_calendar_event` now, so every run skips them. [none]
8. When should it run, in your timezone? [every day at 11:20, 15:20 and 19:20]

## Part 2: create the task

Create a recurring scheduled task named "X bookings" at the chosen times, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's (company)-politereach-x-skill to the task and put its exact name in CONFIG, so every run loads it.

Then tell the user the task name and next run, and remind them to set every tool it uses to always allow, the calendar tools included: nobody is there to approve a tool while it runs. Choosing send mode here is the user's standing approval to send. Offer one run now so they can watch it.

## TASK PROMPT

```text
Booking follow-through: make sure people who agreed to a call on X actually book it. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my company)-politereach-x-skill
- Accounts: (label and id of each, from x_list_accounts)
- Calendar: (which connector and calendar)  My team's domain: (domain)
- Booking link: (my link)  Never say: (my answer)
- MODE: (send | draft). In draft mode never call x_send_reply: put each nudge and close under DRAFTS in the report.

SETUP
1. Load my outreach skill (named in CONFIG) and follow its voice and rules for every word. If it cannot be loaded, notify me "(task name) failed: skill not found" and stop.
2. Make sure the PoliteReach X tools (x_*) are loaded (if this app defers tools, search for them first). If they are missing, notify me "(task name) failed: PoliteReach X not connected" and stop.
3. x_account_health. Match the CONFIG accounts by id, never by label: a label changes when an account is renamed. An id no longer listed: skip that account and report "account not found: re-run setup"; never pick another account. Skip and report any account whose canSend is false, with its fix: the x_login_link URL for a blocked account, the PoliteReach Accounts page for a paused one.
4. Load the calendar tools (list events, get event). If they are missing: send nothing this run, skip steps 1 and 2, put every row of step 3 under NEEDS CHECK with its draft, and notify me "X bookings: calendar not connected, nothing sent". Never guess a booking.

STEP 1: FIND BOOKINGS
On the CONFIG calendar, list events from 7 days ago to 30 days ahead. Keep only booking events (cal.com, Calendly, or my booking link in the organiser, description, location or title). Ignore meetings with only my own team.
Pass every external attendee (name, email, eventId, startsAt and the full description) to x_match_contacts in one call.
- match: x_mark_booked with its handle, calendarEventId and meetingStart.
- ambiguous: do not mark. List under NEEDS CHECK with the candidates.
- none: ignore.
- ignored: marked not from X earlier. Skip; never list it.

STEP 2: CANCELLED OR MOVED
x_bookings with from set to 7 days ago. Look up each calendarEventId on the calendar.
- Gone or cancelled: x_mark_booking_cancelled.
- Start time changed: x_mark_booked with the same calendarEventId and the new time.

STEP 3: NUDGE OR CLOSE
x_booking_due with dueOnly true, and no nudgeAfterDays or closeAfterDays: the send guard always uses the default waits. Keep the rows whose accountId is a CONFIG account. For each row read the thread with x_conversation_history.
- they_wrote: never a nudge. If the thread shows they booked, find the event and x_mark_booked. If they changed their mind, NEEDS CHECK. Otherwise leave it to my replies routine.
- nudge_due: one line pointing back to what they said about the call, then my booking link on its own line. Never propose a time, never pitch.
- close_due: one line, no link: "Should I close this off, or still want a slot this week?" in the tone of the thread.
- waiting or done: nothing.
Max 2 short lines, no em dashes, no exclamation marks, no emojis, no "just checking in". The link is plain: PoliteReach adds nothing to it.
Right before sending, search the calendar again for each row by name and handle: anyone who booked since step 1 gets x_mark_booked and no message.
Send with x_send_reply, confirmSend true, calendarChecked true: the nudges in one call with bookingNudge true, the closes in another with bookingClose true. Each goes from the account holding that thread. PoliteReach refuses any row that is not due: drop the refused rows and call again once with the rest, never retrying a refused person. A job id means queued: never send it again.

If any tool returns a sign-in or blocked error, stop work on that account and report the x_login_link URL; never retry around it.

REPORT: booked, cancelled or moved, then NUDGED and CLOSED (or DRAFTS), each message in a code block with the handle and the account holding the thread, then NEEDS CHECK. Nothing happened: "No booking actions." and no notification. Otherwise notify me once with the counts and one line per NEEDS CHECK item.
```

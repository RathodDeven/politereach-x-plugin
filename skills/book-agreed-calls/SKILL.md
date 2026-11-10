---
name: book-agreed-calls
description: X (Twitter) booking follow-through for PoliteReach for X. Records who agreed to a call in their X DMs, matches calendar bookings to those people, notices cancelled or moved calls, sends the nudges and one polite close on each account's own pace from the account holding each thread, and after the call records whether it happened (no-show or call done) and offers the next follow-ups. Use when the user asks who agreed to a call on X, who booked, whether someone booked, to follow up on a booking link, how a call went or who did not show up, to change how fast booking nudges go out, or to put booking follow-up on a schedule.
---

# Get agreed calls booked

PoliteReach for X stores the booking state, matches names, works out who is due and refuses anything past the limits. It reads no calendar and writes no text: you read the user's calendar through their own calendar connector and write every word.

The pace is per account (see Settings below): by default they agreed, then one nudge 3 days after our last message, then the close 4 days after that, then done. `x_booking_due` never offers a message past the account's schedule, and each row carries it as `scheduleDays`.

## First: what must be connected

1. **The PoliteReach for X connector** (`x_*` tools). Missing: use the `get-started` skill.
2. **A calendar connector in the same app**, for parts B and C. Any of:
   - Google Calendar.
   - Microsoft 365 / Outlook calendar.
   - A Calendly or Cal.com connector that lists bookings.
   It needs its list-events and get-event (or search-events) tools, allowed to run. Check by loading them before part B.
   Missing: tell the user "To see who booked, connect your calendar (Google Calendar or Microsoft 365 / Outlook) in this app's connector settings, then ask again." Do part A meanwhile. Skip B and C: without the calendar you cannot rule out that someone already booked, and a nudge to them is wrong. Never guess.
3. **The booking link**, for nudges: it lives in the user's outreach skill, "(company)-politereach-x-skill". PoliteReach keeps no booking link and adds nothing to it (no name, no parameter). Missing: ask for it.

PoliteReach never connects to the calendar. You pass it only each external attendee's name, email, event id, start time and description.

## A. Who agreed but has not booked (no calendar needed)

1. `x_booking_due` (no `dueOnly`) lists everyone already recorded `interested`, with their stage.
2. Read the replies waiting on the user: `x_replies_to_answer` with `includeThread: true`, and `x_conversation_history` for a thread with no words captured.
3. List everyone who agreed to a call, with their own words about when.
4. Before recording anyone, search the calendar for them (part B tools): someone who already booked gets `x_mark_booked` and no message. For each one the user approves: `x_set_outcome` with `interested` and their words as the `note`. The clock starts the first time `interested` is recorded; recording it again does not restart it.
5. Anyone not yet sent the booking link: draft a short reply with the plain link for the user's approval, and send it with `x_send_reply` on their yes.

## B. Match the calendar (needs the calendar connector)

No calendar tools loaded: say so with the message above and stop after part A. Never guess whether someone booked.

1. **Find bookings.** On the calendar the bookings land on, list events from 7 days ago to 30 days ahead. Keep booking-tool events (cal.com, Calendly, or the user's booking link in the organiser, description, location or title). Ignore meetings with only the user's own team.
2. **Match.** One `x_match_contacts` call with every external attendee: `name`, `email`, `eventId`, `startsAt` and the full `description` (booking forms often hold their X handle). Pass only those fields: X's matcher takes no booking creation time.
   - `match`: `x_mark_booked` with the `handle`, `calendarEventId` and `meetingStart`. A booked call is `meeting_booked`, never `won`. The same event again changes nothing. The attendee's email is theirs: `x_set_contact_email` with it (skip the user's own team).
   - `ambiguous`: mark nobody; list the candidates for the user.
   - `none`: ignore. When the user says a booking is not from X: `x_ignore_calendar_event` with its `eventId` (or their `email` for every booking by them). Never mark one yourself because it did not match.
   - `ignored`: marked not from X earlier. Skip it; never report it.
3. **Cancelled or moved.** `x_bookings` with `from` 7 days ago, compared with the calendar.
   - Gone or cancelled: `x_mark_booking_cancelled`. They come back to `x_booking_due` a day later, nudge count kept.
   - New start time: `x_mark_booked` with the same event id and the new time.

## C. Nudge or close (only after part B ran)

1. `x_booking_due` with `dueOnly: true`. Read `truncated`. Drop rows from accounts the user did not ask you to cover. The waits are each account's own booking pace, the same one the send guard behind `bookingNudge` and `bookingClose` reads; `nudgeAfterDays` and `closeAfterDays` are refused (change the pace with `x_set_booking_settings`). If a follow-up chain is active for a person (`x_list_chains`), leave them to it.
2. Per row, from the thread (`x_conversation_history`):
   - `nudge_due`: one line pointing back to what they said about the call ("you mentioned thursday for a call"). If a message of ours after their last one already has the link (an earlier nudge), point to it ("can you grab a slot from the link above?") and never paste it again; otherwise the plain booking link on its own line. Never propose a time, never pitch.
   - `close_due`: one line, no link, e.g. "Should I close this off, or still want a slot this week?" in the tone of the thread.
   - `they_wrote`: never a nudge. If they booked, find the event and `x_mark_booked`. Otherwise leave it to the `answer-replies` skill (an X reply), or to `keep-warm-conversations` (an email or call they made, logged with `x_log_touch`).
   - `waiting` or `done`: nothing.
   - At most two short lines, no em dashes, no exclamation marks, no emojis, no "just checking in".
3. Right before sending, search the calendar once more for each person (name, then handle): people book minutes after they are listed. Booked: `x_mark_booked`, send nothing.
4. Each nudge or close goes from the account holding that thread.
5. Show the drafts. On a yes (or under a scheduled task's send mode): `x_send_reply` with `confirmSend: true`, `calendarChecked: true` and `replies`, `bookingNudge: true` for nudges, and a separate call with `bookingClose: true` for closes. A batch cannot mix the two. A refusal names its reason: relay it, never retry that person; resend the other rows once. A returned job id means queued: never send it again.

## D. After the call (needs the calendar connector)

1. `x_bookings` with `from` 7 days ago and `to` now: calls whose start has passed. Skip anyone already at `no_show`, `call_done` or a later stage (`x_contact_dossier` shows it in its `warm` block).
2. Per call:
   - Gone or cancelled on the calendar: part B, `x_mark_booking_cancelled`.
   - The calendar or booking tool marks them a no-show (Calendly and Cal.com can): `x_set_stage` `no_show` (`source: "routine"` in a scheduled task).
   - Otherwise ask the user how it went: `no_show`, or `call_done` with `note` (what was agreed), `nextStep`, `owner` and `dueDate` (YYYY-MM-DD). Set it with `x_set_stage`.
3. Offer that stage's chain from the user's outreach skill (`references/stage-templates.md`), written for this person. The `keep-warm-conversations` skill has the rules.
4. A deal agreed on the call is `x_set_outcome` `won`; the call alone is never `won`.

## Settings

The booking pace, per account: `x_set_booking_settings` with `sessionLabel` or `sessionId` and `nudgeScheduleDays`. Each number is days after OUR previous message to them (the first counts from when they agreed, or from our latest message since on any channel, an email logged with `x_log_touch` included). The last number is the close; every one before it is a nudge. [3, 4] (the default) = one nudge after 3 days, the close 4 days after it; [2, 5, 10] = a nudge after 2 days, another 5 days later, the close 10 days after that. 1 to 4 numbers, each 1 to 30, never shrinking. A broken schedule is refused, never trimmed. The booking link stays in the user's outreach skill: PoliteReach keeps none.

## Stop conditions

A sign-in or blocked error: stop work on that account and give the user the `x_login_link` URL. Never retry around it.

## Put it on a schedule

Follow `references/scheduled-task.md`.

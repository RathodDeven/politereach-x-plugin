---
name: book-agreed-calls
description: X (Twitter) booking follow-through for PoliteReach for X. Records who agreed to a call in their X DMs, matches calendar bookings to those people, notices cancelled or moved calls, and sends one nudge then one polite close from the account holding each thread. Use when the user asks who agreed to a call on X, who booked, whether someone booked, to follow up on a booking link, or to put booking follow-up on a schedule.
---

# Get agreed calls booked

PoliteReach for X stores the booking state, matches names, works out who is due and refuses anything past the limits. It reads no calendar and writes no text: you read the user's calendar through their own calendar connector and write every word.

The follow-through is fixed: they agreed, then ONE nudge, then one close, then done. `x_booking_due` never offers a third message.

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
   - `match`: `x_mark_booked` with the `handle`, `calendarEventId` and `meetingStart`. A booked call is `meeting_booked`, never `won`. The same event again changes nothing.
   - `ambiguous`: mark nobody; list the candidates for the user.
   - `none`: ignore. When the user says a booking is not from X: `x_ignore_calendar_event` with its `eventId` (or their `email` for every booking by them). Never mark one yourself because it did not match.
   - `ignored`: marked not from X earlier. Skip it; never report it.
3. **Cancelled or moved.** `x_bookings` with `from` 7 days ago, compared with the calendar.
   - Gone or cancelled: `x_mark_booking_cancelled`. They come back to `x_booking_due` a day later, nudge count kept.
   - New start time: `x_mark_booked` with the same event id and the new time.

## C. Nudge or close (only after part B ran)

1. `x_booking_due` with `dueOnly: true`. Read `truncated`. Drop rows from accounts the user did not ask you to cover. Pass no `nudgeAfterDays` or `closeAfterDays`: the send guard behind `bookingNudge` and `bookingClose` always uses the default waits, so a shorter wait lists people it then refuses. There is no booking settings tool.
2. Per row, from the thread (`x_conversation_history`):
   - `nudge_due`: one line pointing back to what they said about the call ("you mentioned thursday for a call"), then the plain booking link on its own line. Never propose a time, never pitch.
   - `close_due`: one line, no link, e.g. "Should I close this off, or still want a slot this week?" in the tone of the thread.
   - `they_wrote`: never a nudge. If they booked, find the event and `x_mark_booked`. Otherwise leave it to the `answer-replies` skill.
   - `waiting` or `done`: nothing.
   - At most two short lines, no em dashes, no exclamation marks, no emojis, no "just checking in".
3. Right before sending, search the calendar once more for each person (name, then handle): people book minutes after they are listed. Booked: `x_mark_booked`, send nothing.
4. Each nudge or close goes from the account holding that thread.
5. Show the drafts. On a yes (or under a scheduled task's send mode): `x_send_reply` with `confirmSend: true`, `calendarChecked: true` and `replies`, `bookingNudge: true` for nudges, and a separate call with `bookingClose: true` for closes. A batch cannot mix the two. A refusal names its reason: relay it, never retry that person; resend the other rows once. A returned job id means queued: never send it again.

## Stop conditions

A sign-in or blocked error: stop work on that account and give the user the `x_login_link` URL. Never retry around it.

## Put it on a schedule

Follow `references/scheduled-task.md`.

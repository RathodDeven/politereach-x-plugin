# Set up PoliteReach for X, start to finish

For a user who says "set me up" or wants everything ready in one go. Take one step at a time and finish it before starting the next. Say which step you are on. Skip a step only when it is already done, and say so.

## 1. The X account

Follow `get-started`: the connector, an X account connected by cookie import on https://app-x.politereach.com/accounts/login (the live browser sign-in is the fallback), then `x_verify_account` once. Stop here until at least one account verifies: nothing else works without one.

## 2. The outreach skill

Every routine writes in the user's voice and judges fit against their ICPs, so this comes before any routine.

- Ask whether they already have an outreach skill (named "(company)-politereach-x-skill" when PoliteReach built it). If they do, take its exact name, and check it has `references/stage-templates.md` and, in `references/company.md`, the booking link and the calendar the bookings land on (missing: ask those parts of the `outreach-skill-builder` interview now and add them). Then move on.
- Otherwise follow `outreach-skill-builder`: the interview, then the build. Note the skill's exact name: every routine attaches it. Its interview also asks the booking pace and writes the default follow-ups per deal stage.

## 3. The routines

All optional. Offer the six routines in one message, one line each, and ask which to set up:

1. Replies: drafts answers by default, or sends the clear ones and flags the rest; works best with a calendar connected (default every 2 hours, 10:00 to 20:00).
2. Outreach: keeps a running campaign fed from the Prospects pool within each account's DM cap today; drafts the sequences by default, or adds the people itself (default 09:00).
3. Bookings: checks who booked after agreeing to a call, then drafts the nudges and the close on each account's pace by default, or sends them; after a call, asks how it went. Needs a calendar connected in this app (default 11:20, 15:20 and 19:20).
4. Posts: writes tweets in the user's voice and shows drafts first by default, or schedules them (default planned Sunday and Wednesday at 10:45, tweets between 18:45 and 19:30).
5. Morning check: read-only; says each morning whether outreach is moving and what needs the user (default 08:30).
6. Warm follow-ups: every morning, lists the people already in conversation whose next touch is due (a follow-up chain step, a next-step date, someone who wrote last, someone gone quiet) with a draft for each by default, or sends the chain steps they already approved (default 09:30).

Set up only the ones they pick, one at a time: each has its own short interview, then creates its scheduled task. Use `answer-replies`, `keep-outreach-going`, `book-agreed-calls`, `schedule-posts` and `keep-warm-conversations`, each at its `references/scheduled-task.md`, and `get-started` at `references/morning-check.md`. If this app has no scheduled tasks, say so and offer to run a routine's TASK PROMPT once, by hand, in this conversation instead. None picked is fine: every routine can be run by hand later.

## 4. Wrap up

- List what now exists: the skill's name, and each task's name and next run.
- Remind the user to set every PoliteReach for X tool to always allow: nobody is there to approve a tool while a task runs.
- Offer to start outreach now: they paste handles or say who to find, and `start-outreach` stages the sequences for their yes.

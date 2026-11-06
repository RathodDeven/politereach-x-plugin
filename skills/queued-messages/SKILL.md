---
name: queued-messages
description: X (Twitter) queued DMs in PoliteReach for X. Reviews the DMs about to send and fixes or holds queued copy before it goes out. Use when the user asks what goes out on X today or this week, why a DM is waiting, spots wrong wording in a sequence, wants to change a line across a campaign, hold one message, stop a person, pause or resume a campaign, have one campaign's people go first, or retry a send that failed.
---

# Review and fix queued messages

Every person carries their own copy of the sequence text from the moment they were imported. Changing a strategy later changes nothing for people already in it: fix the queued steps instead. The worker reads a step's text at send time, so an edit is honoured even when the job is already queued.

## What goes out soon

`x_scheduled_steps` with `withinHours: 24` for today, `withinHours: 168` for this week (or a `campaignId`). Each row has who, which step, the full text, when it fires (`firesAt`), the sending account and its state, and why it waits (`holdReason`, `holdKind`). Read `jobKnown`: false means the queue could not be read in full.

Report per account: how many go out, the first few with times, and anything waiting with its reason in plain words. A step whose time already passed is overdue: say so. `holdKind: stalled` needs a person; `will_not_send` is someone on the do-not-contact list. `x_due_messages` lists only what is already late (a long list means the worker is behind).

## Change nothing without the user's request

- Wrong words for one or a few people: `x_edit_step` with the step ids (from `x_scheduled_steps` or `x_campaign_contacts`; `x_contact_dossier` shows the steps without ids), `confirmEdit: false` to preview, then `confirmEdit: true`. A sent step is the record of what they got and is refused.
- The same wrong sentence across a campaign: `x_bulk_edit_steps` with `campaignId`, `find` and `replace`. `apply: false` previews. Read `contactsUnmatched` (a non-zero number usually means the find text is not exactly what is stored) and `truncated` (walk on with the reply's `nextCursor`). It is NOT idempotent: never run the same apply twice.
- A message that should not go at all: `x_cancel_step` with the step ids, `confirmCancel: false` first. It cannot be undone, so if the message is still wanted, rewrite it with `x_edit_step` instead. Holding an opener closes that person's sequence.
- Stop everything for one person: `x_stop_contact` (`confirmStop`); `alsoSuppress: true` when they must never be contacted again.
- Confirm with `x_scheduled_steps` again afterwards.

## Pause, resume or prioritise a campaign

- `x_set_campaign_status` with `status` `paused` to pause, `active` to resume. A pause is NOT instant: queued jobs are held on a recheck, and a send already in flight still lands. A resume respects hours, pacing and the cap, so sends may not appear at once; read `accountHold` with its `fixUrl`.
- Several campaigns at once: `campaignIds` (up to 100) or `namePrefix`. Without `confirmChange` it previews; repeat with `confirmChange: true` (a `namePrefix` resume also with `expectCount`, the number previewed). A bulk resume never launches a campaign nobody launched.
- A wrong campaign name, ICP or `sendWindowMode`: `x_edit_campaign`. Its sending account never changes (an X thread belongs to a pair of accounts). Its strategy changes only while it holds nobody.
- One campaign's people first: `x_edit_campaign` with `priority: true`. It only reorders that account's openers; the cap, hours and pacing are unchanged. `priority: false` puts it back in turn.
- People in the wrong campaign: `x_move_contact`, `confirmMove: false` first. Same account only.

## Held or failed sends

- After a fix (a sign-in, a pause lifted): `x_retry_held` asks held sends to try again now. It cannot force a send: each still meets every gate.
- A sequence that failed for a reason that may pass (a network hiccup, a session since reconnected): `x_reset_contact`, `confirmReset: false` first. A failure about the person, or an attempt that may already have reached them, is refused: check the thread on X instead.
- A sign-in or blocked error: give the user the `x_login_link` URL. Never retry around it.

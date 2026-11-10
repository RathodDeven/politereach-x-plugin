---
name: keep-warm-conversations
description: X (Twitter) follow-up for conversations already under way in PoliteReach for X. Plans the next few touches for one person (a follow-up chain on X or email), records where each deal stands (agreed, booked, no-show, call done, proposal sent, trial, paid, closed), logs email, WhatsApp and call touches so nothing double-sends, and works the daily list of people whose next touch is due. Use when the user asks who to follow up with on X, what is due today, to follow up after a call, a no-show or a proposal, to chase someone who went quiet, to set or change a deal's stage, to save someone's email, to log an email or call, or to put warm follow-ups on a schedule. Not for new followers or tweet repliers nobody has messaged yet: those are warm leads, in `pipeline-check` and `start-outreach`.
---

# Keep warm conversations moving

A campaign ends after its last DM. Everything after that (a reply, a call, a proposal, a trial) has nothing scheduled, so it gets forgotten. This skill keeps those people moving. You write every word; PoliteReach for X holds the plan, the timing and the stop rules.

Two lists, never mixed up:
- **Warm leads** (`x_warm_leads`): new followers and people who replied to the user's tweets, nobody has DMed them yet. A list to pick from, in `pipeline-check` and `start-outreach`.
- **The warm queue** (`x_warm_due`, this skill): people ALREADY in conversation whose next touch is due.

## First: what must be connected

1. **The PoliteReach for X connector** (`x_*` tools). Missing: use the `get-started` skill.
2. **The user's outreach skill**, "(company)-politereach-x-skill", for the voice and the stage templates (`references/stage-templates.md`). Missing: offer the `outreach-skill-builder` skill; until then, write in the thread's own tone and ask before every chain.
3. **An email connector, only for email steps** (Gmail, Outlook or similar, with a send or draft tool). Without one, email steps come to the user as text to send themselves. Never suggest one to a user who follows up on X only.

## 1. Follow-up chains

A chain is the next 1-4 touches for ONE person from ONE account, inside or outside a campaign.

**When to offer one:**
- A warm thread went quiet and the next move is to wait on them.
- After a call (`call_done`), a no-show, a proposal, a trial start.
- After the booking follow-through closed with no booking, for a later check-in.
- Not while a campaign sequence still runs for them, and not instead of the booking nudges: someone who agreed to a call is recorded `interested` and the `book-agreed-calls` skill follows them up.

**Writing the steps:**
- Start from the stage's template in the user's outreach skill (`references/stage-templates.md`). No template: propose 2-3 steps.
- Write each step for THIS person from the thread (`x_conversation_history`, or `x_contact_dossier` for everything stored): their words, what was agreed, the next step. Every word final now: PoliteReach never writes text at send time, and has no placeholders.
- `delayDays` (1-60) = days after OUR latest message to them on any channel and any account (an email logged with `x_log_touch` included), then after the step before. Gaps strictly grow (for example 3, 7, 14). The last step is the close. A step is never due sooner than 2 hours after the chain is made.
- Each step: `channel` (`x` or `email`), `mode` (`send` or `draft`), `text`.
  - `send`: X only. PoliteReach reads the thread live and sends it inside the account's sending hours, daily DM cap and pacing, like a campaign step (a capped day moves it to the next one). Never send it by hand.
  - `draft`: waits in the warm queue (`x_warm_due`) for a yes: the user's in chat, or a warm routine the user set to send mode. Default for anything sensitive; tell the user a send-mode routine sends drafts too.
  - `email` is ALWAYS a draft (email with `send` is refused): PoliteReach never sends email. When due, you send it through the user's email connector on their yes, then `x_log_touch` (`channel: "email"`, `direction: "out"`).
- A draft step completes by itself once any message of ours reaches them after it fell due (an `x_send_reply`, or an email logged with `x_log_touch`), so a step sent by hand is never sent again.

**Create it:**
1. `x_create_followup_chain` with `handle`, `steps`, a short `label` (for example "after call") and `stage` when it comes from a stage template, `confirm: false`. The account is the one holding their thread; name it (`sessionLabel` or `sessionId`) only for someone none of the user's accounts has DMed. The dry run shows each step's due time and any warning.
2. Show every step: text, channel, mode, due date. On the user's yes, the same call with `confirm: true`.
3. One active chain per person per account. A second is refused; replace only on the user's yes (`replaceExisting: true` stops the old one).

**It stops by itself** when they write on any channel after it started (the X thread is also read live right before each send), a call is booked (`meeting_booked`), a final verdict is recorded (`won` or `not_interested`; `interested` does not stop it), they go on the do-not-contact list, the account is removed, a step reaches them only in part, or their handle now belongs to a different X account. A send that fails ends the chain; it is never retried. Someone whose thread was never read on that account may stop at the first send step if their message is the newest: the dry run warns about it.

Read chains with `x_list_chains` (default `active`; `stoppedBecause` says why one ended). Stop one with `x_cancel_chain` (`chainId`, or `handle`, narrowed to one account with `sessionLabel` or `sessionId`) and a short `reason` (echoed back, not stored).

## 2. Stages

Where each deal stands, separate from the campaign status. `x_set_stage` with `handles` (up to 200) and `stage`:

| Stage | Means |
|---|---|
| `agreed` | Said yes to a call, not booked yet |
| `booked` | A call is on the calendar |
| `no_show` | The call time passed and they did not join |
| `call_done` | The call happened. Pass `note` (what was agreed), `nextStep`, `owner`, `dueDate` (YYYY-MM-DD) |
| `proposal_sent` | A proposal or quote is out |
| `invited_not_activated` | Signed up for the user's product but never got started. Only for products with sign-ups; always set by hand |
| `trial` | On a trial |
| `paid` | Paying. Leaves the warm queue |
| `closed` | Done either way. Leaves the warm queue |

- `stage: null` clears it. `source: "routine"` when a scheduled task set it; default `manual`.
- Nothing sends because of a stage. A stage is not a verdict: record the outcome with `x_set_outcome` (`interested`, `meeting_booked`, `won`, `not_interested`) or `x_mark_booked` where one applies.
- Then offer that stage's chain from the user's outreach skill.

## 3. Other channels

- **An email address appears** (in a thread, a calendar booking, the user's mail): `x_set_contact_email` with `handle` and `email`. Only an address that is clearly theirs.
- **Every touch off X** (email, WhatsApp, call, other): `x_log_touch` with `handle`, `channel`, `direction` (`in` = they reached the user, `out` = the user reached them), `at` (ISO, default now) and a short `note` (what happened, not the email itself). X messages are never logged: they are read from the thread.
- An **inbound** touch is a reply on another channel: it stops their chains and their live campaign sequences, and counts as "they wrote" for the booking follow-through. It never marks them replied on X and records no verdict.
- An **outbound** touch counts as our latest message: chain and booking waits restart from it.

## 4. The daily warm queue

`x_warm_due` (all accounts, or one with `sessionLabel` or `sessionId`). `limit` goes up to 100 and there is no paging. `truncated` or `scanTruncated` true: say the list is incomplete ("N more due, not shown", from `total`) and never call it complete. `unreadable`: an unreadable list is not an empty one, never say "nothing due". It carries no thread: read `x_conversation_history` for each person you will write to.

Most overdue first. Work each row by its `reasons`:

- `chain_step_due`: `draftedStep` holds the step's text. `draftedStep.sendsItself` true: an X `send` step past its time (sending hours or a hold); PoliteReach sends it, so never send it by hand, just say when. Otherwise re-read the thread; still right: show it for the user's yes, then send (X: `x_send_reply` with `confirmSend: true`, from the account holding the thread; email: the user's email connector, then `x_log_touch` out). Something changed: rewrite it, show both.
- `next_step_date`: the stage's next step (`stageNextStep`) is due. Draft the touch that does it, or remind the user if it is not a message.
- `they_wrote_last`: their latest message on any channel is newer than ours. Draft an answer in the user's voice. An X reply is also in `answer-replies`: follow that skill's calendar check.
- `quiet`: a live stage, no chain, nothing either way for 7 days. Propose the stage's chain.

Each row also carries `lastTouch` per channel, the stage fields, `email`, `unansweredXRun`, `suggestedChannel` and its active chains.

- `unansweredXRun` = our X messages in a row with no answer. X has no read receipts: never say "unread" or "ignored", say "no answer to our last N messages".
- 2 or more with an email stored: `suggestedChannel` is `email`; suggest an email step. `emailMissing: true`: say "email (no address stored)" and ask whether the user has one.

## 5. Duplicate-send guard

`x_send_reply` and `x_send_dm_now` refuse the SAME words to the SAME person within 10 minutes, delivered or still queued. The refusal names when and the job holding it.

- A refusal means it already went or is queued. Check `x_job_result` or the thread; never resend.
- `force: true` only when the user explicitly wants the identical text sent twice. It skips this guard only, never the do-not-contact list, the booking rules or a hold.
- "Could not check for a duplicate": wait a minute and try once more; never `force` around it.

## Message rules

At most three short sentences, one idea per line. No em dashes, exclamation marks, emojis or filler ("just checking in", "circling back", "bumping this"). Each touch brings one new reason or one clear ask. Never re-pitch what the thread covered. Never invent a price, client, metric or date.

## Stop conditions

A sign-in or blocked error: stop and give the user the `x_login_link` URL. Never retry around it.

## Put it on a schedule

To work the warm queue every morning, follow `references/scheduled-task.md`.

---
name: outreach-skill-builder
description: X (Twitter) outreach voice for PoliteReach for X. Interviews the user about their company, ideal customers, writing voice, booking pace and warm follow-ups, then builds their own outreach skill, named "(company)-politereach-x-skill", that every PoliteReach for X workflow writes DMs with, and sets PoliteReach up to match (strategies, ICPs, do-not-contact list, sending hours, booking pace). Use when the user is new to PoliteReach for X, asks for X DMs written "in my voice", wants to set up or update their ICPs, offers, proof points, banned words or follow-up templates, or no X outreach skill exists yet.
---

# Build the user's X outreach skill

Every PoliteReach for X workflow writes in the user's voice and judges fit against the user's ICPs. Those facts belong to the user, so they live in a skill the user owns, built from an interview. Invent nothing. Build only once the user has answered.

If the `skill-creator` skill is available, use it to build and save the result. Otherwise write the files and tell the user how to add them as a skill in the app they use.

## Part 1: interview (four rounds, one message each)

Read `x_list_accounts` first so you can name the user's accounts back to them. Wait for answers after each round. Offer options to pick from where you can. Ask again about anything vague.

**Round 1, the business**
1. The company, and what it sells: one line per offer.
2. Two to five ICPs, in X terms: for each, the words in their bio, the kind of account (founder, operator, agency, creator), a follower range, the pain solved and which offer leads. And when someone fits two, which one wins.
3. Who is never contacted (competitors, students, job seekers, others). Help define a competitor by a test (for example: do they sell what the user sells, or would they buy it?), and ask whether peers who do the same work are partners or a pass. For specific people, ask for each one's X handle: only a handle can go on the do-not-contact list.
4. Proof the user can send: case studies, results, links, and when each fits.
5. The booking link, and the pricing a plain price question may be answered with (or "never quote prices").
6. Where their prospects are (country or region), so DMs land in the prospects' working day.
7. Which calendar their bookings land on (Google Calendar, Microsoft 365 / Outlook, Calendly, Cal.com, other, none), and which booking tool their link uses. Then check only THAT calendar's connector is connected in this app (its list-events tool loads). Not connected: tell them to connect it in this app's connector settings and set its tools to always allow. Never suggest a connector for a calendar they do not use. None: say booking checks and nudges will not run, and move on.

**Round 2, the voice**
8. Three to five real DMs of theirs that got replies, or ones they liked (they paste them).
9. How they write: length, tone, greeting, emoji or not, words they never use, and how casual to be with senior people.
10. Topics that always go to them instead of being answered (pricing beyond answer 5, contracts, legal, anything senior).

**Round 3, how they send**
11. Their PoliteReach X accounts (labels from `x_list_accounts`), whether each one is verified on X (`verified`; null means never read), and which offers or ICPs each handles. Default: every account handles everything. Someone who takes DMs only from verified accounts goes to a verified one.
12. Cadence. Default: an opener, a follow-up after 3 days, another after 7, and a close at 14 days. Five DMs is the most a sequence holds (first, followup_1, followup_2, followup_3, closer). Like and follow steps before the opener are opt-in, only if the user asks for them.
13. Booking follow-up pace: days after each message of ours, the last one the close. [3, 4: one nudge 3 days after the link, the close 4 days after it]

**Round 4, warm follow-ups**
14. Their default next touches per deal stage, used when a conversation goes quiet. Offer these defaults; they confirm, change or drop each. Every step: days after our previous message (gaps grow), X or email, sent by itself or drafted for them. [drafted for them]
    - `agreed`: the booking nudges cover the first days; after the close, one check-in [30 days].
    - `no_show`: a no-worries line with the link to pick a new time, one new reason, a close [1, 4, 10 days].
    - `call_done`: a recap with the agreed next step, one new reason or resource, a close [1, 5, 12 days; email when they gave an address].
    - `proposal_sent`: any questions on it, one proof point, a close [3, 7, 14 days].
    - `trial`: how setup is going with one tip, the one feature that matters to them, a note before the trial ends [2, 5, 10 days].
    - `invited_not_activated`: only if they sell a product people sign up for: the one step to get started, an offer to set it up together, a close [2, 5, 10 days].

## Part 2: build it

Name it "(company)-politereach-x-skill": the company name in lowercase, words joined by hyphens (for example acme-politereach-x-skill). Every PoliteReach for X routine attaches it by that exact name. The `-x` keeps it apart from a LinkedIn outreach skill the user may also have. Layout:

- `SKILL.md`: the loop and the hard rules below, and which reference file to read at which step. Keep it short: one home for each fact.
- `references/company.md`: offers, proof, links, pricing that may be quoted, the strategy name to use, the booking link and pace, and which calendar and booking tool the bookings use (routines read it from here).
- `references/icps.md`: an ordered table of signals (first match wins) with the tie-break, then per ICP: how to spot them on X (bio words, kind of account, follower range), their pain, the offer, the proof, who is NOT this ICP, the account that writes to them, and their opener, follow-ups and close.
- `references/writing.md`: voice, banned words, DM rules.
- `references/stage-templates.md`: from answer 14, per stage: when it applies, then each step (days, X or email, sent or drafted, and what it says, with the slots to fill from the thread in parentheses). A chain is always written out for the person before it is created; a slot never reaches PoliteReach unfilled.
- `references/objections.md`: per objection, what they say, what it really means, and the one line or question that moves it. Plus a step-down ladder (after a no, one other offer per reply, never one they already heard) and a graceful close.
- `references/learned-preferences.md`: empty to start, with a changelog.

Keep the ICP definitions inside the skill itself. PoliteReach stores only an ICP's name, as a label for comparing reply rates, and judges nobody against it.

### The loop the built skill follows

1. Read `x_account_health` every run, never from memory: which accounts can send, today's warm-up cap, and any hold.
2. `x_check_targets` on the handles first: drop anyone `suppressed` or with `outcome_recorded`, and tell the user who was already approached.
3. `x_prospect_context` for their bio and recent original tweets. It looks up anyone missing or stale by itself from the lookup credits; when `lookup.state` is `running`, call it again in a minute. Out of credits, write from what is stored or skip the person. Never research through an outside scraper or by opening their profile. For someone already in touch, `x_contact_dossier` holds the thread and history.
4. Match them to one ICP. If none fits, say so and name who to target instead. Never force a message.
5. Write the whole sequence at once: one text for every DM step of the strategy (`x_list_strategies` shows its steps; like and follow steps take no words). A row missing words for any DM step is refused at import. Five DMs at most: never write a sixth.
6. Show every word, the account and the campaign. Then either submit the texts with `x_submit_opener_drafts` (the user's brief and the tweet each opener refers to) for review in the Drafts panel on the PoliteReach Campaigns page, or import exactly the approved texts: `x_list_campaigns` first, then `x_create_dm_campaign` with `confirmCreate` false, showing its collisions, `rejectedRows`, `templateRejections`, `repeatedOpeners` and `accountHold`, then `confirmCreate` true on the user's yes. People from the Prospects pool go in with `x_add_prospects_to_campaign` instead (`approvedDraftIds` for approved drafts), dry run first the same way.
7. Feed a standing campaign by `campaignId` (one per account and offer, never one per batch or per day). Importing into a launched, active campaign sends: they join its drip (`start.startsWhen`), so say so before the yes.
8. Launch a new campaign only on a second yes: `x_launch_campaign` with `confirmLaunch` true. Say "launched" only when `openerQueued` is true, and hand over any `accountHold` `fixUrl`.
9. A conversation that went quiet, a call done, a proposal out: set the stage (`x_set_stage`) and offer that stage's chain from `stage-templates.md`, written for this person (`x_create_followup_chain`, preview, then the user's yes). It stops by itself if they write. Log email, WhatsApp and call touches with `x_log_touch`, and save an address they give with `x_set_contact_email`. The `keep-warm-conversations` skill has the rules.
10. Apply a one-off correction straight away. When the user makes the same choice two or three times, or says "always", propose the exact change to `learned-preferences.md` and write it only on a yes. Edit the old rule in place instead of adding one that contradicts it. Whenever the skill changes, give the user the whole skill folder back, never only the changed file.
11. Every few weeks, compare ICPs and strategies with `x_outreach_analytics` and `x_outcome_summary`. Say when a number rests on too few sends to trust.

### Hard rules for every DM

- An opener is under 300 characters, two or three short sentences. A follow-up is one or two short lines.
- The opener refers to ONE specific recent tweet of theirs, in the user's own words, never a long quote. With no usable tweet, use the bio or say less. No flattery: no "love your content", "big fan", "impressive".
- Sell the problem, not the service. End on a concrete ask, never "let me know".
- No link in the opener unless the user asked for one. Each follow-up brings one new reason (a proof point, a resource), never a bump.
- The close is a yes or no: "Should I close this off, or (the thing they would still say yes to)?"
- No em dashes, no exclamation marks, no AI words (leverage, streamline, delve, seamless), no "just checking in" or "circling back".
- Never invent a client, a number, a price or a date.
- Write each person's first name into their own text. Use `{firstName}`, `{name}`, `{handle}` or `{a|b}` only in one template shared across many people, and `{firstName}` only when every row carries a name (a row without one is refused). No other curly braces and no square-bracket placeholders: the import reads braces as a template.
- Someone who agrees to a call gets the user's plain booking link and is recorded with `x_set_outcome` as `interested`, so the booking follow-through picks them up. A booked call is `meeting_booked`, never `won`.
- Topics from answer 10 are drafted for the user, never sent.
- Check the exact text against these rules before showing or sending it.

Build nothing that tries to send faster. Each account's daily DM cap, warm-up ramp, pacing and sending hours live on the PoliteReach Accounts page, and every send waits for them.

## Part 3: set up PoliteReach to match

Show the user each of these and do it on their yes:

- `x_list_strategies`, then `x_create_strategy` for each cadence that has none: `confirmCreate` false to preview, `confirmCreate` true on the yes, the first step's delay 0. `like_recent`, `follow` and `followBackWaitDays` only if the user asked for them.
- `x_list_icps`, then `x_create_icp` for each ICP, the same way (`confirmCreate` false, then true).
- `x_set_booking_settings` on each account (`sessionLabel` or `sessionId`), with `nudgeScheduleDays` only if answer 13 changed the [3, 4] pace.
- `x_suppress_handles` for the people from answer 3 who came with a handle: `confirmSuppress` false to preview, then `confirmSuppress` true. PoliteReach then refuses them at every import and send. Competitors and other categories stay rules in the skill.
- `x_set_office_hours` on each account only if the user wants other hours. The account's `timezone` is where its proxy exits, never the prospects' zone. Prospects in several zones: the campaign's `sendWindowMode` (`prospect` or `both`).

When done, tell the user the skill's exact name, "(company)-politereach-x-skill" with their company filled in: the scheduled routines in this plugin attach it by that name. If they keep their outreach skill under another name, tell them that name instead, so the routines use it.

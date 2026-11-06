# Scheduled task: X outreach

Create nothing until the user has answered part 1.

The task writes with the user's outreach skill, named "(company)-politereach-x-skill". Find it among their skills first. If theirs has another name, ask which one it is and use that name. If they have none, offer to build it first (the `outreach-skill-builder` skill, or `x_setup_guide` step `outreach_skill`) and stop.

Also needed: the PoliteReach for X tools (`x_*`), a running campaign on each account it feeds, and the app's scheduled-tasks feature (claude.ai and Cowork: Scheduled; Claude Code: `/schedule`; ChatGPT: tasks). Without scheduling, offer to run the TASK PROMPT below once, by hand, in this conversation.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets, so the user can answer "defaults" or change only some. If their outreach skill already answers one, say what you took from it instead of asking. Ask their timezone if unknown.

1. Which PoliteReach X accounts should it cover? Use the labels from `x_list_accounts`; the task keeps each one's id with its label. [all of them]
2. Which running campaign should each account keep feeding? Use `x_list_campaigns`. [one per account; an account with none is skipped and reported]
3. Add or draft? Add: it writes each new person's sequence with your skill and adds them to the campaign itself. Draft: it submits the sequences for your review in the Drafts panel, and adds only the people whose drafts you approved there. [draft]
4. Who should it add? [people in your Prospects pool who fit an ICP in your skill]
5. When the pool runs low, find more with `x_source_prospects` from your monthly lookup credits? If yes: followers of which account, which X list, repliers to which tweet, or which search, and how many per run. [no: tell me the pool is low]
6. When should it run, in your timezone? [every day at 09:00]

## Part 2: create the task

Create a recurring scheduled task named "X outreach" at the chosen times, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's (company)-politereach-x-skill to the task and put its exact name in CONFIG, so every run loads it.

Draft mode remembers its batches in its own CONFIG, so each run must be able to update this scheduled task (the app's update-task tool). If this app gives a run no way to update its own task, say draft mode cannot run on a schedule here and offer add mode or a run by hand.

Then tell the user the task name and next run, and remind them to set every tool it uses to always allow: nobody is there to approve a tool while it runs. Choosing add mode here is the user's standing approval to add people. Offer one run now so they can watch it.

## TASK PROMPT

```text
Keep my X outreach going: add new people to the running campaigns via PoliteReach. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my company)-politereach-x-skill
- Accounts and the campaign each feeds: (account label and id: campaign name and id)
- MODE: (add | draft). In draft mode never import anyone whose draft I have not approved: submit each new sequence with x_submit_opener_drafts and report the batchId, so I approve them in the Drafts panel on the Campaigns page.
- Draft batches: (none yet). Draft mode only: the batchIds earlier runs submitted, with the account each feeds.
- Who: (my answer)
- Buy more: (no | the source, and at most N people per run)

SETUP
1. Load my outreach skill (named in CONFIG) and follow its voice and rules for every word. If it cannot be loaded, notify me "(task name) failed: skill not found" and stop.
2. Make sure the PoliteReach X tools (x_*) are loaded (if this app defers tools, search for them first). If they are missing, notify me "(task name) failed: PoliteReach X not connected" and stop.
3. x_account_health. Match the CONFIG accounts by id, never by label: a label changes when an account is renamed. An id no longer listed: skip that account and report "account not found: re-run setup"; never pick another account. Skip and report any account whose canSend is false, with its fix: the x_login_link URL for a blocked account, the PoliteReach Accounts page for a paused one.

0. EARLIER DRAFTS (draft mode only): for each batch on the Draft batches line, x_submit_opener_drafts with only that batchId. Drafts with status approved and no usedAt: I approved them, so unless that campaign is paused or completed (then report them), add them to that account's CONFIG campaign with x_add_prospects_to_campaign, the campaignId, their handles and approvedDraftIds, confirmCreate false first; drop every row in missingText, templateRejections or refusalSummary, then call it again with confirmCreate true for wouldImport only. Never draft any handle in these batches again: a newer draft withdraws my approval of the older one. Count the drafts still waiting (status draft, or approved and not yet added) as people waiting for their opener in step 1.
1. HOW MANY: per account, warmup.cap from x_account_health is today's DM cap. x_campaign_status on its campaign: when the people still waiting for their opener already cover two days of that cap, add nobody. Otherwise add at most one day's cap. A paused or completed campaign: add nobody, report it.
2. WHO: x_list_prospects (status new, sourceKind if CONFIG names one) and judge each against my skill's ICPs on the stored profile. In draft mode leave out every handle in a batch on the Draft batches line. Pool too small and Buy more is set: x_source_prospects with confirm false; if it is not refused for credits, call it again with confirm true and use the people it found on the next run. Pool too small and Buy more is no: say so in the report.
3. RESEARCH: x_prospect_context for the chosen handles. It looks up missing or stale ones by itself from the lookup credits; if lookup.state is running, call it again in a minute. Out of credits, write from the bio or skip the person.
4. WRITE each person's whole sequence with my skill: one text for every DM step of the campaign's strategy, the opener under 300 characters and tied to one recent tweet.
5. ADD (add mode): x_add_prospects_to_campaign with the campaignId and texts per person, confirmCreate false first; drop every row in missingText, templateRejections or refusalSummary, then call it again with confirmCreate true for wouldImport only. Report start.startsWhen and any accountHold with its fixUrl. Never make or launch a campaign.
6. DRAFT (draft mode): x_submit_opener_drafts with every DM step's text per person, then update this scheduled task so its Draft batches line lists the new batchId with its account. Drop a batch from that line only once every draft in it was added (usedAt set): a batch holding a draft I skipped stays, so that person is never drafted again. If the task cannot be updated, report the batchId as NOT REMEMBERED: the next run may draft those people again.

If any tool returns a sign-in or blocked error, stop work on that account and report the x_login_link URL; never retry around it.

REPORT per account: today's cap, people added (handle, ICP, the opener in a code block) or the draft batchId, people skipped with a short reason, and the pool left. Notify me once when anyone was added or drafted, the pool is low, or an account is held.
```

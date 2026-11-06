# Scheduled task: X posts

Create nothing until the user has answered part 1.

The task writes with the user's outreach skill, named "(company)-politereach-x-skill", or a content skill they name. Find it among their skills first. If theirs has another name, ask which one it is and use that name. If they have none, offer to build it first (the `outreach-skill-builder` skill, or `x_setup_guide` step `outreach_skill`) and stop.

Also needed: the PoliteReach for X tools (`x_*`), and the app's scheduled-tasks feature (claude.ai and Cowork: Scheduled; Claude Code: `/schedule`; ChatGPT: tasks). Without scheduling, offer to run the TASK PROMPT below once, by hand, in this conversation.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets, so the user can answer "defaults" or change only some. If their skill already answers one, say what you took from it instead of asking. Ask their timezone if unknown.

1. Which PoliteReach X accounts should it cover? Use the labels from `x_list_accounts`; the task keeps each one's id with its label. [all of them]
2. Which account posts on which weekday? [one tweet per weekday, your accounts taking turns]
3. What each account posts about. [what your skill says each person is known for]
4. Anchor each tweet to recent news in that space? [yes, from the last 72 hours, never invented]
5. Anything a tweet must never say or link, and the one call to action allowed. [never pitch or name your product; at most one soft "DM me" per batch]
6. Schedule them on their own, or show you drafts first? Scheduled ones you can still cancel with `x_cancel_scheduled_tweet` before they go out. [draft]
7. What time should tweets go out? [weekdays between 18:45 and 19:30, a different minute each time]
8. When should the planning task run, in your timezone? [Sunday and Wednesday at 10:45, each run filling the next few days]

## Part 2: create the task

Create a recurring scheduled task named "X posts" at the chosen times, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's (company)-politereach-x-skill to the task and put its exact name in CONFIG, so every run loads it.

Then tell the user the task name and next run, and remind them to set every tool it uses to always allow: nobody is there to approve a tool while it runs. Choosing schedule mode here is the user's standing approval to schedule tweets. Offer one run now so they can watch it.

## TASK PROMPT

```text
Write and schedule the next batch of my tweets via PoliteReach. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my company)-politereach-x-skill, or my content skill if I named one
- Grid: (account label and id per weekday, from x_list_accounts)  Post time: (window, my timezone)
- Topics per account: (my answer)  News-anchored: (yes | no)
- Never say: (my list)  Call to action: (my answer)
- MODE: (schedule | draft). In draft mode never call x_schedule_tweet: put the tweets in the report for my yes.

SETUP
1. Load my skill (named in CONFIG) and follow its voice and rules for every word. If it cannot be loaded, notify me "(task name) failed: skill not found" and stop.
2. Make sure the PoliteReach X tools (x_*) are loaded (if this app defers tools, search for them first). If they are missing, notify me "(task name) failed: PoliteReach X not connected" and stop.
3. x_list_accounts, then x_account_health. Match the CONFIG accounts by id, never by label: a label changes when an account is renamed. An id no longer listed: skip that account and report "account not found: re-run setup"; never pick another account. Skip and report any account whose canSend is false, with its fix: the x_login_link URL for a blocked account, the PoliteReach Accounts page for a paused one.

1. x_scheduled_tweets: skip any grid slot already filled. Fill the next unfilled slots in the coming 3 days. An account that cannot post: skip its slot and report it; never move its tweet to another account's day.
2. One tweet per slot, text only, at most 280 characters. If news-anchored, tie it to a real development from the last 72 hours in that account's space; never invent news, numbers or personal details, and never quote our own reply rates or volumes. At most one call to action per batch.
3. Humanize every tweet: no em dashes, no AI words, no "not X, it is Y", vary the openings.
4. Schedule each with x_schedule_tweet: the sessionId of its exact account, scheduledFor an ISO time with my offset at a varied minute inside the window, confirmSchedule true. A reply with accountHold or sendHold means that tweet would not post now: list it in the report with its fixUrl, and never move it to another account. A reply with accountHoldUnreadable or sendHoldUnreadable means the check did not answer: list that tweet as not known to post, never as scheduled to post. Never x_post_tweet: nothing publishes now. Then x_scheduled_tweets to confirm the times.

If any tool returns a sign-in or blocked error, stop work on that account and report the x_login_link URL; never retry around it.

REPORT: for each tweet the date, account, time, the news it is tied to and the full text, plus any skipped slot and why, so I can cancel one before it goes out. Notify me once when tweets were scheduled or drafted.
```

# Scheduled task: X morning check

A read-only routine: each morning it says whether X outreach is moving and what needs the user. It writes nothing and needs no outreach skill.

Create nothing until the user has answered part 1.

Also needed: the PoliteReach for X tools (`x_*`), and the app's scheduled-tasks feature (claude.ai and Cowork: Scheduled; Claude Code: `/schedule`; ChatGPT: tasks). Without scheduling, offer to run the TASK PROMPT below once, by hand, in this conversation.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets, so the user can answer "defaults". Ask their timezone if unknown.

1. Which PoliteReach X accounts should it cover? Use the labels from `x_list_accounts`; the task keeps each one's id with its label. [all of them]
2. Tell you every morning, or only when something needs you? [only when something needs me]
3. When should it run, in your timezone? [every day at 08:30]

## Part 2: create the task

Create a recurring scheduled task named "X morning check" at the chosen time, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word.

Then tell the user the task name and next run, and remind them to set every tool it uses to always allow: nobody is there to approve a tool while it runs. Offer one run now so they can watch it.

## TASK PROMPT

```text
Morning check: is my X outreach moving, and does anything need me? Nobody is watching this run: never ask a question. Read-only: never send, import, launch, pause or stop anything.

CONFIG
- Accounts: (label and id of each, from x_list_accounts)
- Report: (every morning | only when something needs me)

1. Make sure the PoliteReach X tools (x_*) are loaded (if this app defers tools, search for them first). If they are missing, notify me "X morning check failed: PoliteReach X not connected" and stop.
2. x_operations_health: the verdict in one line. sending, queue_empty, outside_office_hours and daily_cap_reached are healthy: for the last two, one information line saying when sending opens or the cap resets, never something that needs me. Any other verdict: lead with the cause and the fix. If it is account_blocked, give me the x_login_link URL; never retry anything. If it is sending_held, report it; never release it.
3. x_account_health for the CONFIG accounts, matched by id (a label changes when an account is renamed; an id no longer listed: report "account not found: re-run setup"): canSend, today's warm-up cap, the sending window, and sendHealth.degraded (it clears itself; signing in again does not help).
4. x_daily_digest: blockedAccounts, sessionsAtRisk (sign them in before they die) and anything that failed.
5. x_scheduled_steps: what goes out today, and why anything is waiting.
6. x_replies_to_answer: how many people are waiting on me, and the longest wait.
7. Only if the verdict points at the machine (an unowned browser, sends failing on every account): x_host_health, and quote loadPerCpu and browsers.

REPORT: one line per account (can send, cap today, hours), then what needs me, each with its fix. With Report set to only when something needs me, notify nothing when the verdict is healthy, every account can send and nothing is owed.
```

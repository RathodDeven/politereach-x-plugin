---
name: schedule-posts
description: X (Twitter) tweets through PoliteReach for X. Writes tweets in the user's voice and schedules them on their own X accounts, or posts one now on an explicit request. Use when the user wants a tweet written, posted now or later, a content calendar for one or more X accounts, to see or cancel scheduled tweets, or to put tweet planning on a schedule.
---

# Plan and schedule tweets

The user's approval is the gate: never schedule or post text the user has not approved (or a scheduled task they set up to do it). Tweets are text, at most 280 characters. `imagePath` takes only a file already on PoliteReach's host; there is no image upload from chat.

## Steps

1. **See what is already queued.** `x_scheduled_tweets`. Skip any slot already filled. A failed tweet carries its reason; a failed post is never retried by itself. It is also the only post history: a tweet posted now leaves no record.
2. **Know the plan.** Which account posts on which day, what each account posts about, the posting window, anything a tweet must never say, and the one call to action allowed. Take these from the user's outreach or content skill, or ask. Defaults: one tweet per weekday with the accounts taking turns; never pitch or name the product; at most one soft "DM me" per batch.
3. **Write.** One tweet per slot, at most 280 characters. If anchoring to news, use a real development from the last 72 hours in that account's space; never invent news, numbers or personal details. Humanize: no em dashes, no AI words, no "not X, it is Y", varied openings.
4. **Show every tweet** with its account, date and time. Change anything the user asks.
5. **On a yes:**
   - Later: `x_schedule_tweet` with the `sessionId` of its exact account (from `x_list_accounts`), `text`, `scheduledFor` (an ISO time with the user's offset, a varied minute inside the window) and `confirmSchedule: true`. A time in the past is refused.
     - A reply with `accountHold` or `sendHold`: the tweet is saved but would not post now (the account is paused, blocked or signed out, or all sending is held). Tell the user and hand over the `fixUrl`. Never move the tweet to another account.
     - A reply with `accountHoldUnreadable` or `sendHoldUnreadable`: that check did not answer. Never say "will post".
   - Now: `x_post_tweet` with `confirmPost: false` to preview, then `confirmPost: true`. Only when the user explicitly wants it out right now.
6. **Confirm.** `x_scheduled_tweets` again and check the times.

To cancel a tweet that has not gone out: `x_cancel_scheduled_tweet` with its `scheduledPostId` and `confirmCancel: true`. A posted tweet cannot be taken back this way.

## Report

For each tweet: date, account, time, the news it is tied to, and the full text, plus any skipped slot and why.

## Stop conditions

A sign-in or blocked error: stop work on that account and give the user the `x_login_link` URL. Never retry around it.

## Put it on a schedule

Follow `references/scheduled-task.md`.

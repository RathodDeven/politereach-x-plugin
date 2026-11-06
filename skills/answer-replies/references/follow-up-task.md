# One-time follow-up for a warm X thread

For one person whose thread has nothing scheduled: they replied, a call happened, a proposal went out, or you are waiting on them. The run happens days later with no memory of this chat, so the prompt must carry everything.

## Part 1: settle it with the user

1. When: about 7 days after our last message, at a morning hour in the user's timezone, unless they named a date. If a follow-up task for this person already exists, update it instead of making a second one.
2. Send or draft. [draft]
   - Draft: the run writes the message and sends nothing.
   - Send: only the exact text the user approves now, stored word for word in the prompt. Text written during the run is never sent.

## Part 2: create the task

Create a one-time scheduled task named "X follow-up: (name)" at that time, with the TASK PROMPT below, every field filled. Tell the user when it runs. Remind them to set the PoliteReach for X tools to always allow when it is in send mode.

## TASK PROMPT

```text
One X follow-up via PoliteReach. Nobody is watching this run: never ask a question, decide and act within MODE.

PERSON
- Name: (name)
- Handle: (their @handle)
- Sending account: (the label and id of the account holding their thread, from x_list_accounts)
- What we offer them: (one line)
- Skill: (my company)-politereach-x-skill. Load it first and follow its voice; if it cannot be loaded, notify me "(task name) failed: skill not found" and stop.

THE THREAD SO FAR
(the thread in a few lines, oldest first, with dates: what was said, what was agreed, what was sent, who waits on whom)
Our last message: (its words). Anything from them after it is new.
Corrections I made while drafting: (list, or none)

MODE: (send | draft)
Approved text (send mode only, sent word for word): (text)

STEPS
1. Make sure the PoliteReach X tools (x_*) are loaded. If they are missing, notify me "(task name) failed: PoliteReach X not connected" and stop.
2. Read the thread live: x_read_conversation with the handle and the sending account's sessionId. If it errors or answers errored true, the thread is unknown, never empty: in send mode send nothing, report "could not read the thread, nothing sent" and stop. In draft mode, draft from x_conversation_history and say the draft rests on the stored thread.
3. If they agreed to a call anywhere in the thread, search my calendar for them before deciding (name, then handle). Booked: send nothing and report it; in send mode also record it with x_mark_booked. Calendar not connected: send nothing to someone who agreed to a call; report it.
4. Decide on the newest message:
   a. They wrote after our last message: send nothing. Report their message word for word and a suggested reply.
   b. They said no or asked us to stop: send nothing. Report it.
   c. Nothing new: in send mode, x_send_reply with the handle, the approved text and the sending account's sessionId, confirmSend true, once. A job id means queued: never send it again. In draft mode, write the follow-up in my voice and report it.

MESSAGE RULES (hard): max 3 short sentences, no em dashes, no exclamation marks, no emojis, no "just checking in" or "circling back", never re-pitch what the thread already covered, never invent a fact.

If a tool returns a sign-in or blocked error, stop and report the x_login_link URL; never retry around it.

REPORT: which case (a, b or c, or why nothing was sent), their newest message, and the message sent or drafted in a code block. Notify me once.
```

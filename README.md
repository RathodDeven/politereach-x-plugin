# PoliteReach for X: plugin for Claude, ChatGPT and Codex

Run X (Twitter) DM outreach from your own X accounts by talking to Claude. Claude finds and researches people, writes every DM and tweet in your voice, and decides what to answer. [PoliteReach](https://politereach.com) does the X work on its servers: it sends your approved DM sequences inside each account's own daily cap, warm-up, pacing and sending hours, checks for a reply before every follow-up, reads threads and posts scheduled tweets.

The plugin has two parts:

- **A connector** to the PoliteReach for X MCP server at `https://mcp-x.politereach.com/mcp`. You sign in with your PoliteReach account (OAuth). No API key is stored in this plugin.
- **Skills** that teach Claude the PoliteReach for X workflows: what to read first, what to ask you, and when to stop and wait for your approval.

You need a PoliteReach account with at least one X account connected.

## Install

### Claude Code

```text
/plugin marketplace add RathodDeven/politereach-x-plugin
/plugin install politereach-x@politereach-x
```

Then run `/mcp`, choose `politereach-x` and sign in.

### claude.ai, Claude Desktop and Cowork

1. Go to **Customize > Plugins > Add > Add marketplace**, choose **Add from a repository**, and enter `RathodDeven/politereach-x-plugin`.
2. Install **PoliteReach for X**.
3. Open the plugin's **Connectors** tab and connect it with **Continue with your PoliteReach account**.
4. In **Manage marketplaces**, turn on **Sync automatically** so new skill versions arrive by themselves.

The plugin is not listed in Anthropic's plugin directory yet. Until it is, use the marketplace above.

Then say "Set me up on PoliteReach for X" to go through your X account, your outreach skill and the routines.

### Connector only

To use the tools without the skills, add a custom connector with the URL `https://mcp-x.politereach.com/mcp` and sign in. The `x_setup_guide` tool then serves the same setup and routine instructions the skills carry.

### ChatGPT and Codex

The same skills and connector also ship in OpenAI's plugin format ([Agent Plugins](https://agent-plugins.org/)): `plugin.json` and `mcp.json` at the repository root, with the shared `skills/` folder. One set of skills serves Claude, ChatGPT and Codex. PoliteReach for X is not in OpenAI's plugin directory yet.

**ChatGPT today: add the connector by URL.** This gives ChatGPT the PoliteReach for X tools without the skills.

1. Open **Settings > Security and login** and turn on **Developer mode**. In a Business, Enterprise or Edu workspace, an admin may have to allow it first.
2. Go to [chatgpt.com/plugins](https://chatgpt.com/plugins) and select **+**.
3. Name it `PoliteReach for X`. Under **Connection**, enter `https://mcp-x.politereach.com/mcp`.
4. Create it, then sign in with your PoliteReach account when asked (OAuth). Steps from [Connect to ChatGPT](https://developers.openai.com/plugins/deploy/connect-chatgpt).

What works depends on your ChatGPT plan, per [OpenAI's help center](https://help.openai.com/en/articles/12584461-developer-mode-and-mcp-apps-in-chatgpt):

- **Business, Enterprise and Edu:** full tools, including sending (beta). On Business, only workspace admins and owners can use developer mode.
- **Pro:** read-only tools. ChatGPT can report on your pipeline and replies, but cannot send or post.
- Custom connectors work on the web, not in the mobile apps.

**ChatGPT Business or Enterprise: import this repository.** A workspace admin can import it under **Workspace settings > Plugins > Import marketplace**. That brings the skills as well as the connector. ChatGPT reads `.claude-plugin/marketplace.json` from this repository and syncs daily ([OpenAI help](https://help.openai.com/en/articles/20001504-importing-and-syncing-plugin-marketplaces-from-github)).

**Codex:**

```text
codex plugin marketplace add RathodDeven/politereach-x-plugin
```

Then install **PoliteReach for X** from the plugin list and sign in. Codex reads `.agents/plugins/marketplace.json` ([Codex plugins](https://developers.openai.com/codex/plugins/build)). This path has not been tested with Codex yet.

**What the skills do there.** The skills work the same way once the tools are connected. Two notes:

- The routine skills (`answer-replies`, `keep-outreach-going`, `book-agreed-calls`, `schedule-posts`, `keep-warm-conversations`, and the morning check in `get-started`) ask your app to create a recurring scheduled task. If your app cannot schedule tasks, run them by hand.
- `book-agreed-calls` needs a calendar connector in the same app (Google Calendar, or Microsoft 365 / Outlook calendar) with its list-events and get-event tools allowed. Without it, it records who agreed but cannot tell who booked, so it sends no nudges.

## Skills

| Skill | What it does |
|---|---|
| `using-politereach` | The overview: what to connect for each job (an X account, credits, a calendar for bookings, scheduled tasks), which skill does what, and every tool with its safety rule. |
| `get-started` | Connects the connector and your X account (cookie import by default), checks the account really works, and reads back its own cap, warm-up and hours. Also the read-only morning check. |
| `outreach-skill-builder` | Interviews you about your company, ICPs and writing voice, then builds your own outreach skill, "(company)-politereach-x-skill", that every other workflow writes with. |
| `start-outreach` | From handles, a source, warm leads or a second attempt: researches each person, writes the whole DM sequence, stages it for your review and launches on your yes. |
| `keep-outreach-going` | Keeps a running campaign fed from your Prospects pool, a day's cap at a time. Can run on a schedule. |
| `answer-replies` | Finds every reply waiting on you, drafts answers, sends the ones you approve, and records how each conversation went. Can run on a schedule. |
| `book-agreed-calls` | Matches bookings on your calendar to the people who agreed to a call, follows up on the pace you set (nudges, then one polite close), and after the call asks how it went. Can run on a schedule. |
| `keep-warm-conversations` | Plans the next few touches for people already in conversation (agreed, no-show, call done, proposal sent), records where each deal stands, logs email, WhatsApp and call touches, and works the daily list of who is due. Can run on a schedule. |
| `schedule-posts` | Writes tweets in your voice and schedules them on your accounts. Can run on a schedule. |
| `pipeline-check` | Daily digest, campaign progress, outcomes, the follow funnel, warm leads, credits and your do-not-contact list. |
| `queued-messages` | Shows what goes out soon and fixes or holds queued wording before it is sent. |

## How it behaves

- **You approve before anything is sent.** Every tool that sends, imports, posts or writes previews first. A scheduled routine acts only in the mode you chose when you set it up, and every routine drafts by default.
- **Text is written ahead.** PoliteReach sends the words you approved and stored; it runs no model and never writes messages on its own.
- **Limits belong to each account.** Each X account has its own daily DM cap, warm-up ramp, pacing and sending hours, set on the PoliteReach Accounts page. Claude reads them and never raises them on its own.
- **Follows and likes are opt-in public steps.** A strategy may like someone's recent tweet or follow them before the first DM. Those steps never reply to or quote anything.
- **Warm leads are a list only.** New followers and people who reply to your tweets are shown to you; nothing DMs them unless you pick them.
- **No X passwords or cookies in chat.** You connect an account by importing its cookies on PoliteReach's own page (https://app-x.politereach.com/accounts/login), or with the live browser sign-in there. Never paste them into a chat.
- **Your own X accounts only.** The do-not-contact list is checked before every import and every send.

## Data and privacy

This plugin contains only instructions and a connector address. It runs no code on your machine. When you use it, Claude sends the following to PoliteReach at `mcp-x.politereach.com`:

- X handles of people you ask about, and the DMs and tweets you approve.
- Your decisions on replies (outcomes, notes).
- For `keep-warm-conversations` only: follow-up steps you approve, deal stages and notes, a person's email address when you save one, and a one-line note of each email, WhatsApp or call touch you log. PoliteReach never connects to your mailbox and never sends email.
- For `book-agreed-calls` only: the name, email, event id, start time and description of external attendees on your calendar booking events, read by Claude through your own calendar connector, so PoliteReach can match them to the people you messaged. PoliteReach never connects to your calendar.

Profile and tweet lookups and prospect sourcing use lookup credits from your workspace's monthly allowance.

- Privacy policy: https://politereach.com/privacy/
- Terms of service: https://politereach.com/terms/

## Support

- Contact: https://politereach.com/contact/
- Plugin issues: https://github.com/RathodDeven/politereach-x-plugin/issues

## License

The files in this repository are MIT licensed. See [LICENSE](LICENSE). The PoliteReach service is covered by its own terms.

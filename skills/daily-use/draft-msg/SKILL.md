---
name: draft-msg
description: "Draft a short, copy-paste-ready message summarizing what was discussed in the current conversation. For investigations, includes root cause, reason and the analysis steps. Use when asked to: draft a message, write a summary to send, summarize this for Slack/Jira/a teammate, or write up the findings."
---

# Draft Message

Summarize the current conversation into a message the user can paste straight into Slack, Jira or a chat.

## What to include

1. **Summary** — what was discussed and the outcome (decision, fix, answer, or open question).
2. **If it was an investigation** (debugging, incident, "why did X happen"), also include:
   - **Root cause** — the specific thing that broke or caused the behavior
   - **Reason** — why it happened (the underlying condition, change or trigger)
   - **Analysis steps** — the key steps taken to reach the conclusion, in order, with the evidence each one produced (command, query, log line, metric). Skip dead ends unless they rule out a likely suspect.
   - **Fix / next steps** — what was done or still needs doing, if known
3. Only include facts established in the conversation. Do not invent details; if something is still unconfirmed, say so.

## Style

- Concise and straightforward: short sentences, bullet points, no filler or preamble.
- Lead with the conclusion, then supporting detail.
- Use the user's terminology and names (services, envs, tickets) exactly as they appeared.
- Write in the language the user has been using unless they ask otherwise.
- If the user named an audience or channel, adjust the level of detail to it.
- If the message is to a user (a client, end user or anyone outside the team), keep it casual, concise and clear: write like a friendly reply, not a formal report. Use plain words, explain jargon or skip it, and leave out internal detail they don't need (e.g. detailed analysis steps). Avoid stiff phrasing like "Please be advised" or "We would like to inform you".

## Output format

Put the whole message in **one** fenced code block so it can be copied without formatting breaking:

- Use a 4-backtick fence (` ```` `) as the outer fence, so any inner triple-backtick snippets don't close it early.
- Inside, use plain text with simple markup that pastes cleanly: `-` bullets, `*bold*` (Slack style), inline `code`. No markdown headings (`#`), tables, or HTML.
- Keep links as raw URLs.

Output nothing else besides at most one short line before the block (e.g. noting any assumption). No commentary after it.

### Template (investigation)

````
*<Issue title>*

*Root cause:* <one or two sentences>
*Reason:* <why it happened>

*Analysis:*
1. <step> — <finding>
2. <step> — <finding>
3. <step> — <finding>

*Fix / next steps:*
- <action>
````

### Template (general discussion)

````
*<Topic>*

- <key point / decision>
- <key point>

*Next steps:*
- <action, owner if known>
````

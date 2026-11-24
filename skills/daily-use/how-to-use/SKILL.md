---
name: how-to-use
description: "Explain how the thing discussed in the current conversation (a tool, command, feature, config, concept or pattern) is used in practice, with one simple, concrete, step-by-step example. Use when asked: how do I use this, how is this used in practice, show me an example, walk me through it, give me a practical example."
---

# How To Use

Show how the thing we've been talking about is used in real life, through one simple worked example.

## 1. Identify the subject

- Work out the subject from the conversation: the tool, command, feature, config option, concept or pattern the user is asking about. If the user named one, use that.
- If several things were discussed and it's unclear which one is meant, ask in one line before going further.

## 2. Fill gaps in context

Use what's already in the conversation first. Get more context only when it's needed to make the example correct:

- **In a repo:** read the relevant code, config or docs (README, `--help`, existing usages via grep) so the example matches how it's actually used here.
- **External tool/library:** check its official docs for the current syntax and flags rather than relying on memory.
- **Internal/team topic:** search Slack, Jira or Confluence for how the team uses it.

Don't invent flags, fields or behavior. If something can't be confirmed, say so.

## 3. Pick one simple, realistic scenario

- One scenario only: the most common, everyday use case.
- Small enough to follow end to end; skip edge cases and advanced options.
- Make it concrete: real-looking names, values and commands (prefer ones from the user's own context, e.g. their services, envs, repo paths), not `foo`/`bar`.

## 4. Write the walkthrough

Use this structure:

```
**What it's for:** <one sentence: the problem it solves>

**Scenario:** <one or two sentences: the situation in the example>

**Steps**
1. <action> — <why this step>
   <command / code / config snippet>
   <what you should see, if useful>
2. ...

**Result:** <what you end up with, and how to check it worked>

**Gotchas:** <1–3 common mistakes or caveats, only if they matter>
```

## Style

- Concise and straight to the point: short sentences, no preamble.
- Each step does one thing and says why, so the user understands rather than just copies.
- Keep snippets minimal and runnable as written (or clearly mark placeholders like `<your-bucket>`).
- Write in the language the user has been using.
- Output as plain markdown in the chat. Don't create files or run commands that change anything; the example is for explanation.

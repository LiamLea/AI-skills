---
name: weigh-points
description: "Objectively judge which point in a discussion makes more sense, whether it's other people's discussion or one the user is part of. If the user's own point is wrong, stop and correct them; if it is right, give strong arguments to back it up. Use when asked: which point makes more sense, who is right here, weigh in on this discussion, does my point make sense, am I wrong, back me up, check my argument."
---

# Weigh Points

Give an honest, objective verdict on which point makes more sense. Never side with anyone by default, including the user.

## 1. Identify the points

- From the conversation (or a pasted Slack thread, PR comment, doc, etc.), work out each point: what is claimed and why.
- Restate each point in one sentence, in its strongest fair form (steelman, not strawman).
- Work out whether one of the points is the user's own:
  - **Other people's discussion** (the user isn't a party) → steps 2, 3, then 4.
  - **The user has a point in it** → steps 2, 3, then 5. If the user states a point with no other side, the opposing view is "the user's point is wrong".
- If it's unclear what is being claimed, or which point is the user's, ask in one line before going further.

## 2. Check the facts

Base the verdict on evidence, not on who said it:

- **In a repo / infra:** read the relevant code, config, docs or run read-only checks to confirm what is actually true.
- **External tools / tech:** check the official docs rather than relying on memory.
- **Internal context:** search Slack, Jira or Confluence for prior decisions or constraints.

Separate what is **fact** (verifiable), what is **trade-off** (depends on priorities), and what is **opinion**. If something can't be confirmed, say so.

## 3. Judge objectively

Weigh each point on:

- **Correctness:** are its facts and logic right?
- **Context fit:** does it fit the actual constraints (scale, team, cost, timeline, risk)?
- **Consequences:** what happens if we follow it, and what does it cost if it's wrong?

Decide which point makes more sense. If it's a genuine trade-off (both valid under different priorities), say so plainly, state which priority favours which point, and which one fits this context better.

Don't soften a verdict to be agreeable, and don't invent objections to look balanced.

## 4. Other people's discussion: give the verdict

- Lead with which point makes more sense, in one line.
- Explain why, with evidence: what the stronger point gets right and where the weaker one fails.
- Note anything valid in the weaker point, and what would change the verdict (e.g. different scale or priorities).

## 5. User's own point

### 5a. Doesn't make sense: stop and correct

- Lead with it directly: "Your point doesn't hold up, because …"
- Name the specific flaw: wrong fact, faulty logic, missing context, or a wrong assumption.
- Give the correct view and the evidence for it.
- Acknowledge any part of the user's point that is still valid.
- Don't give the user arguments to defend the wrong point.

### 5b. Makes sense: back it up

- Lead with it directly: "Your point holds up."
- Give 2–4 strong arguments, strongest first. Each one should be:
  - concrete (facts, numbers, docs, code, real examples, prior incidents), not generic;
  - tied to the actual context of the discussion.
- Pre-empt the best counterargument from the other side and answer it.
- Optionally add a short, copy-paste-ready reply the user can send.

## Output format

```
**Verdict:** <which point makes more sense / your point holds up / doesn't hold up / trade-off — one line>

**Points**
- <who>: <one-sentence steelman>
- <who>: <one-sentence steelman>

**Why:** <reasoning with evidence (4), correction (5a) or supporting arguments (5b)>

**Counterargument & answer:** <5b only>

**Suggested reply:** <optional, 5b only>
```

## Style

- Concise and straight to the point: no preamble, no hedging.
- Honest over agreeable: being right matters more than making anyone feel right.
- Cite evidence (file paths, doc links, data) where you have it.
- Write in the language the user has been using.
- Output as plain markdown in the chat. Don't change files or post anything.

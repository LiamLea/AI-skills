---
name: pr-review
description: "Review a GitHub PR: explain what/why/how, flag explicit errors and structural concerns without nitpicking, and approve simple PRs once the user gives the go-ahead. Use when asked to: review a PR, look at a PR, check a pull request, or given a GitHub PR URL/number."
---

# PR Review

Keep everything concise and straight to the point. No preamble, no filler. The user will ask if something is unclear.

## 1. Gather

1. `gh pr view <pr> --json title,body,author,baseRefName,headRefName,files,additions,deletions` — metadata
2. `gh pr diff <pr>` — the full diff
3. Follow links in the PR body (Jira, Slack, issues) only if needed to understand the why
4. Read surrounding code in the repo when the diff alone doesn't show how the change fits

## 2. Context

Explain the PR in three short parts:

- **What** — what changes, in a sentence or two
- **Why** — the problem or goal behind it
- **How** — the approach taken, key files/pieces involved

## 3. Review

- **Don't be picky.** No style nits, naming bikesheds, or personal preferences.
- **Find explicit errors.** Bugs, broken logic, wrong config values, missing pieces that will fail, security holes, things that won't work as intended.
- **Leave optional things alone.** If something could be done differently but works fine, don't mention it.
- **Raise structural concerns.** If the result looks fine but the structure, approach, or placement feels off (wrong layer, duplicated logic, hard-to-maintain design, surprising side effects), definitely flag it.

List each finding with `file:line`, what's wrong, and why it matters. If there are no findings, say so in one line.

## 4. Verdict

End with one of:

- **Looks good — ready to approve.** For very simple PRs or ones you're fully confident in. Ask the user for the green light.
- **Needs changes.** Summarize the blocking issues in one or two lines.
- **Needs discussion.** Structural concerns worth talking through before approving.

## 5. Approve

Only after the user explicitly gives the green light:

```sh
gh pr review <pr> --approve
```

Never approve, request changes, or post comments on the PR without the user's go-ahead.

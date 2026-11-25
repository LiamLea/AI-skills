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

## 3. Dependencies

Before reviewing, inventory every external dependency the change introduces or relies on: images, actions, charts, modules, libraries, binaries. Output it as a table; the verdict can't be "Looks good" while any row is unchecked.

| Dependency | Packages (upstream path @ ref) | Status | Source |
|---|---|---|---|

- **Trace internal images and wrappers to what they build.** Search the org for the image name in build workflows (`gh search code "<image>" --owner <org>`), open the build context's Dockerfile, and note every `go install`, `pip install`, `FROM` and download.
- **Check the exact path, not just the repo.** Confirm the package or subdirectory still exists on the default branch and read its recent commits (`gh api "repos/<owner>/<repo>/commits?path=<dir>"`). An active repo can still have deleted, deprecated or moved the part you use.
- **Note floating refs** (`@latest`, `master`, `:latest`): the next rebuild picks up upstream changes, including removals.
- **Usage elsewhere is not evidence.** A pattern shared by sibling repos means they share the risk; check it anyway.

Feed anything deleted, archived, deprecated or near end-of-life into the review below.

## 4. Review

- **Don't be picky.** No style nits, naming bikesheds, or personal preferences.
- **Find explicit errors.** Bugs, broken logic, wrong config values, missing pieces that will fail, security holes, things that won't work as intended.
- **Leave optional things alone.** If something could be done differently but works fine, don't mention it.
- **Raise structural concerns.** If the result looks fine but the structure, approach, or placement feels off (wrong layer, duplicated logic, hard-to-maintain design, surprising side effects), definitely flag it.
- **Flag outdated or soon-to-break choices.** Code that works today but is already behind, or will stop working soon, is worth raising. For example:
  - **Old way, modern way exists**: a legacy API, pattern or config style where the tool now recommends a replacement (a deprecated Terraform argument or resource, a Kubernetes `apiVersion` with a newer GA version, old GitHub Actions syntax).
  - **Retired or retiring dependency**: a plugin, action, module, Helm chart, library or base image that is archived, unmaintained, deprecated, or has an announced end-of-life or removal date.
  - **Unsupported or near-EOL versions**: pinned runtimes, providers, engines or cluster versions that are out of support or close to it, or pins many major versions behind.
  - **Superseded within the repo**: the PR uses an approach the rest of the repo has already moved away from (an older module, a helper the codebase has replaced).

  Verify before flagging: check the changelog, release notes, deprecation notices or the upstream repo (archived? last release?), with a web search when unsure. Don't rely on memory alone, since these facts change. Cite the source and, when known, the date it stops working. Flag it only when it matters for this change: it will break, lose support, or create avoidable migration work. "A slightly newer version exists" is not a finding.

List each finding with `file:line`, what's wrong, and why it matters. If there are no findings, say so in one line.

## 5. Verdict

End with one of:

- **Looks good — ready to approve.** For very simple PRs or ones you're fully confident in. Ask the user for the green light.
- **Needs changes.** Summarize the blocking issues in one or two lines.
- **Needs discussion.** Structural concerns worth talking through before approving.

## 6. Approve

Only after the user explicitly gives the green light:

```sh
gh pr review <pr> --approve
```

Never approve, request changes, or post comments on the PR without the user's go-ahead.

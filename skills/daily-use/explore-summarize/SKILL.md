---
name: explore-summarize
description: "Given a URL (web page or Slack thread), fetch it, follow all links found in it, and produce a cited summary. Use when asked to: summarize a link, explore a URL, research a page and its links."
---

# Explore & Summarize

## If the URL is a Slack thread (`slack.com/archives/` or `app.slack.com`)

1. `slack_read_thread` — extract channel ID and ts from the URL (`p1234567890123456` → `1234567890.123456`)
2. `slack_search_public_and_private` — 2–3 queries on key terms/topics from the thread to find related context
3. For every web URL in the thread, apply the web steps below with that URL as the root

## If the URL is a web page

1. WebFetch the URL; extract content and all links
2. Skip: nav/footer, auth pages, social profiles, file downloads, `#anchor`-only links; cap at 20 links per page
3. WebFetch each included link; follow their links one level deeper with the same rules
4. If a fetch fails, skip and note it

## Output

Output the summary as plain markdown in the chat — do not use the Artifact tool.

Group findings under thematic headings with inline citations. End with a **References** list.
- Slack: cite as `[#channel @author](slack-url)`
- Web: cite as `[title](url)`
- Consolidate duplicate info across sources rather than repeating it
- State how many links were found vs. followed

---
name: draw-data-flow
description: "Draw a data flow diagram for the system, feature or request discussed in the current conversation: where data comes from, which components process it, where it is stored and where it goes. Gathers more context from the code repo, config or infra when needed. Simple flows are drawn directly in the chat; complex ones are published as an Artifact. Use when asked to: draw a data flow, show how data flows, diagram this flow, visualize the request path, where does this data go, map the pipeline."
---

# Draw Data Flow

Draw how data moves through the thing we've been talking about: sources, processing steps, stores and destinations.

## 1. Identify the scope

- Work out the subject from the conversation: a system, a feature, one request/endpoint, a job, a pipeline. If the user named one, use that.
- Fix the boundaries: where the flow starts (trigger or source) and where it ends (final store or consumer). Keep it to one flow unless the user asked for the whole system.
- If the scope is unclear (e.g. several flows were discussed), ask in one line before going further.

## 2. Gather context

Use what's already in the conversation first. Get more only where the flow has gaps:

- **Code repo:** find the entry point (HTTP route, gRPC handler, CLI command, consumer, cron job), then follow the calls: DB queries, cache reads/writes, outbound HTTP/gRPC calls, queue publishes/consumes, file/object storage, emitted events. Use grep for client constructors, SQL, topic/queue names, bucket names, URLs.
- **Config / infra:** read config files, env vars, k8s manifests, Terraform, etc. to resolve what a client actually points at (which DB, which bucket, which external service).
- **Docs / team knowledge:** README, design docs, Slack, Jira or Confluence when the code doesn't explain the why or the external parts.

For a large codebase, delegate the tracing to a subagent and keep only the resulting list of nodes and flows.

Don't invent components or flows. Mark anything inferred but not confirmed as `(unverified)` in the diagram or notes.

## 3. Model the flow

Before drawing, list the elements:

- **External entities:** users, clients, third-party services (sources/sinks outside the system).
- **Processes:** services, functions, jobs that transform or route data.
- **Data stores:** databases, caches, queues/topics, buckets, files.
- **Flows:** directed arrows, each labeled with *what data* moves (e.g. `order JSON`, `user_id + token`) and, if useful, *how* (HTTP, SQL, Kafka, S3 PUT). Number them when order matters.
- **Boundaries (optional):** trust boundaries, network/VPC, cluster or account boundaries, if they matter to the question.

Name nodes as they're named in the code/infra, so the user can find them.

## 4. Decide where to draw it

**Simple → draw in the chat.** Roughly: ≤ 5 nodes, one main path with few branches, a single level of detail.

- Use a plain-text box-and-arrow diagram in a code block (it renders everywhere, including the terminal). Keep it under ~100 characters wide.
- Use `[ ]` for processes, `( )` for external entities, `[( )]` for data stores, and put the data label on the arrows.

```
(User) --order JSON (HTTPS)--> [api-gateway] --CreateOrder (gRPC)--> [order-svc]
                                                                       |  1. INSERT order
                                                                       v
                                                                   [(orders DB)]
                                                                       |
                                              2. order.created (Kafka) v
                                                                  [notify-worker] --email--> (SendGrid)
```

**Complex → publish an Artifact.** Roughly: more than 8 nodes, several branches or parallel paths, multiple flows, boundaries, or the user wants something to share.

- Load the `artifact-design` and `artifact-diagramming` skills (or the Artifact tool's quickstart) before writing the page, and follow them.
- Draw the diagram as inline SVG: distinct shapes for entities, processes and stores; labeled, numbered arrows; boundaries as dashed groups; a small legend.
- Below the diagram, add a numbered step list matching the arrow numbers (what data, which protocol, and the source location, e.g. `internal/order/service.go:42`).
- If there are several flows (e.g. read path vs write path), show each as its own diagram or as a toggle, rather than one tangled picture.

## 5. Explain it

After the diagram (in chat, or as the reply that links the Artifact), add:

```
**Flow:** <one or two sentences: what triggers it and where the data ends up>

**Steps**
1. <source> → <target>: <what data, how> (<file:line or config>)
2. ...

**Notes:** <only if they matter: unverified parts, async/retry behavior, sensitive data, things left out of scope>
```

## Style

- Concise and straight to the point; the diagram carries the explanation.
- Every arrow says what data moves, not just that something is called.
- Prefer fewer, accurate nodes over a complete but unreadable picture; collapse internals that don't matter to the question.
- Write in the language the user has been using.
- Don't change any code or infra; this is for explanation only.

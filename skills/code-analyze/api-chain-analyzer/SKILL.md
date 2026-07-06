---
name: api-chain-analyzer
description: "Analyzes a codebase to discover all API endpoints, SQL queries, HTTP client calls, gRPC calls, message queue operations, and cache interactions — then traces the full downstream call chain for each endpoint. Use when asked to: map API dependencies, show what an endpoint calls, find all SQL used by a service, visualize request flows, or produce an API surface report. Outputs an interactive HTML report rendered as an Artifact."
---

# API Call Chain Analyzer

Produces an interactive HTML call-chain report for the target codebase.

## Workflow

### First run (no previous artifact URL)

1. **Determine target** — Always prefer the remote GitHub URL. If the user provides one, use it directly. If not, derive it from the repo name: `https://github.com/theplant/<repo>`. Only fall back to a local path if the remote is genuinely unavailable, and confirm with the user first.
2. **Discover APIs** — Find all route/handler registrations. Detect the language/framework from project files and search accordingly.
3. **Trace each API** — Read each handler file; follow function calls recursively (handler → service → repo layers) up to 4 levels deep or until leaf calls.
4. **Classify leaf calls** — Tag each discovered call as: `sql`, `http`, `grpc`, `queue`, `cache`, `external`, or `service`.
5. **Get current commit** — Run `git rev-parse HEAD` in the repo root. Store as `commit` in `DATA`.
6. **Build the call graph JSON** — Assemble `DATA` following the schema below (include `commit`).
7. **Render the report** — Read the template from `assets/report-template.html` (in the same skill directory), replace the literal token `CALL_GRAPH_DATA` with your JSON, then publish via the Artifact tool.

### Incremental re-run (user passes a previous artifact URL)

1. **Fetch previous result** — `WebFetch` the URL; extract `DATA` from the `const DATA = ` line.
2. **Check commit** — Run `git rev-parse HEAD`. If it matches `DATA.commit`, nothing changed — tell the user and stop.
3. **Scope the re-analysis** — Run `git diff --name-only <DATA.commit>..HEAD`. Re-trace only endpoints (across `DATA.services[].apis[]`) whose call chain touches a changed file; keep the rest unchanged.
4. **Rebuild and redeploy** — Merge results, update `commit` and `scanned_at`, then publish via Artifact with `url: <previous artifact URL>` to redeploy in place.

## Call Graph JSON Schema

```json
{
  "project": "repo-name",
  "scanned_at": "YYYY-MM-DD",
  "commit": "abc1234",
  "services": [
    {
      "name": "user",
      "apis": [
        {
          "method": "GET",
          "path": "/api/v1/users",
          "handler": "UserHandler.List",
          "file": "internal/user/handler.go:45",
          "calls": [
            {
              "type": "service",
              "label": "UserService.List(ctx, params)",
              "description": "Fetches the paginated list of active users, checking cache first",
              "file": "internal/user/service.go:23",
              "detail": null,
              "calls": [
                {
                  "type": "cache",
                  "label": "cache.Get(\"users:list\")",
                  "description": "Returns cached result if available, keyed by the users:list prefix",
                  "file": "internal/user/service.go:30",
                  "detail": null,
                  "calls": []
                },
                {
                  "type": "sql",
                  "label": "SELECT users",
                  "description": "Reads all active users ordered by newest first for the list response",
                  "file": "internal/user/repo.go:67",
                  "detail": "SELECT id, name, email FROM users WHERE active = true ORDER BY created_at DESC",
                  "calls": []
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

**Field rules:**
- `services[].name`: derived from the top-level directory containing the handler (e.g. `internal/user/…` → `user`)
- `type`: one of `service | sql | http | grpc | queue | cache | external | internal`
- `label`: short human-readable name (function name, SQL verb + table, URL)
- `description`: one sentence explaining what this call does in business terms — its purpose, not its mechanics (e.g. "Checks whether the user has an active session before proceeding", "Loads product catalogue for the given store"). Always populate this field; never leave it null.
- `detail`: the full SQL string, full URL, or proto method — omit (`null`) for service/internal calls
- `file`: `relative/path/to/file.go:lineNumber` — omit if unknown
- `calls`: nested array; empty array `[]` for leaf nodes

## Rendering

After building the JSON:

1. Read `assets/report-template.html` from the skill directory (same directory as this SKILL.md).
2. Replace the exact token `CALL_GRAPH_DATA` (appears on the line `const DATA = CALL_GRAPH_DATA;`) with the JSON object.
3. Publish via the Artifact tool with:
   - `file_path`: scratchpad path ending in `/api-chain-report.html`
   - `favicon`: `"🔗"`

## Tips

- If the codebase is large, focus grepping on `internal/`, `src/`, `app/`, `handler/`, `controller/`, `api/` directories first.
- For monorepos, each top-level service directory becomes one `services` entry — no need to prefix `path`.
- If a handler delegates to a method you can't locate (e.g. generated code), add a leaf node with `type: "external"` and label it clearly.
- SQL `detail` field: include the actual query string if visible in source; for ORM chains describe the operation (e.g. `"gorm: WHERE active=true ORDER BY created_at"`).
- Limit recursive tracing to 4 hops to avoid runaway analysis. Mark anything beyond depth 4 as `type: "internal"` with label `"(deeper calls not traced)"`.

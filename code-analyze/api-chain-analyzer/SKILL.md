---
name: api-chain-analyzer
description: Analyzes a codebase to discover all API endpoints, SQL queries, HTTP client calls, gRPC calls, message queue operations, and cache interactions — then traces the full downstream call chain for each endpoint. Use when asked to: map API dependencies, show what an endpoint calls, find all SQL used by a service, visualize request flows, or produce an API surface report. Outputs an interactive HTML report rendered as an Artifact.
---

# API Call Chain Analyzer

Produces an interactive HTML call-chain report for the target codebase.

## Workflow

1. **Determine target** — Use the directory the user specifies, or the current working directory.
2. **Detect language/framework** — Check for `go.mod`, `package.json`, `pom.xml`, `requirements.txt`, etc. Adjust grep patterns accordingly.
3. **Discover APIs** — Find all route/handler registrations (see patterns below).
4. **Trace each API** — Read each handler file; follow function calls recursively (handler → service → repo layers) up to 4 levels deep or until leaf calls.
5. **Classify leaf calls** — Tag each discovered call as: `sql`, `http`, `grpc`, `queue`, `cache`, `external`, or `service`.
6. **Build the call graph JSON** — Assemble `DATA` following the schema below.
7. **Render the report** — Read the template from `assets/report-template.html` (in the same skill directory), replace the literal token `CALL_GRAPH_DATA` with your JSON, then publish via the Artifact tool.

## Discovery Patterns

### Go
```
# HTTP routes
grep -rn --include="*.go" -E 'r\.(GET|POST|PUT|PATCH|DELETE|Handle|HandleFunc)\(|router\.(GET|POST|PUT|PATCH|DELETE)\(|mux\.Handle|e\.(GET|POST|PUT|PATCH|DELETE)\(|http\.Handle'

# SQL (raw + sqlx + gorm)
grep -rn --include="*.go" -E 'db\.(Query|Exec|QueryRow|QueryContext|ExecContext|QueryRowContext|Get|Select|NamedQuery)\(|\.Find\(|\.Create\(|\.Save\(|\.Delete\(|\.Where\(|\.First\('

# Outbound HTTP
grep -rn --include="*.go" -E 'http\.(Get|Post|NewRequest)\(|client\.(Get|Post|Do)\('

# gRPC clients
grep -rn --include="*.go" -E 'pb\.New\w+Client\(|grpc\.Dial\('

# Message queues (SQS/SNS/Kafka/NATS)
grep -rn --include="*.go" -E '\.SendMessage\(|\.Publish\(|\.Produce\(|\.Subscribe\(|\.Consume\('

# Cache (Redis)
grep -rn --include="*.go" -E 'rdb\.(Get|Set|HGet|HSet|Del|Expire)|cache\.(Get|Set|Delete)\('
```

### TypeScript / JavaScript
```
# Express / Next.js / tRPC routes
grep -rn --include="*.ts" --include="*.tsx" -E 'app\.(get|post|put|patch|delete)\(|router\.(get|post|put|patch|delete)\(|export default.*handler|procedure\.'

# SQL (prisma / knex / pg)
grep -rn --include="*.ts" -E 'prisma\.\w+\.(find|create|update|delete|upsert)|knex\(|db\.query\('

# Outbound HTTP
grep -rn --include="*.ts" -E 'fetch\(|axios\.(get|post|put|patch|delete)\('
```

## Call Graph JSON Schema

```json
{
  "project": "repo-name",
  "scanned_at": "YYYY-MM-DD",
  "apis": [
    {
      "method": "GET",
      "path": "/api/v1/users",
      "handler": "UserHandler.List",
      "file": "internal/handler/user.go:45",
      "calls": [
        {
          "type": "service",
          "label": "UserService.List(ctx, params)",
          "file": "internal/service/user.go:23",
          "detail": null,
          "calls": [
            {
              "type": "cache",
              "label": "cache.Get(\"users:list\")",
              "file": "internal/service/user.go:30",
              "detail": null,
              "calls": []
            },
            {
              "type": "sql",
              "label": "SELECT users",
              "file": "internal/repo/user.go:67",
              "detail": "SELECT id, name, email FROM users WHERE active = true ORDER BY created_at DESC",
              "calls": []
            }
          ]
        }
      ]
    }
  ]
}
```

**Field rules:**
- `type`: one of `service | sql | http | grpc | queue | cache | external | internal`
- `label`: short human-readable name (function name, SQL verb + table, URL)
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
- For monorepos, produce one `DATA.apis` array covering all services; prefix `path` with the service name (e.g. `/user-service/api/v1/…`).
- If a handler delegates to a method you can't locate (e.g. generated code), add a leaf node with `type: "external"` and label it clearly.
- SQL `detail` field: include the actual query string if visible in source; for ORM chains describe the operation (e.g. `"gorm: WHERE active=true ORDER BY created_at"`).
- Limit recursive tracing to 4 hops to avoid runaway analysis. Mark anything beyond depth 4 as `type: "internal"` with label `"(deeper calls not traced)"`.

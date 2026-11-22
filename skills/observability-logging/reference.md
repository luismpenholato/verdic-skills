# Reference – Observability questions

Before adding telemetry, write the questions operators need answered:

1. Is the service up?
2. Which endpoint is failing and for whom?
3. How slow is it (p50/p95)?
4. Did a dependency fail (DB, HTTP, broker)?
5. Can we find one user/request across services?

## Suggested minimum fields

| Field | Purpose |
|---|---|
| `timestamp` | Ordering |
| `level` | Filtering |
| `message` | Human summary |
| `requestId` / `traceId` | Correlation |
| `route` / `operation` | Where |
| `statusCode` / `outcome` | Result |
| `durationMs` | Performance |

## Redaction

Always redact or hash: passwords, refresh tokens, API keys, connection strings, session cookies, full payment payloads.

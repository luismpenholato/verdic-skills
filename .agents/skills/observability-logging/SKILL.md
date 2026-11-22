---
name: observability-logging
description: Add structured logging, correlation, metrics and safe diagnostics for .NET (Serilog) and Node/Next apps. Use when debugging production issues, adding App Insights/OpenTelemetry, correlating requests, or when the user asks for logging, metrics, tracing or observability.
---

# Observability and logging

Make failures diagnosable without leaking secrets. Prefer structured logs, correlation IDs, and a small set of useful metrics over noisy `Console.WriteLine`.

## When to use

- Production incidents, intermittent bugs, slow endpoints.
- Adding Serilog, OpenTelemetry, Application Insights, Seq, Prometheus.
- New public APIs that need audit/correlation.
- User asks for logging, metrics, tracing or observability.

## When not to use

- Tiny local scripts with no operators.
- Replacing a proper debugger session for a single local failure.
- Logging for the sake of logging (noise without questions to answer).

## Principles

- Logs answer **who/what/when/where/outcome** for a request or job.
- Structure fields (`RequestId`, `UserId`, `Feature`) — avoid unstructured blobs of secrets.
- Never log passwords, tokens, connection strings, full cards/PII unless required and redacted.
- Prefer sampling and levels (`Debug` local, `Information`+ in prod).

## Checklist

- [ ] Correlation / request id flows through API → workers → logs.
- [ ] HTTP middleware or framework logging captures method, path, status, duration.
- [ ] Exceptions logged once at the boundary with stack in protected sinks only.
- [ ] Business-significant events logged (create order, login failure) without sensitive payloads.
- [ ] Metrics for latency/error rate on critical endpoints when ops need SLOs.
- [ ] Health checks for dependencies (DB, broker) without exposing internals publicly.

## .NET notes

- Serilog with enrichers for machine/environment/request id.
- Use `ILogger<T>` in handlers; avoid static loggers in Domain entities.
- Map unhandled exceptions in middleware; do not duplicate log + rethrow carelessly.

## Frontend / Next.js notes

- Client: log actionable UI failures to a controlled sink; do not spam console in production builds.
- Server components/routes: structured server logs; keep `NEXT_PUBLIC_*` free of secrets.
- Prefer correlating browser errors with API `RequestId` returned in ProblemDetails/extensions when available.

## Anti-patterns

- Logging entire request bodies by default.
- `Information` logs inside tight loops.
- Different formats per service with no correlation field.
- Exposing detailed health/diagnostics on anonymous public URLs.

## Related

- Security of logs/PII: `application-security-review`
- More patterns: [reference.md](reference.md)

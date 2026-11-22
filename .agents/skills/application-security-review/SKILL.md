---
name: application-security-review
description: Review and harden application security for .NET, Angular, React/Next.js and APIs. Covers authZ/authN, secrets, injection, XSS, IDOR, SSRF, CORS, supply chain and logging. Use when touching login, tokens, permissions, public endpoints, file/URL handling, or when the user asks for a security review.
---

# Application security review

Practical security review for product code. Passing functional tests is **not** proof that risk is absent. Prefer defense in depth: design → code → CI → deploy → operate.

## When to use

- Auth, sessions, JWT, cookies, roles/permissions, multi-tenant data.
- New or changed public HTTP endpoints, webhooks, file upload/download, outbound HTTP.
- Secrets, connection strings, CI/CD, Docker, dependency upgrades.
- User asks for a security review, threat model, or hardening before release.

## When not to use

- Pure docs/typo changes with no trust-boundary impact.
- Full red-team / penetration test engagements (use specialized process).
- Inventing scanner results that were not executed.

## Principles

- Fail closed; default deny; least privilege.
- Validate on the **server**; never trust the client.
- Secrets in git history are compromised — **rotate**, do not only delete.
- CORS is not access control.
- No single control is enough (SAST ≠ SCA ≠ human review).

## Review checklist

Skip categories with no exposure and say what was out of scope.

### 1. Design and trust boundaries

- [ ] Assets, actors, entry points and trust boundaries identified.
- [ ] AuthN/AuthZ explicit for write/admin paths (not anonymous by accident).
- [ ] Sensitive data classified (credentials, PII, tokens, business data).
- [ ] Abuse cases considered: IDOR, privilege escalation, mass assignment, replay.

### 2. Input, output, injection

- [ ] Untrusted input validated (type, length, allowlist when possible).
- [ ] SQL/NoSQL/OS commands use parameterized APIs — no string concatenation.
- [ ] HTML/JS escaped; no untrusted `dangerouslySetInnerHTML` / `innerHTML` / raw `[innerHTML]`.
- [ ] Paths confined to allowlisted directories (no traversal).
- [ ] Server-initiated fetches allowlisted (SSRF); block private/link-local ranges.
- [ ] Safe deserialization; reject unexpected shape/size.
- [ ] No unjustified `eval`, dynamic code, or shell injection sinks.

### 3. Authentication and authorization

- [ ] No default/hardcoded credentials.
- [ ] Secrets from env/secret store; `.env` gitignored.
- [ ] Cookie flags (`HttpOnly`, `Secure`, `SameSite`) and CSRF considered when cookies are used.
- [ ] Authorization checked **per resource** (not only “user is logged in”).
- [ ] Frontend guards are UX only — API enforces the same rules.

### 4. Secrets and repository hygiene

- [ ] No secrets in source, committed config, Docker layers, or CI logs.
- [ ] If leaked: rotate/revoke first, then remove from tree/history as needed.
- [ ] Prefer secret scanning in pre-commit/CI; protect `main`.

### 5. Supply chain

- [ ] Lockfiles present and used (`package-lock.json`, NuGet restore consistency).
- [ ] Known vulnerable packages checked (`npm audit`, `dotnet list package --vulnerable`).
- [ ] New dependencies justified; watch typosquatting and abandoned packages.

### 6. Runtime, containers, config

- [ ] Non-root container user, minimal base image, no secrets in `ENV`/`COPY`.
- [ ] Debug/Swagger/dev servers not exposed publicly in production.
- [ ] Ports and privileges minimized; databases not published to untrusted networks.

### 7. Errors, logs, abuse controls

- [ ] Client errors generic; detailed traces only in protected logs.
- [ ] Logs omit secrets, tokens, passwords and unnecessary PII.
- [ ] Rate limits / size limits / timeouts on expensive or risky endpoints.
- [ ] Audit trail for sensitive mutations when repudiation matters.

## Stack notes

### .NET API

- Prefer FluentValidation / model binding + explicit checks in handlers.
- Never concatenate SQL; use EF parameterized queries or Dapper parameters.
- Return ProblemDetails without stack traces in production.
- Authorize at endpoint/handler level; do not rely on “hidden” routes.

### Angular / React / Next.js

- Treat SPA auth as UX; APIs must re-check.
- Avoid rendering untrusted HTML; sanitize if unavoidable.
- Centralize HTTP clients; do not stash tokens in `localStorage` when httpOnly cookies are viable.
- Next.js: keep secrets off `NEXT_PUBLIC_*`; server-only env for private keys.

## High-risk changes

For auth, multi-tenant data, payments, secrets, remote file/URL handling or public exposure, produce a short threat note covering:

1. Assets and actors  
2. Entry points  
3. Top threats (STRIDE-oriented)  
4. Mitigations  
5. Residual risk and verification status  

## Output format

Report findings by severity (`Critical` / `High` / `Medium` / `Low` / `Info`) with:

- location (file/route/config),
- impact and likely abuse path,
- concrete mitigation,
- residual risk if unfixed,
- verification status (executed / not executed).

If no findings, state what was reviewed and what was **not** verified.

## Anti-patterns

- “Security later” on public write endpoints.
- Trusting only frontend route guards.
- Logging Authorization headers or full JWTs.
- Claiming “secure” because CI is green without listing which checks ran.

## Related

- Method-level practice: [Verdicspec security skill](https://github.com/luismpenholato/verdicspec)
- Quick checklist: [reference.md](reference.md)

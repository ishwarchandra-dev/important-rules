# Production-Readiness Checklist — SDLC Master Reference

> **Purpose:** A universal, testable checklist to determine if ANY web application is ready for production. Covers security, reliability, compliance, financial safety, and operational excellence.
>
> **Sources:** OWASP Top 10, SANS/CWE Top 25, NIST SSDF, Google SRE PRR, AWS ORR, Azure Well-Architected, PCI DSS, GDPR/CCPA, HIPAA, Netflix Chaos Engineering, SOC 2, ISO 27001.
>
> **How to use:** Run through every item. Tag each as ✅ Met / ⚠️ Partial / ❌ Gap. Any ❌ in categories 1–6 blocks production. Categories 7–15 are strongly recommended.

---

## Table of Contents

1. [Authentication & Identity](#1-authentication--identity)
2. [Authorization & Access Control](#2-authorization--access-control)
3. [Input Validation & Injection Prevention](#3-input-validation--injection-prevention)
4. [Secrets Management](#4-secrets-management)
5. [Cryptography & Data Protection](#5-cryptography--data-protection)
6. [Transport & Network Security](#6-transport--network-security)
7. [Session Management](#7-session-management)
8. [Database Reliability](#8-database-reliability)
9. [API Design & Reliability](#9-api-design--reliability)
10. [Error Handling & Resilience](#10-error-handling--resilience)
11. [Observability](#11-observability)
12. [CI/CD & Deployment](#12-cicd--deployment)
13. [Testing & Code Quality](#13-testing--code-quality)
14. [Financial Safety & Billing](#14-financial-safety--billing)
15. [Data Privacy & Compliance](#15-data-privacy--compliance)
16. [Disaster Recovery & Backups](#16-disaster-recovery--backups)
17. [Incident Response](#17-incident-response)
18. [Dependency & Supply Chain Security](#18-dependency--supply-chain-security)
19. [Infrastructure & Cloud Hardening](#19-infrastructure--cloud-hardening)
20. [Documentation & Runbooks](#20-documentation--runbooks)
21. [Common Bug Prevention](#21-common-bug-prevention)

---

## 1. Authentication & Identity

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 1 | Enforce MFA on all admin, CI/CD, and cloud-console accounts | Stolen credentials are the #1 initial-access vector (Verizon DBIR) | Attempt console login with password only; it must be rejected |
| 2 | Enforce password policy: ≥12 chars, breach-list check, no reuse of last 10 | Weak/reused passwords dominate account takeover | Submit a known-breached password at signup; it must be rejected |
| 3 | Lock or rate-limit failed logins (e.g., 10 attempts / 10 min → lockout) | Prevents credential stuffing and brute force | Send 11 bad logins; observe lockout and alert |
| 4 | Rotate machine-to-machine credentials and API keys every 90 days | Limits blast radius of leaked service credentials | Inspect IAM key `CreateDate`; older than 90 days fails audit |
| 5 | Use OAuth 2.1 / OIDC with PKCE for user auth; never custom token issuance | Custom auth protocols are historically bug-ridden | Confirm `code_challenge`/`code_verifier` in auth flow |
| 6 | Hash passwords with Argon2id (or bcrypt cost ≥12); never store plaintext | Defeats offline cracking if the DB leaks | DB column shows hash prefix; plaintext search returns nothing |
| 7 | Validate password-reset tokens are single-use, expire ≤15 min, invalidate all sessions on reset | Prevents token replay and account-takeover via hijacked email | Use a reset token twice; second use is rejected |
| 8 | Validate JWT `alg` field; reject `alg:none` and `alg:RS256→HS256` confusion | JWT algorithm confusion bypasses signature verification | Submit a token with `alg:none`; confirm rejection |

## 2. Authorization & Access Control

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 9 | Enforce deny-by-default access control checked on EVERY request, not just at login | Missing authorization (CWE-862) is #4 in CWE Top 25 | Call an admin endpoint as a regular user; confirm 403 |
| 10 | Verify ownership before every write: user can only modify resources they own | Cross-tenant data modification is a critical privilege escalation | Attempt to update another user's resource by ID; confirm 404/403 |
| 11 | Apply RBAC with least-privilege; super_admin role escalation requires super_admin | Prevents admins from promoting themselves to super_admin | As admin, attempt to set role=super_admin; confirm 403 |
| 12 | Prevent self-role-change and self-deactivation | Lockout footgun — user locks themselves out | Attempt to change own role or deactivate own account; confirm 403 |
| 13 | Scope all list/read endpoints by owner_id for user-role callers | Without scoping, any user sees all tenants' data | List resources as user; confirm only own resources returned |
| 14 | Enforce CSRF protection on all state-changing endpoints (if using cookies) | CSRF allows attackers to perform actions as the victim | Submit a cross-origin POST without CSRF token; confirm rejection |
| 15 | Validate that indirect object references (IDs) are checked against the caller's scope | IDOR (CWE-639) lets users access other users' data by guessing IDs | Iterate IDs as user A; confirm user B's resources return 404 |

## 3. Input Validation & Injection Prevention

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 16 | Use parameterized queries / prepared statements exclusively; no string-concatenated SQL | SQL injection (CWE-89) is #3 in CWE Top 25 | Grep for raw SQL string concatenation; confirm zero |
| 17 | Sanitize/escape all user input before rendering in HTML (XSS prevention) | XSS (CWE-79) is #1 in CWE Top 25 | Submit `<script>alert(1)</script>` in every input field; confirm it's escaped |
| 18 | Validate and sanitize all inputs at the boundary (schema + length + type + range) | Bad input causes 500s, injection, and data corruption | Send malformed payloads; confirm 400, never 500 |
| 19 | Enforce maximum payload size (e.g., 1MB) at the gateway | Prevents memory exhaustion from huge bodies | POST a 10MB body; confirm 413 rejection |
| 20 | Validate file uploads: check magic bytes, enforce size limits, store outside webroot | Malicious uploads can lead to RCE | Upload a renamed executable; confirm rejection |
| 21 | Prevent SSRF: validate and restrict outbound URLs (reject RFC1918, loopback, link-local) | SSRF lets attackers read cloud metadata and internal services | Set a provider base_url to `http://169.254.169.254/`; confirm rejection |
| 22 | Prevent path traversal: use `filepath.Rel()` and reject `..` segments | Path traversal lets attackers read/write arbitrary files | Submit `../../etc/passwd` as a filename; confirm rejection |
| 23 | Cap regex complexity to prevent ReDoS (catastrophic backtracking) | ReDoS can freeze the server with a single malicious input | Submit a pathological regex input; confirm it times out, not hangs |

## 4. Secrets Management

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 24 | Store all secrets (API keys, DB passwords, tokens) in a vault/KMS, never in code or env files committed to git | Committed secrets are the most common breach vector | Run `gitleaks`/`trufflehog` on the repo; confirm zero findings |
| 25 | Use a pre-commit hook that scans for secrets before allowing commits | Prevents secrets from entering git history | Attempt to commit a fake `sk-` key; confirm the hook blocks it |
| 26 | Encrypt credentials at rest with AES-256-GCM + per-row nonces + keyring versioning | DB dump leaks plaintext keys without encryption | `pg_dump` the DB; grep for known API key strings; confirm zero matches |
| 27 | Rotate encryption master keys with zero downtime (versioned keyring + multi-version decrypt) | Key rotation is required for SOC 2 / PCI compliance | Rotate the master key; confirm existing credentials still decrypt |
| 28 | Never log secrets, tokens, or PII — redact at the logger level | Logged secrets are a major breach vector | Log a test `cgk_` / `sk-` / `Bearer ` value; confirm `[REDACTED]` in output |
| 29 | Separate dev/staging/prod secrets; no shared credentials across environments | A leaked dev key shouldn't grant prod access | Compare env files across environments; confirm different secrets |

## 5. Cryptography & Data Protection

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 30 | Enforce TLS 1.2+ everywhere with HSTS (max-age ≥ 1 year, includeSubDomains, preload) | Plaintext transport exposes credentials | `curl -I http://...` returns 301 to HTTPS; HSTS header present |
| 31 | Use AES-256-GCM (not CBC/ECB) for encryption at rest | GCM provides authenticated encryption; CBC is vulnerable to padding oracle | Review crypto code; confirm `crypto/cipher.NewGCM` or equivalent |
| 32 | Generate random nonces/IVs with `crypto/rand` (never `math/rand`) for crypto operations | Predictable nonces break encryption entirely | Grep for `math/rand` near crypto code; confirm zero |
| 33 | Use constant-time comparison for tokens/secrets (`subtle.ConstantTimeCompare` or `hmac.Equal`) | Timing attacks leak token values byte-by-byte | Grep for `==` comparisons on secrets; confirm only constant-time is used |
| 34 | Enforce minimum key lengths: RSA ≥2048, ECDSA ≥256, AES ≥256, HMAC-SHA ≥256 | Short keys are brute-forceable | Check key generation code; confirm minimums |
| 35 | Hash bcrypt dummy passwords at the configured cost (not a fixed lower cost) for "user not found" | Timing-based user enumeration via different bcrypt costs | Time a login with nonexistent vs. wrong-password user; confirm similar latency |

## 6. Transport & Network Security

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 36 | Configure CORS with explicit origin allowlist (never `*` with credentials) | Wildcard CORS with credentials allows any site to make authenticated requests | Inspect CORS config; confirm specific origins, not `*` |
| 37 | Set security headers: X-Content-Type-Options, X-Frame-Options: DENY, Referrer-Policy, CSP, Permissions-Policy | Missing headers enable clickjacking, MIME sniffing, and XSS | `curl -I` the app; confirm all headers present |
| 38 | Run containers as non-root with read-only filesystem where possible | Root containers can escape to the host | Inspect Dockerfile; confirm `USER nonroot` and `readOnlyRootFilesystem` |
| 39 | Expose only necessary ports (close all non-essential ports) | Open ports expand the attack surface | `nmap` the server; confirm only 80/443 (or app-specific) are open |
| 40 | Enable rate limiting per IP and per API key with 429 + Retry-After header | Protects the system from abusive or buggy clients | Burst a client past its limit; confirm 429 with Retry-After |
| 41 | Set request timeouts on every API call (no infinite waits) | Hanging connections exhaust the connection pool | Inspect config; every outbound HTTP client has a timeout |

## 7. Session Management

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 42 | Set cookie flags: HttpOnly, Secure, SameSite=Lax (or Strict) | Without these, cookies can be stolen via XSS or CSRF | Inspect Set-Cookie header; confirm all three flags |
| 43 | Enforce session limits (e.g., max 5 concurrent sessions per user with FIFO eviction) | Prevents session sprawl / token hoarding | Log in 6 times; confirm oldest session is revoked |
| 44 | Issue short-lived access tokens (≤15 min) + rotating refresh tokens (single-use) | Long-lived tokens are a theft risk; rotation detects reuse | Replay a used refresh token; confirm rejection |

## 8. Database Reliability

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 45 | Use connection pooling with a bounded pool size | Prevents DB connection exhaustion under load | Load-test shows connections plateau, not grow unbounded |
| 46 | Index every foreign key column and every column in WHERE/JOIN/ORDER BY | Missing indexes cause full table scans and lock contention | Run `EXPLAIN` on top queries; no `Seq Scan` on large tables |
| 47 | Wrap multi-step writes in explicit transactions | Partial writes leave data inconsistent | Code review confirms transaction boundaries around multi-statement writes |
| 48 | Run migrations online and backward-compatible (expand-then-contract) | Locking migrations block the app during deploy | Confirm migrations avoid long AccessExclusive locks |
| 49 | Back up the database and verify restores periodically | Untested backups are not backups | Perform a restore drill into staging quarterly; assert row counts |
| 50 | Enable point-in-time recovery (PITR) with RPO ≤15 min | Limits data loss after accidental deletes/corruption | Restore to a timestamp 10 min ago; confirm success |
| 51 | Detect and fix N+1 queries (eager-load associations, use JOINs) | N+1 is the most common DB performance anti-pattern | Enable ORM N+1 detection; confirm zero in top endpoints |
| 52 | Set statement timeouts on the DB and app side (e.g., 5s) | A single slow query can hold locks and cascade | Run `SHOW statement_timeout`; confirm non-zero, sane value |
| 53 | Enforce foreign-key constraints in production | Constraints are the last line of data-integrity defense | Inspect schema; FKs present and enforced |
| 54 | Add UNIQUE constraints on natural business keys | Prevents duplicate records from race conditions | Review schema for expected unique constraints |
| 55 | Use optimistic locking (version column) for concurrent updates | Prevents lost updates without heavy locking | Confirm version/etag column on mutable entities |
| 56 | Monitor slow-query log and review weekly | Slow queries degrade the whole system over time | Confirm slow-query log enabled and triaged |
| 57 | Verify column types match across JOINs (e.g., TEXT vs UUID) | Type mismatches cause runtime SQL errors | Review all JOINs for type compatibility |

## 9. API Design & Reliability

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 58 | Enforce pagination with a max page size on all list endpoints | Unbounded queries exhaust memory and DB | Call a list endpoint with no params; confirm capped response |
| 59 | Make POST/PUT/DELETE idempotent via an idempotency-key header | Safe client retries without duplicate side effects | Replay the same request with identical key; assert single effect |
| 60 | Return structured, consistent error envelopes (code, message, request-id) | Enables clients to handle errors programmatically | Inspect error responses across endpoints; schema is uniform |
| 61 | Version APIs explicitly with a deprecation policy | Allows safe evolution without breaking clients | Confirm v1 still serves while v2 ships, with sunset date |
| 62 | Add a request-id/correlation ID propagated end-to-end | Enables tracing a request across services | Assert the same ID appears in logs of all downstream services |
| 63 | Validate response shapes match between frontend types and backend structs | Mismatched shapes cause runtime undefined errors | Compare Go struct JSON tags against TypeScript types |

## 10. Error Handling & Resilience

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 64 | Implement retries with exponential backoff + jitter for transient failures | Plain retries amplify load; jitter spreads it out | Code review confirms backoff + jitter, not fixed delays |
| 65 | Cap total retry time/attempt count per operation | Infinite retries can hang a request and exhaust resources | Confirm a `maxRetries` / `maxDuration` bound in config |
| 66 | Use circuit breakers on outbound calls to failing dependencies | Prevents cascading failure when a dependency is down | Trip a downstream; confirm circuit opens and fails fast |
| 67 | Implement graceful degradation (fallback responses) when non-critical deps fail | Keeps core UX working during partial outages | Kill a secondary dependency; confirm primary feature returns fallback |
| 68 | Never swallow exceptions silently — at minimum log with context | Silent failures hide bugs until they become outages | Grep for empty `catch` blocks; confirm none in production code |
| 69 | Never leak internal error details (SQL errors, stack traces, file paths) to clients | Error leakage reveals internal architecture to attackers | Send malformed input; confirm 400/500 with generic message, no internals |
| 70 | Distinguish retryable from non-retryable errors (5xx vs 4xx) | Retrying a 400 wastes resources and never succeeds | Verify retry logic checks error type before retrying |
| 71 | Use bulkheads (separate pools per dependency) to isolate failures | One slow dependency shouldn't exhaust all worker threads | Confirm thread/connection pools are partitioned per downstream |

## 11. Observability

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 72 | Emit structured (JSON or tabular) logs with request-id, user-id, timestamp | Unstructured logs are unsearchable at scale | Sample logs; confirm parseable format with required fields |
| 73 | Collect the four golden signals (latency, traffic, errors, saturation) per service | These are the minimum signals to detect any problem | Open the service dashboard; all four are present and alerting |
| 74 | Implement distributed tracing across all services (OpenTelemetry) | Traces pinpoint where latency/failures originate | Inspect a trace; it spans all services in the request path |
| 75 | Define alerting on SLO burn rate, not raw thresholds alone | Burn-rate alerts catch sustained degradation early | Confirm alerts use multi-window burn-rate |
| 76 | Set up synthetic/uptime monitoring for critical user journeys | Catches outages before users report them | Confirm a synthetic probe hits the login flow every minute |
| 77 | Centralize logs in a single searchable backend | Distributed logs make incident triage impossible | Confirm all services ship to one log store |
| 78 | Run a real `/readyz` health check (DB ping, Redis ping) — not hardcoded "ok" | Fake health checks hide broken dependencies | Kill the DB; confirm `/readyz` returns 503 |

## 12. CI/CD & Deployment

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 79 | CI pipeline blocks merge on failing tests, lint, and SAST | Prevents known-defective code from reaching main | Attempt to merge a red build; confirm rejection |
| 80 | Use progressive deployment (canary or blue-green) for production releases | Limits blast radius to a small cohort before full rollout | Confirm the deploy pipeline includes a canary stage with metrics gating |
| 81 | Implement automated, tested one-command rollback | Rollback speed directly limits MTTR | Trigger rollback in staging; assert service returns < 5 min |
| 82 | Use feature flags to decouple deploy from release | Enables dark launches and instant kill-switches | Confirm new risky features ship behind a flag defaulting to off |
| 83 | Every build is immutable, versioned, and reproducible from a single artifact ID | Ensures the exact artifact tested is the one deployed | Confirm prod runs the same image SHA staged in CI |
| 84 | Database migrations are backward-compatible and reversible | Bad migrations are the #1 cause of deploy outages | Review the last 3 migrations; each has a forward + backward step |

## 13. Testing & Code Quality

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 85 | Unit test coverage ≥80% on core packages (business logic, auth, crypto) | Low coverage means untested edge cases in production | Run `go test -cover` / `jest --coverage`; confirm ≥80% |
| 86 | Integration tests against real containers (testcontainers), not mocks only | Mock-only tests miss real-world interaction bugs | Confirm CI runs service-level tests against real DB/Redis containers |
| 87 | Contract tests verify frontend types match backend response shapes | Mismatched shapes cause runtime crashes | Run contract test suite; confirm all endpoints match |
| 88 | Load test to ≥2x expected peak traffic before launch | Proves the system survives growth, not just nominal load | Existence of a load-test report hitting the target without SLO breach |
| 89 | Run `go test -race` (or equivalent) to detect race conditions | Race conditions corrupt data and are hard to reproduce | Confirm CI runs with `-race` flag; zero findings |
| 90 | Run security scanning (gosec, govulncheck, npm audit, Trivy) in CI | Automated scanning catches known CVEs | Confirm CI security job runs on every PR + weekly |
| 91 | Code review required for every PR before merge (no direct pushes to main) | Human review catches bugs that automated tools miss | Confirm branch protection rules on main/dev |

## 14. Financial Safety & Billing

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 92 | Require Idempotency-Key header on every payment/charge mutation | Prevents double-charging when users double-tap | Send the same key twice; confirm a single charge |
| 93 | Store idempotency key + request body BEFORE calling the PSP; replay stored result on retries | Re-invoking the PSP on retry causes duplicate charges | Code review the charge path; unit-test the "key already processed" branch |
| 94 | Wrap charge + order creation in a single DB transaction with row-level locking | Race conditions cause oversell and double-billing | Run 100 concurrent requests on same cart; assert exactly one charge |
| 95 | Use a state machine for payment/order status with explicit allowed transitions | Invalid transitions cause silent double-fulfillment | Assert the state machine throws on illegal transition |
| 96 | Reconcile PSP webhook events against your ledger daily; alert on unmatched charges | Drift between DB and PSP hides phantom revenue | Force a mismatch; confirm alert fires |
| 97 | Never return fake/mock billing data — show real usage or "not available" | Fake billing data misleads users and can cause legal issues | Inspect billing endpoint; confirm only real data or honest empty states |

## 15. Data Privacy & Compliance

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 98 | Maintain a data inventory (ROPA) listing every data store, purpose, legal basis, retention | GDPR Art. 30 requires it; you can't honor deletion requests without it | Check the ROPA doc covers each DB table/object store |
| 99 | Provide working data-subject rights endpoints: access, erasure, portability within 30 days | Failure to fulfill DSARs triggers fines up to 4% of global revenue | Submit a test DSAR; confirm export + deletion within SLA |
| 100 | Apply data minimization: collect only fields needed for the stated purpose | Over-collection violates purpose-limitation principle | Diff every form/API payload against documented purpose |
| 101 | Encrypt personal data at rest (AES-256) and in transit (TLS 1.2+) | Encryption is the GDPR "appropriate technical measure" | Confirm field-level encryption on PII columns |
| 102 | Have a documented 72-hour breach-notification process | GDPR Art. 33 mandates notification within 72 hours | Review the runbook + run a tabletop drill |

## 16. Disaster Recovery & Backups

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 103 | Document RPO (data loss tolerance) and RTO (recovery time) targets per service | Without targets you can't measure if DR is adequate | Review the DR doc; each service has RPO/RTO |
| 104 | Automate backups with a schedule matching the RPO | Manual backups are forgotten when it matters most | Confirm automated backup schedule; review last 7 days of runs |
| 105 | Store backups in a different region/zone than production | A single AZ failure shouldn't destroy backups | Confirm backup storage location differs from prod |
| 106 | Test backup restoration quarterly (not just backup creation) | Untested backups are not backups | Perform a restore drill; assert row counts match |
| 107 | Maintain a runbook for disaster recovery with step-by-step instructions | At 3am, no one remembers the recovery procedure | Spot-check the DR runbook for actionable, tested steps |

## 17. Incident Response

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 108 | Define an escalation policy with primary + secondary on-call, never a single person | Single-person on-call creates bus-factor risk | Review PagerDuty/Opsgenie schedule; confirm 24/7 dual coverage |
| 109 | Conduct blameless postmortems for every Sev-1/Sev-2 within 5 business days | Drives systemic fixes instead of blame | Check last 3 incidents each have a published postmortem |
| 110 | Track postmortem action items to completion with owners and due dates | Lessons learned but not fixed will recur | Query the action-item tracker; ≥80% closure rate |
| 111 | Define severity levels (Sev1–Sev4) with objective criteria | Removes ambiguity in paging decisions | Review the severity matrix; each level has measurable thresholds |
| 112 | Run periodic incident game-days / tabletop exercises | Muscle memory for response improves MTTR | Calendar shows ≥1 game-day per quarter with notes |

## 18. Dependency & Supply Chain Security

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 113 | Run SCA (Software Composition Analysis) on every PR + weekly | Known CVEs in dependencies are the fastest-growing attack vector | Confirm `govulncheck` / `npm audit` runs in CI |
| 114 | Remove dead/unused dependencies from go.mod / package.json | Dead deps inflate supply-chain surface and slow installs | Confirm `go mod tidy` runs in CI; no unused packages |
| 115 | Pin Docker base images by digest (not mutable tags like `:latest`) | Mutable tags can be replaced with malicious images | Inspect Dockerfiles; confirm `@sha256:...` pinning |
| 116 | Generate an SBOM (Software Bill of Materials) for every release | SBOMs are required for federal software (EO 14028) | Confirm build pipeline emits an SBOM artifact |
| 117 | Sign container images (cosign) and verify signatures before deploy | Unsigned images can be tampered with in the registry | Confirm cosign verify runs before k8s deployment |

## 19. Infrastructure & Cloud Hardening

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 118 | Run containers as non-root with dropped capabilities | Root containers can escape to the host | Inspect Dockerfile/k8s manifest; confirm `runAsNonRoot: true` |
| 119 | Enable network policies / security groups with deny-by-default | Default-open networking lets attackers pivot between services | Confirm only required ports are open between services |
| 120 | Use IAM roles/service accounts (not static access keys) for cloud API access | Static keys are easily leaked and hard to rotate | Confirm cloud auth uses IRSA / Workload Identity |
| 121 | Enable audit logging on all cloud control-plane actions | Without audit logs you can't investigate breaches | Confirm CloudTrail / Activity Log is enabled |
| 122 | Apply security patches to base images within 30 days of upstream release | Unpatched images have known CVEs | Check the base image age; confirm < 30 days or auto-rebuild |

## 20. Documentation & Runbooks

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 123 | Maintain a runbook per critical service covering diagnosis, mitigation, escalation | On-call engineers need deterministic steps at 3am | Spot-check 3 runbooks for actionable, tested steps |
| 124 | Document the architecture with data flow + dependency graph | Without a diagram, new engineers can't understand the system | Confirm an architecture diagram exists and is current |
| 125 | Keep a CONTRIBUTING.md / DEVELOPMENT.md with setup instructions | New developers can't contribute without setup docs | Follow the setup guide from scratch; confirm it works |
| 126 | Document all environment variables with purpose, required?, default, secret? | Operators need to know what to set | Review `.env.example`; every var is documented |

## 21. Common Bug Prevention

| # | Checklist Item | Why It Matters | How to Verify |
|---|---|---|---|
| 127 | Protect shared mutable state with locks, atomics, or immutability | Race conditions corrupt data and are impossible to reproduce | Run `go test -race`; zero findings |
| 128 | Bound all in-memory caches/queues with eviction policies | Unbounded caches cause OOM kills over time | Review cache configs; every cache has max size + LRU/TTL |
| 129 | Explicitly close/release resources (defer rows.Close(), defer resp.Body.Close()) | Leaked resources exhaust the process and crash it | Run a static analyzer; zero unclosed-resource findings |
| 130 | Guard against integer overflow / divide-by-zero on user-supplied numerics | These crash processes with unhandled exceptions | Fuzz numeric inputs; confirm validated before use |
| 131 | Validate date/time handling across timezones and DST boundaries | Timezone bugs cause wrong scheduling and billing | Unit-test conversions across UTC and multiple local zones |
| 132 | Use `errors.Is()` / `errors.As()` for error comparison, never `==` | Wrapped errors fail `==` comparison, causing silent logic bugs | Grep for `err ==` comparisons on sentinel errors; confirm `errors.Is` |
| 133 | Implement `http.Flusher` + `Unwrap()` on all ResponseWriter wrappers | Missing Flusher breaks SSE streaming through middleware | Test SSE endpoint through the full middleware stack |
| 134 | Cast TEXT columns to UUID when joining to UUID columns | Type mismatch causes `operator does not exist: uuid = text` errors | Review all cross-table JOINs for type compatibility |
| 135 | Use `pq.Array()` for PostgreSQL array columns (not JSON marshal/unmarshal) | JSON format doesn't match Postgres array literal format | Test array column write + read round-trip |
| 136 | Add ownership checks before cross-resource writes (e.g., rate limits on API keys) | Missing ownership checks allow cross-tenant tampering | Attempt to write a rate limit on another user's API key; confirm 404 |

---

## Scoring Guide

| Score | Status | Action |
|-------|--------|--------|
| 136/136 | ✅ **Production Ready** | Ship it |
| 120–135 | ⚠️ **Almost Ready** | Fix remaining gaps before launch |
| 100–119 | 🟡 **Needs Work** | Multiple gaps — do not ship to production |
| < 100 | 🔴 **Not Ready** | Critical gaps — do not deploy |

---

## Priority Order (fix highest-risk items first)

1. **Items 1–8** (Authentication) — prevents account takeover
2. **Items 9–15** (Authorization) — prevents cross-tenant data access
3. **Items 16–23** (Injection) — prevents RCE and data exfiltration
4. **Items 24–29** (Secrets) — prevents credential leaks
5. **Items 45–57** (Database) — prevents data corruption and crashes
6. **Items 64–71** (Error handling) — prevents cascading failures
7. **Items 92–97** (Financial) — prevents double-charging and billing bugs
8. **Items 113–117** (Supply chain) — prevents dependency-based attacks

---

*This checklist is based on industry frameworks (OWASP, CWE, NIST, Google SRE, AWS ORR, PCI DSS, GDPR, HIPAA, SOC 2, ISO 27001) and real-world production incident postmortems. It should be reviewed quarterly as the threat landscape evolves.*

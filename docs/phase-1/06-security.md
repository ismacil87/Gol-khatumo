# 06 — Security Strategy

Target baseline: **OWASP ASVS 4.0 Level 2** and the **OWASP Top 10 (2021)**. Security requirements are part of each phase's acceptance criteria, not saved for the end.

## 1. Authentication

| Control | Design |
|---------|--------|
| Identifier | **Confirmed (C-15):** email address **or** mobile phone number (E.164). Every staff account has at least one; each is globally unique. The login form accepts either in one field. Login errors are generic ("Invalid credentials"), so they do not reveal which accounts exist. |
| Password hashing | **Argon2id** (memory 64 MiB, iterations 3, parallelism 1, re-tuned on production hardware to ~250 ms). Rehash on login if parameters change. |
| Password policy | ≥ 10 characters; checked against a list of common or breached passwords. No forced periodic rotation (NIST SP 800-63B). Must change on first login and after an admin reset. |
| Brute force | Per-account progressive lockout (5 failures → 15 min lock, then exponential); per-IP rate limit on `/auth/*` (Redis-backed); all failures logged. |
| Tokens | **Access token:** JWT (EdDSA or HS512), 15-minute lifetime, held in memory only, never in `localStorage`. **Refresh token:** opaque random 256-bit value, stored **hashed** in `sessions`, sent as an `HttpOnly; Secure; SameSite=Strict; Path=/api/v1/auth` cookie, 7-day sliding / 30-day absolute expiry. |
| Rotation & reuse detection | A new refresh token on every refresh. Reuse of an old token revokes the whole token family (theft detection). |
| Logout | Revokes the session. "Log out all devices" revokes all of the user's sessions. Deactivating a user revokes all of their sessions. |
| MFA | **[PROPOSED]** TOTP (RFC 6238) with recovery codes: **mandatory for Super Admin**, optional (school-enforceable) for School Admin and Accountant (Q-S1) |
| Password reset | v1: School Admin resets a staff password and a temporary password is shown once (staff may not have email). Self-service reset by email/SMS when a provider is configured (Q-N1). Reset tokens are single-use, hashed and expire after 30 minutes. |
| Super Admin accounts | Created only by CLI/seed or by another Super Admin; MFA required; login alerts. |

## 2. Authorization & tenant isolation
- Default deny. Every route declares its permission; services enforce scope (see 04 — Roles & Permissions §6).
- `school_id` is taken only from the verified token or session; client input is never trusted for it.
- Isolation layers: schema, composite FKs, Prisma tenant extension, authorization and automated tests (see 02 — Architecture §6).
- Insecure direct object reference (IDOR) protection: every lookup by ID is tenant-scoped. Cross-tenant and out-of-scope access returns 404.
- Mass assignment protection: Zod schemas are strict (unknown keys rejected). DTOs are allow-listed. `schoolId`, `status`, `createdBy` and similar fields are never accepted from the client unless a specific endpoint is meant to change them.

## 3. Input validation & output encoding
- **Server-side validation of every input** with Zod (type, length, format, range, enum). Client-side validation is for usability only.
- **SQL injection:** Prisma parameterised queries only. Raw SQL is restricted to a reviewed folder with tagged templates (`$queryRaw\`…\``), and `$queryRawUnsafe` is banned by lint rule.
- **XSS:** React escapes by default. `dangerouslySetInnerHTML` is banned by lint rule. PDF templates use an auto-escaping engine. Strict CSP (below).
- **CSV/Excel formula injection:** exported cells beginning with `= + - @ \t \r` are prefixed with `'`.
- **Request limits:** JSON body ≤ 1 MB, uploads ≤ configured size, query complexity bounded (page size ≤ 100, sort/filter allow-lists).

## 4. HTTP security
- TLS 1.2+ only, HSTS (`max-age=31536000; includeSubDomains`), HTTP → HTTPS redirect.
- `helmet` headers: `Content-Security-Policy` (`default-src 'self'; script-src 'self'; object-src 'none'; frame-ancestors 'none'; base-uri 'self'`; `img-src 'self' blob: data:` and the storage origin), `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy` minimal.
- **CORS:** allow-list of the web app origin(s) only, with credentials.
- **CSRF:** the access token travels in the `Authorization` header (not CSRF-able). The refresh cookie is `SameSite=Strict`, path-scoped, and the refresh endpoint also requires a custom header (`X-Requested-With`) and an `Origin` check.
- **Rate limiting:** global per-IP, per-user for write endpoints, stricter for auth, import and PDF endpoints.
- Errors never leak stack traces, SQL or internal IDs in production.

## 5. Secure file uploads
| Control | Design |
|---------|--------|
| Allow-list | Images (`jpg`, `png`, `webp`) for photos and logos; `pdf` for documents; `xlsx`, `xls`, `csv` for imports. Anything else is rejected. |
| Verification | Check **magic bytes** (`file-type`), not only the extension or the client's MIME type. |
| Size limits | Photos 2 MB, documents 10 MB, imports 5 MB (configurable) |
| Image sanitisation | Re-encode images with `sharp`: strips EXIF/GPS metadata and embedded payloads, resizes to max dimensions |
| Storage | Private bucket. Random UUID object keys under `schools/{schoolId}/…`. The original filename is stored only as metadata (sanitised). Never stored under the web root, never executed. |
| Access | Download via an authorising API endpoint → short-lived signed URL (≤ 5 min) with `Content-Disposition: attachment` for non-images |
| Malware scanning | **[PROPOSED]** ClamAV sidecar scan before a file is marked available (Phase 10; optional depending on hosting) |
| Spreadsheets | Parsed in the worker; macros (`.xlsm`) rejected; formulas read as cached values; row and column limits; parsing time-out |

## 6. Secrets & configuration
- All secrets (DB URL, JWT keys, storage keys, SMTP/SMS keys) come from environment variables injected by the platform secret manager. `.env` exists for local development only, is git-ignored, and `.env.example` documents the variables without values.
- The frontend bundle contains **only** public configuration (API base URL, build version). CI fails if known secret patterns appear in the `web` build output or the repository (secret scanning).
- JWT signing keys are rotatable (a `kid` header; two active keys during rotation).
- Field-level encryption (AES-256-GCM, key from the secret manager) for MFA secrets and optionally national ID numbers.

## 7. Data protection & privacy
- Data minimisation: collect only fields needed for school operations. Medical notes and staff sensitive data are behind dedicated permissions.
- Encryption at rest: managed database, backups and object storage encrypted with provider-managed or customer-managed keys.
- Logs scrub PII (passwords, tokens, phone numbers partially masked).
- Staging never contains real personal data unless anonymised.
- Per-school data export, and a documented deletion procedure on offboarding.

## 8. Audit logging
**Audited operations (minimum):** login success/failure, logout, password change/reset, MFA changes, user create/deactivate/role change, school create/suspend, support sessions, settings changes, student create/status change/transfer/delete-attempt, guardian contact create/change/replace, guardian contact list exports, billing status changes (including the School Admin's KEEP/WAIVE decision on existing unpaid charges with reason, user and date, C-18), charge generation runs, waivers, discounts, payment create/void/refund, expense approve/void, exam lock/publish/unlock, automatic end-of-year outcomes (system actor), placement decisions for PENDING PLACEMENT late joiners (decision, reason, user, date), graduations, result re-publication (including changes to ABSENT statuses), mark changes after lock, attendance edits after lock, imports committed, bulk exports of personal data, permission-denied events (sampled).

**Record contents:** actor, role, school, support session (if any), action, entity, before/after (redacted), reason, IP, user agent, request ID, timestamp.

**Integrity:** append-only (DB grants: INSERT/SELECT only); written in the same transaction as the business change; **[PROPOSED]** daily hash chain (each day's digest includes the previous one) for tamper evidence, Phase 10. Retention ≥ 2 years (Q-G5).

## 9. Backup & recovery strategy

| Item | Strategy |
|------|----------|
| Database (primary) | Managed MySQL automated daily snapshots + **binary log based point-in-time recovery** (PITR). **RPO ≤ 15 min** (typically ≤ 5 min). |
| Database (independent copy) | Nightly encrypted logical dump (`mysqldump --single-transaction` or `mydumper`), copied to a **separate account/region** bucket with object lock (immutability). Protects against provider, account or ransomware incidents. |
| Retention | Daily for 35 days, weekly for 12 weeks, monthly for 12 months; year-end snapshot kept 7 years **[PROPOSED]** (Q-G5) |
| Object storage | Versioning enabled; lifecycle rules; replication to the backup location |
| Redis | Not a system of record. Queued jobs can be re-created; no backup needed beyond provider persistence. |
| Configuration | Infrastructure as code and secrets manager (versioned) |
| Restore testing | **Quarterly restore drill** into an isolated environment: restore, run integrity checks (row counts, ledger balance reconciliation, sample PDFs), record time taken. RTO target ≤ 4 h. |
| Single-school restore | Procedure to restore one school's data from a backup into a scratch database and re-import selected records. Rare, but tenants share a database, so it must be documented. |
| Runbooks | Documented in `docs/runbooks/` in Phase 10: restore, PITR, key rotation, incident response, school offboarding |

## 10. Secure development lifecycle
- Pull request reviews required; a security checklist in the PR template.
- CI: `pnpm audit` / dependency review, static analysis (ESLint security rules, TypeScript strict), secret scanning, container image scan (Trivy or equivalent), SBOM generation.
- Dependabot/Renovate for dependency updates.
- Tenant isolation suite and authorization tests block merges.
- Pre-release: OWASP ZAP baseline scan on staging (Phase 10); external penetration test recommended before onboarding a second school.
- Incident response plan: detection (alerts), containment (revoke sessions, rotate keys), notification to affected schools, post-mortem.

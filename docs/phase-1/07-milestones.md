# 07 — Development Milestones & Acceptance Criteria

## 1. Definition of Done (applies to every phase)
A phase is complete only when **all** of the following hold:
1. Features meet the phase's acceptance criteria below and are demonstrable on a running build (local `docker compose up`, and staging from Phase 3).
2. TypeScript strict, lint and format pass with zero errors.
3. Automated tests pass in CI: unit, integration (real MySQL) and E2E smoke for the phase's user journeys. Domain service coverage ≥ 80%.
4. Tenant isolation and authorization tests exist for every new endpoint.
5. New endpoints are documented in OpenAPI. New screens are available in Somali and English, with no missing translation keys.
6. Database changes are delivered as reviewed Prisma migrations, plus seed updates.
7. UI is checked at 360 px, 768 px and 1280 px widths; keyboard navigation and labels pass an automated accessibility check (axe) with no serious violations.
8. Security checklist reviewed (validation, permissions, audit events, file handling).
9. A changelog and short phase report: what was delivered, how to test it, known limitations, and decisions made.
10. Stakeholder approval before the next phase starts.

## 2. Phase plan

### Phase 1 — Requirements & architecture *(this phase)*
**Deliverables:** this document set.
**Acceptance:** stakeholders approve the requirements, architecture, ERD, permission matrix and module plan, and answer or accept the defaults for the open questions in 08.

### Phase 2 — Project setup, database schema & backend foundation
**Deliverables**
- Monorepo (pnpm workspaces): `apps/api`, `apps/web` (empty shell), `packages/shared`; TS configs, ESLint, Prettier, Husky pre-commit.
- Docker Compose for local development: MySQL 8, Redis, MinIO, Mailpit; Dockerfiles for api/worker/web.
- Complete `schema.prisma` for **all** domains in 03 (so later phases add features, not table rewrites), first migration, seed (permissions, system roles, finance system categories, official grading scale, demo school).
- API core: config loader with env validation, structured logging, request IDs, error handling (problem+json), Zod validation middleware, pagination helpers, tenant context (AsyncLocalStorage) + Prisma tenant extension, permission guard, audit service, health/readiness, OpenAPI generation.
- Spike results confirmed: composite FKs via Prisma, Chromium PDF in container, SheetJS parsing of .xls.
- CI pipeline: lint, typecheck, unit + integration tests (Testcontainers), build.

**Acceptance criteria**
- `docker compose up` + `pnpm dev` starts the API, and `/health` and `/ready` report OK. `/api/docs` shows the OpenAPI UI.
- `prisma migrate deploy` on an empty database creates the full schema. Seed creates the system roles and permissions.
- A test proves the tenant extension: a query on a tenant model without context throws; with context it is scoped; a create with a foreign `schoolId` is rejected.
- A test proves the composite FK: inserting a mark that references another school's enrolment fails at database level.
- CI is green on the branch.

### Phase 3 — Authentication, user management & admin dashboard
**Deliverables**
- Auth: login, refresh rotation, logout, logout-all, change password, forced change, lockout, rate limits; TOTP MFA for Super Admin.
- Platform: Super Admin UI for schools (create with first School Admin, edit, suspend), support sessions.
- School: users and roles management, audit log viewer, school profile and settings, language switcher.
- Web shell: responsive layout (sidebar, top bar, mobile drawer), design-system primitives (Button, Input, Select, DatePicker, DataTable, Modal, Toast, Badge, EmptyState, Skeleton), i18n setup, permission-aware routing and menus.
- Dashboards: platform dashboard; school dashboard skeleton with placeholders for upcoming modules.
- In-app notification centre (core).

**Acceptance criteria**
- A Super Admin can onboard "Gol Khatumo School" and its School Admin, who can log in, is forced to change the password, and can create Principal, Accountant and Teacher users.
- Each role sees only permitted menu items. Direct API calls to forbidden endpoints return 403 (or 404 out of scope).
- Six failed logins lock the account. Refresh-token reuse revokes the session family. All of these are covered by tests.
- Audit log shows logins, user creation and role changes, with filters.
- Switching Somali/English changes all visible text immediately and persists across sessions.
- E2E: login → create user → logout passes on mobile and desktop viewports.

### Phase 4 — Students, parent information, teachers & staff
**Deliverables:** grade levels and sections (basic), student admission and profile, enrolment, status lifecycle and history, transfers, guardians and sibling links, staff records and documents, photo and document upload (secure pipeline), list views with search, filters and Excel export, duplicate warnings.

**Acceptance criteria**
- Admit a student with guardians, enrol them in a section, transfer them to another section, change status to WITHDRAWN with a reason. History shows every step.
- Duplicate admission number is rejected. Same name + DOB shows a warning.
- A teacher sees only students of assigned sections (verified once assignments exist; stubbed by class teacher in this phase). An accountant sees basic student data but cannot edit it.
- Upload rejects a renamed `.exe` as `.jpg` (magic-byte check). Photos are re-encoded without EXIF.
- Filtered student list exports to .xlsx with the same rows as the screen.

### Phase 5 — Academic management
**Deliverables:** academic years with their two terms (Mid Term, Final Term) created from the school's term template, term dates, set current and close year, calendar and holidays, full section management, subjects and curriculum, teaching assignments, bell schedule and timetable with conflict detection, student daily attendance (mark-all-present flow, lock window), staff attendance and leave, attendance reports, year-end rollover (preview/commit).

**Acceptance criteria**
- Creating an academic year for Gol Khatumo School produces exactly two terms, labelled "Mid Term" (sequence 1) and "Final Term" (sequence 2). Overlapping term dates, dates outside the year, or an extra term are rejected. A new year copies the two-term structure.
- Attendance is expected only on working days inside Mid Term or Final Term. Term attendance summaries (per student, per section) match the raw records.
- Timetable rejects a teacher double-booking and a section slot clash with clear localized messages.
- A teacher can take attendance only for assigned sections, on working days, within the edit window. The Principal can edit later, with a reason (audited).
- Monthly attendance register for a section matches the raw records. Holidays are excluded.
- Rollover is available only after Final Term results are published (stubbed until Phase 6). The preview lists promote/repeat/graduate per student. Commit creates next-year enrolments correctly. Re-running the commit does nothing.

### Phase 6 — Examinations & results
**Deliverables:** official grading scale (seeded, read-only), one examination per term with one paper per subject out of 100, lifecycle (draft → publish, unlock), mark entry grid (autosave, progress), marks computation, independent Mid Term and Final Term results (each out of 100%), rankings with tie rules, result snapshots, report card remarks, Mid Term and Final Term report card PDFs (first version), exam schedule per term, mark sheet/broadsheet.

**Acceptance criteria**
- Every exam belongs to Mid Term or Final Term, and paper dates outside that term are rejected. A second examination in the same term, a second paper for the same grade level and subject, or maximum marks other than 100 are rejected.
- **Grading scale boundaries:** automated tests cover every boundary of the official scale: 100 and 91 → A+, 90 and 85 → A-, 84 and 80 → B+, 79 and 75 → B, 74 and 70 → C+, 69 and 65 → C, 64 and 60 → D+, 59 and 50 → D, 49 and 0 → F. Non-whole percentages follow the rule confirmed in Q-E8.
- **PASS/FAIL:** a Final Term overall percentage of 50 gives PASS and 49.99 gives FAIL. Mid Term results show "no PASS/FAIL decision". A student who scored below 50 in Mid Term but at least 50 in Final Term is PASS, and the reverse is FAIL.
- The grading scale is read-only in v1. Any attempt to edit it is rejected, and published snapshots record the scale version.
- Publishing Mid Term produces Mid Term results, grades and ranks. Publishing Final Term produces Final Term results, grades and ranks. Each is calculated independently on a 100% scale and matches hand-calculated fixtures.
- No subject term mark can exceed 100, and the overall term percentage is between 0 and 100.
- **No annual result is produced:** tests assert that no combined or weighted annual percentage, grade or rank is calculated, stored, returned by the API or printed, and that the Final Term result is never labelled as an annual result. The annual result policy is fixed to `NONE`, and attempts to set another mode are rejected.
- Marks outside 0–100 are rejected. ABSENT/EXEMPT are handled per agreed rules (Q-E3).
- Teachers can enter marks only for assigned subject × section, and only during MARKS_ENTRY.
- Publishing produces snapshots. Editing marks afterwards requires unlock (audited) and re-publish. Published ranks never change silently.
- Ranking tests cover ties (1, 2, 2, 4), missing marks and exempt subjects, using fixtures with hand-calculated expected results.
- Mid Term and Final Term report card PDFs for a class render in Somali and in English with the correct term label; for each subject the mark, percentage and letter grade; term total, percentage, overall grade, rank, result status (Final Term PASS/FAIL; Mid Term "no PASS/FAIL decision"), grading scale legend and term attendance. The Final Term report card shows the Mid Term results in a separate column, with no combined or annual figure.

### Phase 7 — Finance & fee management
**Deliverables:** fee categories and structures, charge generation (preview, idempotent), ad-hoc charges, discounts, waivers with approval, billing profiles (FREE / FEE_PAYING) with approval and history, payments with allocation and credit, receipt numbering and **receipt PDF**, payment void/refund (maker-checker), outstanding balances and aging, class financial report, free student register, finance accounts, income/expenses with approval, daily cash report, fee statement PDF.

**Acceptance criteria**
- TERMLY fee items produce exactly one charge for Mid Term and one for Final Term per fee-paying student. Outstanding reports can be filtered by term.
- Generating charges twice for the same period creates no duplicates. Free students receive no charges, and manual attempts to charge them (non-allowed categories) are rejected.
- For a seeded scenario (paid, partial, unpaid, free, credit students), balances and statuses match hand-calculated expected values exactly. Class report totals reconcile: expected = collected + outstanding + waived/discounted.
- 100 concurrent payment requests produce 100 unique, gapless receipt numbers. An idempotent retry creates no duplicate payment.
- A voided payment keeps its receipt number, shows VOID, reverses its allocations and income transaction, and is audit-logged with the approver.
- Teachers cannot access any finance endpoint or financial fields (tests).

### Phase 8 — Advanced modules
**Deliverables:** library (catalogue, copies, circulation, overdue, lost-book charges), inventory (items, movements, suppliers, low-stock alerts), transport (vehicles, routes, stops, assignments, transport fees), notifications (event subscriptions, announcements, delivery preferences, channel abstraction), dashboard KPIs completed.

**Acceptance criteria**
- Issuing an ON_LOAN copy is rejected. Returning a copy updates its status. An overdue list and notification appear after the due date.
- Stock on hand equals the sum of movements. An OUT movement exceeding stock is rejected. A low-stock alert fires once per crossing.
- A student assigned to a route receives transport charges via the fee engine (unless FREE and the category is not allowed). Route capacity is enforced.
- Announcements reach only the targeted roles or users.

### Phase 9 — Excel integration, PDF reports & analytics
**Deliverables:** import engine (templates, validate, preview, duplicates, row errors, error report download, commit) for students, guardians, staff, fee structures, opening balances and marks; full PDF catalogue (02 §8.3) with shared layout kit; report centre with filters and PDF/Excel output; analytics charts; bulk PDF jobs with progress.

**Acceptance criteria**
- Each template downloads in Somali and English and re-imports without errors when filled with valid data.
- A test file with known errors (missing required fields, invalid dates, unknown section, duplicate admission numbers within the file and against the DB) yields exactly the expected row-level errors. The downloadable error report contains them.
- Nothing is saved before confirmation. A failed commit leaves no partial data.
- .xlsx, .xls and .csv inputs all work. Formula-injection payloads are neutralised in exports.
- Every PDF in the catalogue has a header with logo, title, academic year, date, page X of Y and signature section; previews, downloads and prints correctly on desktop and mobile; Somali characters render correctly.
- A bulk report card job for 300 students finishes within the target time and reports progress.

### Phase 10 — Testing, security, deployment & documentation
**Deliverables:** full regression and E2E suites, load test (target from NFR-P-01), OWASP ZAP baseline scan and remediation, dependency and container scanning, audit hash chain, ClamAV (if adopted), infrastructure as code for the selected cloud provider, production deployment, monitoring and alerts, backup automation and **a verified restore drill**, runbooks, API documentation, administrator and user guides (Somali and English), training materials, handover.

**Acceptance criteria**
- Load test meets p95 latency targets at the agreed concurrency.
- No high/critical findings open from the security scans.
- Restore drill completed within RTO; ledger reconciliation passes on the restored data.
- Production is live with TLS, monitoring, alerts and backups. The School Admin of Gol Khatumo School signs off UAT.

## 3. Indicative timeline
Durations depend on team size, which has not been confirmed (Q-G7). For a team of 2–3 full-stack engineers:

| Phase | Indicative duration |
|-------|---------------------|
| 2 | 2 weeks |
| 3 | 2–3 weeks |
| 4 | 2–3 weeks |
| 5 | 3 weeks |
| 6 | 3 weeks |
| 7 | 3–4 weeks |
| 8 | 3 weeks |
| 9 | 3 weeks |
| 10 | 2–3 weeks |
| **Total** | **≈ 23–27 weeks** |

Gol Khatumo School can begin a **pilot after Phase 7**, running students, academics and finance in production with real data while Phases 8–10 continue. This requires a Phase 10-lite hardening and backup step before the pilot (see Q-G8).

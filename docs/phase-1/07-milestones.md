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

- Auth: login with email **or** phone number (C-15), refresh rotation, logout, logout-all, change password, forced change, lockout, rate limits; TOTP MFA for Super Admin.
- Platform: Super Admin UI for schools (create with first School Admin, edit, suspend), support sessions.
- School: users and roles management, audit log viewer, school profile and settings, language switcher.
- Web shell: responsive layout (sidebar, top bar, mobile drawer), design-system primitives (Button, Input, Select, DatePicker, DataTable, Modal, Toast, Badge, EmptyState, Skeleton), i18n setup, permission-aware routing and menus.
- Dashboards: platform dashboard; school dashboard skeleton with placeholders for upcoming modules.
- In-app notification centre (core).

**Acceptance criteria**

- A Super Admin can onboard "Gol Khatumo School" and its School Admin, who can log in, is forced to change the password, and can create Principal, Accountant and Teacher users.
- Each role sees only permitted menu items. Direct API calls to forbidden endpoints return 403 (or 404 out of scope).
- A user can log in with their email address and, separately, with their phone number in any accepted format (normalised to E.164). Creating a user with neither, or with an email/phone already used by another account, is rejected (C-15).
- Six failed logins lock the account. Refresh-token reuse revokes the session family. All of these are covered by tests.
- Audit log shows logins, user creation and role changes, with filters.
- Switching Somali/English changes all visible text immediately and persists across sessions.
- E2E: login → create user → logout passes on mobile and desktop viewports.

### Phase 4 — Students, parent information, teachers & staff

**Deliverables:** grade levels and sections (basic), student admission and profile, enrolment, status lifecycle and history, transfers, guardians and sibling links, staff records and documents, photo and document upload (secure pipeline), list views with search, filters and Excel export, duplicate warnings.

**Acceptance criteria**

- Admit a student with guardians, enrol them in a section, transfer them to another section, change status to WITHDRAWN with a reason. History shows every step.
- Duplicate admission number is rejected. The same four-part name + DOB shows a warning.
- **Four-part name (C-12):** registration without any one of First Name, Father's Name, Grandfather's Name or Fourth Name is rejected with a field-level error, in the UI, the API and at database level.
- **Mandatory guardian (C-13):** registration without the guardian full name or phone is rejected, as is an invalid phone. Local formats are normalised to E.164. Removing the primary guardian without a replacement is rejected. Guardian name and phone are visible to School Admin, Principal, Accountant and to Teachers of the student's section; Super Admin is denied. Contact changes appear in the audit log. The guardian contact list exports to PDF and Excel.
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
- Rollover is available only after Final Term results are published (stubbed until Phase 6). The preview shows each student's automatic outcome (PROMOTED → next grade level, REPEAT GRADE → same grade level, GRADUATED → no next-year enrolment), plus a separate list of PENDING PLACEMENT late joiners. Automatic outcomes cannot be edited in the preview or through the API. Late joiners are placed only by the School Admin, with a reason. Commit creates next-year enrolments accordingly; re-running it does nothing.

### Phase 6 — Examinations & results

**Deliverables:** official grading scale (seeded, read-only), fixed absence rule (ABSENT = 0/100), decimal marks, one examination per term with one paper per subject out of 100, lifecycle (draft → publish, unlock), mark entry grid (autosave, progress), marks computation, independent Mid Term and Final Term results (each out of 100%), rankings with tie rules, result snapshots, report card remarks, Mid Term and Final Term report card PDFs (first version), exam schedule per term, mark sheet/broadsheet.

**Acceptance criteria**

- Every exam belongs to Mid Term or Final Term, and paper dates outside that term are rejected. A second examination in the same term, a second paper for the same grade level and subject, or maximum marks other than 100 are rejected.
- **Grading scale boundaries:** automated tests cover every boundary of the official scale: 100 and 91 → A+, 90 and 85 → A-, 84 and 80 → B+, 79 and 75 → B, 74 and 70 → C+, 69 and 65 → C, 64 and 60 → D+, 59 and 50 → D, 49 and 0 → F. Overall percentages use exact values, with no rounding (C-04): 90.6% → A-, 84.99% → B+.
- **PASS/FAIL from overall marks across all subjects (C-05)**, with hand-calculated fixtures:
  - Marks 40, 60, 55 → 155/300 = 51.666…% (shown 51.66%) → **PASS**, even though one subject is F (the subject still shows grade F).
  - Marks 45, 50, 54 → 149/300 = 49.666…% (shown 49.66%) → **FAIL**.
  - 7 subjects, 350/700 = 50.00% → **PASS**; 349/700 = 49.857…% → **FAIL**, displayed as 49.85% (truncated), never as 50% (C-04).
  - EXEMPT subjects are excluded from both totals. ABSENT counts as 0/100 and is included: marks 67.5, 55 and one ABSENT give 122.5/300 = 40.83% → FAIL.
  - **Mid Term PASS/FAIL (C-14):** Mid Term 155/300 → PASS and 149/300 → FAIL, calculated exactly like Final Term. A Mid Term FAIL **never** creates REPEAT GRADE, and a Mid Term PASS never creates PROMOTED or GRADUATED; outcomes are produced only when Final Term is published.
  - Combinations: Mid Term FAIL + Final Term PASS → PROMOTED; Mid Term PASS + Final Term FAIL → REPEAT GRADE. No combined or annual figure is calculated.
- **Automatic outcome (C-06):** publishing Final Term sets PROMOTED for every PASS and REPEAT GRADE for every FAIL, with no approval step. Outcomes appear on the student profile, the rollover preview and the Final Term report card. No role can override them: tests assert that the API has no endpoint for it and that attempts are rejected. Correcting Final Term marks through unlock → re-publish recalculates the outcome, audit-logged.
- **Graduation (C-09):**
  - In the grade level explicitly marked `is_final_grade`, PASS → GRADUATED, with no next-year enrolment and student status GRADUATED after rollover. FAIL → REPEAT GRADE.
  - **Negative case:** a grade level with no next grade that is **not** marked final never produces GRADUATED. Publishing its Final Term is rejected with `GRADE_PROGRESSION_NOT_CONFIGURED`, naming the grade level.
  - Marking a second grade level as final, or setting a next grade on the final grade, is rejected.
- **Late joiners (C-10):**
  - A student first enrolled after the first Final Term paper date is absent from the mark-entry grid and gets no ABSENT and no result. They receive PENDING PLACEMENT.
  - Only the School Admin can resolve it (Principal, Accountant and Teacher get 403). The reason is mandatory, and the audit log records decision, reason, user and date.
  - Control case: a student enrolled before the Final Term but absent from every paper gets ABSENT = 0/100 → 0% → FAIL → REPEAT GRADE, never PENDING PLACEMENT.
  - A section transfer during the year is not treated as late joining.
- The grading scale is read-only in v1. Any attempt to edit it is rejected, and published snapshots record the scale version.
- Publishing Mid Term produces Mid Term results, grades and ranks. Publishing Final Term produces Final Term results, grades and ranks. Each is calculated independently on a 100% scale and matches hand-calculated fixtures.
- No subject term mark can exceed 100, and the overall term percentage is between 0 and 100.
- **No annual result is produced:** tests assert that no combined or weighted annual percentage, grade or rank is calculated, stored, returned by the API or printed, and that the Final Term result is never labelled as an annual result. The annual result policy is fixed to `NONE`, and attempts to set another mode are rejected.
- **Decimal marks (C-08):** 0.00, 67.25, 78.50, 90.25 and 100.00 are accepted and stored exactly. -0.01, 100.01, 67.255 and 90.125 (more than two decimal places) are rejected with a validation error and are never rounded. Subject grades: 90.25 → A-, 90.5 → A-, 84.5 → B+, 64.5 → D+, 49.5 → F. Overall: 67.25 + 78.50 + 90.25 = 236.00/300 = 78.666…% (shown 78.66%) → B, PASS.
- **Absence rule (C-07):**
  - Recording ABSENT stores score 0 automatically. Sending a non-zero score with ABSENT is rejected.
  - ABSENT is included in both totals (0 obtained, 100 possible). The subject grade is F, and the overall grade, ranking, Final Term PASS/FAIL and outcome are calculated normally.
  - A student absent from every Final Term subject gets 0% → FAIL → repeat grade.
  - EXEMPT is excluded from both totals.
  - No API endpoint, setting or permission exists for choosing an absence policy (tests assert this).
  - Report cards and mark sheets show "ABSENT – 0/100".
  - Changing an ABSENT status on a published exam requires unlock → re-publish. The previous result version stays readable, and tests assert that no other write path changes published snapshots.
- Teachers can enter marks only for assigned subject × section, and only during MARKS_ENTRY.
- Publishing produces snapshots. Editing marks afterwards requires unlock (audited) and re-publish. Published ranks never change silently.
- **Ranking ties (C-11):** with exact overall percentages 91.50, 88.00, 88.00 and 80.25, the ranks are **1, 2, 2, 4**. Two students who both display 78.83% (78.833…% and 78.834%) are **not** tied; the higher exact value ranks first. The same results appear in section and grade-level ranks, Mid Term and Final Term, and every term ranking report. Students with incomplete marks or PENDING PLACEMENT are not ranked. All fixtures are hand-calculated.
- Mid Term and Final Term report card PDFs for a class render in the confirmed layout (C-20, 09 — Report Card Layout): one A4 page, **bilingual with Somali first and English second**, the correct term label and the student's four-part name (C-12); for each subject the mark, percentage and letter grade; term total, overall percentage (truncated), overall grade, **section rank and grade rank**, result status (Mid Term PASS/FAIL labelled as a progress assessment; Final Term PASS/FAIL with Promoted / Repeats / Graduated), grading scale legend, term attendance, remarks, class teacher, principal and **parent/guardian signature lines**, and a unique **document number** in the footer. The Final Term card shows the Mid Term results in a separate column headed **"Reference Only – Not Combined"**, with no combined or annual figure. Tests assert that **no student photo, no guardian name or phone and no QR code** are printed, and that reprinting the same result version keeps its document number.

### Phase 7 — Finance & fee management

**Deliverables:** fee categories and structures, charge generation (preview, idempotent), ad-hoc charges, discounts, waivers with approval, billing profiles (FREE / FEE_PAYING) with approval and history, payments with allocation and credit, receipt numbering and **receipt PDF**, payment void/refund (maker-checker), outstanding balances and aging, class financial report, free student register, finance accounts, income/expenses with approval, daily cash report, fee statement PDF.

**Acceptance criteria**

- TERMLY fee items produce exactly one charge for Mid Term and one for Final Term per student; ONE_TIME items exactly one charge per student per year; MONTHLY items one per month. A fee item with frequency ANNUAL is rejected (C-19). Outstanding reports can be filtered by term.
- All amounts are in USD; no other currency can be entered (C-16).
- Generating charges twice for the same period creates no duplicates. FREE students receive **no tuition charges**, and manual or imported tuition charges for them are rejected (`FEE_STUDENT_IS_FREE`); their non-tuition charges (e.g. books, uniform, transport) are generated and collected normally (C-17).
- Changing a student to FREE mid-year: only the School Admin can record the KEEP/WAIVE decision for existing unpaid charges; KEEP leaves them payable, WAIVE writes waiver adjustments that bring them to zero; the audit log shows the decision, reason, user and date; no tuition charge is generated after the effective date, while non-tuition charges continue (C-18).
- FREE students always show the FREE status; a FREE student with unpaid books, uniform or transport charges shows FREE together with UNPAID (or PARTIAL, PAID, CREDIT) for those charges, and those charges appear in outstanding and class financial reports. Setting FREE never reduces a non-tuition charge; only an approved waiver does (C-22).
- For a seeded scenario (paid, partial, unpaid, free without charges, free with non-tuition charges, credit students), balances and statuses match hand-calculated expected values exactly. Class report totals reconcile: expected = collected + outstanding + waived/discounted.
- 100 concurrent payment requests produce 100 unique, gapless receipt numbers. An idempotent retry creates no duplicate payment.
- A voided payment keeps its receipt number, shows VOID, reverses its allocations and income transaction, and is audit-logged with the approver.
- Teachers cannot access any finance endpoint or financial fields (tests).

### Phase 8 — Advanced modules

**Deliverables:** library (catalogue, copies, circulation, overdue, lost-book charges), inventory (items, movements, suppliers, low-stock alerts), transport (vehicles, routes, stops, assignments, transport fees), notifications (event subscriptions, announcements, delivery preferences, channel abstraction), dashboard KPIs completed.

**Acceptance criteria**

- Issuing an ON_LOAN copy is rejected. Returning a copy updates its status. An overdue list and notification appear after the due date.
- Stock on hand equals the sum of movements. An OUT movement exceeding stock is rejected. A low-stock alert fires once per crossing.
- A student assigned to a route receives transport charges via the fee engine, including FREE students, because transport is not tuition (C-17). Route capacity is enforced.
- Announcements reach only the targeted roles or users.

### Phase 9 — Excel integration, PDF reports & analytics

**Deliverables:** import engine (templates, validate, preview, duplicates, row errors, error report download, commit) for students, guardians, staff, fee structures, opening balances and marks; full PDF catalogue (02 §8.3) with shared layout kit; report centre with filters and PDF/Excel output; analytics charts; bulk PDF jobs with progress.

**Acceptance criteria**

- **Student template (FR-XLS-10):** rows missing any of the four name parts, the guardian full name or the guardian phone (or with an invalid phone) are listed as row-level errors and nothing is saved. A guardian phone matching an existing guardian is offered as a sibling link in the preview.
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

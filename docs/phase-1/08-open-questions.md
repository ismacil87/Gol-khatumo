# 08 — Risks, Technical Decisions & Open Questions

Every item below has a **proposed default**, so you can approve Phase 1 by accepting the defaults and changing only the ones you disagree with. Items marked ★ are **critical**: they change the database schema or financial behaviour and should be confirmed before Phase 2 starts.

## 0. Confirmed by the project owner

| ID | Decision | Effect on the documentation |
|----|----------|-----------------------------|
| C-01 | **Academic structure: two TERMS per academic year, not semesters.** Official labels: **1. Mid Term**, **2. Final Term**. No third term. | Requirements §1.6 (AS-1 to AS-6), FR-SCH-05/07, attendance, exams, results, ranking, report cards and fees use the two-term structure. ERD: `term.code` = `MID_TERM` / `FINAL_TERM`, `school_settings.term_template`, `fee_charge.term_id`. Module lifecycle and milestone acceptance criteria updated. Terms are stored as data per academic year (configurable for future years); the initial configuration contains exactly Mid Term and Final Term. |
| C-02 | **Grading and term results policy.** Mid Term and Final Term are each calculated independently on a 100% scale, with their own results and final grade (each subject's term result is out of 100). No combined annual result with any weighting, and the Final Term alone is not the annual result. Both term results are displayed separately. A future annual result policy may be configured by an authorised School Admin only after formal school approval. | Requirements §1.7 (GP-1 to GP-9), FR-EXM-01/05, FR-MRK-04/05/07/08, FR-RNK-01, FR-RPC-01/02, FR-SCH-07. ERD: `result_snapshot` scope limited to EXAM/TERM (YEAR reserved), new reserved `annual_result_policy` (mode `NONE`). Reserved permission `results.annual_policy.manage`. Lifecycle diagram, PDF catalogue and Phase 6 acceptance criteria updated. Previous proposal of a 50/50 annual weighting withdrawn. |
| C-03 | **Official grading scale and examination rules.** 91–100 A+, 85–90 A-, 80–84 B+, 75–79 B, 70–74 C+, 65–69 C, 60–64 D+, 50–59 D, 0–49 F. Each term has one examination per subject, marked out of 100. Percentages and letter grades are calculated automatically, independently for Mid Term and Final Term. **The Final Term score decides PASS (≥ 50) or FAIL (< 50)**; Mid Term stays separate and never overrides it. No annual combination. Report cards show marks, percentage, letter grade and the applicable result status. The scale may become configurable by an authorised administrator in future; v1 uses this scale only. | Requirements §1.7 (GP-3, GP-4, GP-8 to GP-11) and new §1.8 (GS-1 to GS-6); FR-EXM-01/02/05, FR-MRK-02/04, FR-RPC-01/02, FR-SCH-07. ERD: `exam_type` and exam weights removed; one exam per term; `max_marks` fixed at 100; `pass_marks` and `is_pass` removed; `grading_scale` gains version, pass threshold and read-only flag; `result_snapshot.result_status` and scale version. Permissions `grading.scale.read`, reserved `grading.scale.manage`. Phase 2 seed and Phase 6 acceptance criteria updated. Resolves Q-E1, Q-E2 and Q-E4; the earlier example scale is removed. PASS/FAIL detail refined by C-05; promotion by C-06. |
| C-04 | *Not assigned.* Decisions C-05 and C-06 keep the numbers given by the project owner. | — |
| C-05 | **Overall term PASS/FAIL calculation.** Overall percentage = (total marks obtained across all subjects taken ÷ total possible marks across those subjects) × 100. ≥ 50% = PASS; < 50% = FAIL. A student is not failed for scoring below 50% in an individual subject if the overall percentage is at least 50%. Subject grades are still calculated and shown separately. Exact percentages, with no rounding, decide grades and PASS/FAIL. | Requirements GP-3, GP-10, GP-12, GP-13, GS-3, GS-4, FR-MRK-04, FR-RPC-02. ERD: `result_snapshot.total_score`/`total_max` across all subjects; exact comparison on totals; `percentage` for display only (truncated). Phase 6 fixtures use overall totals. Resolves Q-E8 (rounding part) and Q-E9. |
| C-06 | **Final Term promotion / repetition policy.** Final Term overall percentage ≥ 50% → PASS, eligible to progress to the next grade; < 50% → FAIL, repeats the same grade in the next academic year. Automatic: no Principal or School Admin approval to decide PASS/FAIL or repetition. The outcome is recorded and displayed. Mid Term stays independent, is not combined with Final Term, and does not decide promotion. | Requirements GP-8, GP-11, FR-SCH-07, FR-STU-03, FR-RPC-01. ERD: `result_snapshot.end_of_year_outcome`, `enrollment.outcome` (PROMOTED / REPEAT_GRADE / GRADUATED) set automatically with `outcome_snapshot_id`. Permissions: no approval or override for any role; rollover only places students. Lifecycle diagram, audit list, Phase 5 and Phase 6 acceptance criteria updated. Resolves Q-E5. |
| C-07 | **Examination absence rule (revised by the project owner; replaces the earlier selectable policy and lock).** A student absent from a subject examination is recorded as ABSENT and automatically receives 0/100. The zero is included in the overall term percentage, and the subject grade and overall PASS/FAIL are calculated normally. There is no School Admin option to select a different absence policy, and the alternative policies ("absent = excluded", "absent = pending/make-up") are removed. | Requirements §1.9 (AP-1 to AP-6, rewritten), GP-13, FR-EXM-06, FR-MRK-02. ERD: `exam_absence_policy` and `exam_absence_policy_change` removed; `mark` check constraints (ABSENT ⇒ score 0); policy fields removed from `result_snapshot`. Absence-policy permissions and approval rows removed. Lifecycle, audit list and Phase 6 acceptance criteria updated. Resolves Q-E12 and Q-E13. |
| C-08 | **Decimal marks (precision confirmed via Q-E8).** Marks from 0.00 to 100.00 with up to two decimal places (e.g. 67.25, 78.50, 90.25). More than two decimal places is rejected. Exact values, never rounded, are used for grades and PASS/FAIL. | Requirements GP-3, GS-3, FR-MRK-02. ERD: `mark.score DECIMAL(5,2)` with API validation that rejects extra decimals (MySQL would round them silently). Phase 6 decimal tests. Resolves Q-E8. |
| C-09 | **Graduation.** A student who passes the Final Term in the school's highest grade receives GRADUATED and is not promoted to a non-existent next grade. | Requirements GP-8, GP-14, FR-SCH-07, FR-STU-03. ERD: `grade_level.next_grade_level_id` null = final grade; `end_of_year_outcome`/`enrollment.outcome` = GRADUATED; no next-year enrolment. Lifecycle and Phase 5/6 acceptance criteria. Resolves Q-E10. |
| C-10 | **Late joiners.** A student who was not enrolled when the Final Term examination took place receives PENDING PLACEMENT. The School Admin decides the placement, and the decision, reason, responsible user and date are recorded in the audit log. This never affects students who were enrolled but absent; they receive ABSENT = 0/100. | Requirements GP-15, FR-SCH-07, FR-STU-03, FR-RNK-02. ERD: `enrollment.outcome = PENDING_PLACEMENT`, `outcome_source`, `placement_*` fields; rule "no active enrolment on the first Final Term paper date". Permission `academic.placement.decide` (School Admin only); approval table and audit list updated. Phase 5/6 tests including the enrolled-but-absent control case. Resolves Q-E11. |
| C-11 | **Ranking ties.** Students with identical overall percentages share the same rank, using standard competition ranking (1, 2, 2, 4), applied consistently to all term ranking reports. | Requirements GP-16, FR-RNK-02, FR-SUB-01 (the per-subject "counts toward ranking" flag is removed, so ranking uses the same overall percentage as C-05). ERD: ranking rule with exact comparison of totals. Phase 6 ranking fixtures. Resolves Q-E3. |

## 1. Technical decisions for approval

| ID | Decision | Proposed | Alternatives | Why |
|----|----------|----------|--------------|-----|
| D-01 ★ | Tenancy model | Shared DB/schema + `school_id`, five isolation layers | DB per school | Lowest operating cost; one migration path; isolation enforced in layers and by tests. Can move a large school to a dedicated DB later. |
| D-02 | Repository layout | pnpm-workspaces monorepo (`apps/api`, `apps/web`, `packages/shared`) | Separate repositories | Shared types and validation without publishing packages; one CI |
| D-03 | Primary keys | UUIDv7 strings | Auto-increment integers | Do not leak counts; safe to generate in the application; time-ordered so they index well |
| D-04 | Background jobs | Redis + BullMQ | MySQL-backed job table (no Redis) | Mature retries, scheduling and progress. Redis also serves rate limiting. Adds one managed service. |
| D-05 | PDF engine | HTML/CSS → headless Chromium (Playwright) in a worker | pdfmake/PDFKit | Professional layouts, repeating table headers, charts, page X of Y, easier maintenance. Costs a heavier worker image. |
| D-06 | Excel libraries | SheetJS CE (official distribution, pinned) for parsing .xlsx/.xls/.csv; ExcelJS for styled .xlsx templates and exports | SheetJS only; ExcelJS only | ExcelJS cannot read legacy .xls; SheetJS CE cannot style output. Note: the SheetJS package on the npm registry is outdated, so it is installed from the vendor's official tarball, pinned and checksum-verified. |
| D-07 | Auth tokens | 15-min JWT in memory + rotating refresh token in httpOnly cookie, server-side sessions | Session cookies only | Works for a separate SPA/API origin, supports revocation and "log out all devices" |
| D-08 | UI primitives | Tailwind + Radix primitives (own design system) | A full component kit (MUI, Ant) | Accessible, lightweight, consistent with Tailwind, no heavy theme to override |
| D-09 | Validation & API docs | Zod shared schemas → OpenAPI generation | Hand-written OpenAPI | One source of truth for validation, types and documentation |

## 2. Business rules needing confirmation

### Platform & roles
| ID | Question | Proposed default |
|----|----------|------------------|
| Q-P1 | May the Platform Super Admin view school data? | Only through a time-limited, reason-logged, read-only **support session**, which is visible to the school |
| Q-P2 | Will the platform bill schools (subscriptions, plans, limits)? | Not in v1. The `schools` table reserves a `plan` field for later. |
| Q-R1 | Can schools create custom roles (e.g. Librarian, Receptionist, Deputy Principal)? | Yes, from the permission catalogue, in Phase 3 |
| Q-R2 | Maker–checker: may a School Admin approve their own request? | Yes by default (small schools), audited; a school setting can require a different approver |
| Q-S1 | Two-factor authentication | Mandatory for Super Admin; optional and school-enforceable for School Admin and Accountant |
| Q-S2 ★ | Login identifier for staff (many may not have email) | Email **or** phone number, both globally unique |

### Students & academics
| ID | Question | Proposed default |
|----|----------|------------------|
| Q-ST1 ★ | Name structure | Given name, father's name, grandfather's name, optional fourth/family name. Is this right for your records? |
| Q-ST2 | Admission number format | `GK-{YYYY}-{0000}`, auto-generated, editable for historical students |
| Q-ST3 | Grade structure at Gol Khatumo School | Please list the grades and typical sections (e.g. Grade 1–8 primary, Form 1–4 secondary) |
| Q-A1 | Student attendance granularity | **Daily** per section (per-period attendance is a later option) |
| Q-A2 | Who records teacher attendance? | School Admin/Principal or a delegate records it; no self check-in in v1 |
| Q-A3 | Attendance edit window for teachers | Same day until midnight school time; later edits by the Principal |
| Q-AY1 | Somali wording of the term labels | Use the official labels "Mid Term" and "Final Term" unchanged in both languages, unless the school provides official Somali labels |
| Q-AY2 | Typical term dates | Please give the usual start and end dates of Mid Term and Final Term, so they can pre-fill each new academic year |
| Q-E1 | Exams held in each term | **Resolved (C-03):** one examination per subject per term, marked out of 100 |
| Q-E2 | Composition of a term's 100 marks | **Resolved (C-03):** no composition; the single examination is the term result |
| Q-E3 | Ranking rules | **Resolved (C-11):** rank by exact overall term percentage; ties share a rank (1, 2, 2, 4); ABSENT counts as 0/100 (C-07); EXEMPT excluded; incomplete results and late joiners not ranked |
| Q-E4 | Grading scale | **Resolved (C-03):** the official scale in Requirements §1.8 |
| Q-E5 | Promotion rule | **Resolved (C-06):** automatic. PASS → promoted; FAIL → repeats the same grade. No approval or override. |
| Q-E6 | Report card types, language and layout | Two report cards per year: **Mid Term report card**, and **Final Term report card** showing Final Term results with the Mid Term results in a separate column (no combined or annual figure). Both show marks, percentage, letter grade and the applicable result status (C-03). Bilingual labels by default. Please share any current report card so its layout can be matched. |
| Q-E8 | Decimal places for marks | **Resolved (C-08):** up to two decimal places, 0.00–100.00; more decimals rejected; exact values, no rounding |
| Q-E9 | Effect of failing an individual subject | **Resolved (C-05):** no effect on PASS/FAIL if the overall percentage is ≥ 50%; the subject is shown with grade F |
| Q-E10 | Students in the highest grade who PASS the Final Term | **Resolved (C-09):** GRADUATED; never promoted to a non-existent grade |
| Q-E11 | Students not enrolled when the Final Term examination took place | **Resolved (C-10):** PENDING PLACEMENT; the School Admin decides placement with a reason (audited); enrolled-but-absent students get ABSENT = 0/100 |
| Q-E12 | Definitions of the absence-handling options | **Resolved (C-07, revised):** a single fixed rule. ABSENT = 0/100, included in the overall percentage. No alternative options. |
| Q-E13 | Scope of an absence-policy selection | **No longer applicable (C-07, revised):** there is nothing to select |

### Finance
| ID | Question | Proposed default |
|----|----------|------------------|
| Q-F1 ★ | Currency (or currencies) | One currency per school, **USD**. Should multi-currency payments (e.g. Somali/Somaliland shilling converted at a recorded rate) be supported? Default: no. |
| Q-F2 ★ | Do FREE students pay any fees (e.g. uniform, books, exam, transport)? | FREE = exempt from **all** generated fees; per category, a school can mark exceptions (`applies_to_free_students`) |
| Q-F3 | Partial discounts (siblings, staff children, partial scholarships)? | Yes: percentage or fixed, per student per category, with approval |
| Q-F4 ★ | When a student becomes FREE mid-year, what happens to existing unpaid charges? | The approver chooses KEEP or WAIVE; future charges stop |
| Q-F5 | Mobile money (EVC Plus, Zaad, eDahab, Sahal) | Recorded manually with provider and transaction reference; API integration is a later phase |
| Q-F6 | Expense approval | Expenses are recorded by the Accountant and approved by the School Admin before they count in reports |
| Q-F7 | Withhold report cards when fees are outstanding? | No by default; optional school setting |
| Q-F8 | Arrears from previous years | Stay on the old year's ledger and appear as arrears in balances and statements |
| Q-F9 ★ | Billing frequency of tuition | Monthly tuition (e.g. 10 months per year, each month assigned to Mid Term or Final Term), plus one-time registration and **termly** exam fees, charged once for Mid Term and once for Final Term. Or is tuition charged per term? Please confirm the actual fee schedule. |
| Q-F10 | Late payment penalties | None in v1 |
| Q-F11 | Receipt format | A4 by default, optional A5/half-page; bilingual; gapless numbering per school per year |

### General
| ID | Question | Proposed default |
|----|----------|------------------|
| Q-G1 | Library management staff | Custom role "Librarian" (needs Q-R1), or the School Admin |
| Q-G2 | Offline support (unreliable internet) | Not in v1; lightweight UI and retry-safe mutations. Offline attendance as a PWA enhancement after Phase 10 if needed. |
| Q-G3 | Expected scale (schools in 1–3 years, students per school) | 100 schools × 3,000 students design target |
| Q-G4 | Payroll | Out of scope; salaries recorded as expenses |
| Q-G5 | Data retention | Financial records 7 years; audit logs 2 years; student records indefinitely while the school is active |
| Q-G6 | Somali translations | We draft them; a native speaker from the school reviews them before each release. Who will review? |
| Q-G7 | Team and timeline | Please confirm team size and any target go-live date (e.g. start of a Mid Term or Final Term) |
| Q-G8 | Pilot | Pilot at Gol Khatumo School after Phase 7, with backups and basic hardening in place |
| Q-N1 | SMS/email to guardians | Not in v1; channel abstraction prepared. Which provider would you prefer later (local SMS gateway)? |
| Q-H1 | Hosting region / provider preference | Provider-independent until Phase 10. Consider a region with good latency to East Africa (e.g. a Middle East, East Africa or Europe region). |
| Q-H2 | Domain name | Needed before staging (e.g. `app.<your-domain>`) |

## 3. Gaps identified in the brief
1. **Payroll** is not listed but salaries are a major expense (Q-G4).
2. **Discounts / partial scholarships** sit between "free" and "fee-paying" and are common in practice (Q-F3).
3. **Opening balances** are needed to migrate existing debts on go-live (included in imports: FR-FEE-07).
4. **Year-end rollover / promotion** is not listed explicitly but is required (FR-SCH-07).
5. **Leave management** for staff relates to teacher attendance (FR-TAT-03).
6. **Data migration plan** from current records (paper/Excel), via the import engine (Phase 9). If go-live is earlier, basic student/staff import moves to Phase 4 (decision with Q-G7).
7. **Document verification** (QR on receipts and report cards) to fight forged documents (FR-PDF-07).
8. **Super Admin access policy** to school data (Q-P1).
9. **Training and support model** for school staff (Phase 10 guides; who provides first-line support?).

## 4. Risk register

| ID | Risk | Likelihood | Impact | Mitigation |
|----|------|-----------|--------|-----------|
| R-01 | Cross-school data leak via a missed tenant filter | Medium | Critical | Five isolation layers; composite FKs; automated isolation test for every route; code review checklist |
| R-02 | Unreliable internet at schools causes failed or double submissions | High | High | Idempotency keys; retry-safe TanStack mutations; small bundles; autosave for mark entry; offline PWA later |
| R-03 | Financial errors (wrong balances, duplicate receipts) damage trust | Medium | Critical | Ledger-derived balances; DB constraints; row locks; hand-calculated test scenarios; maker–checker; audit |
| R-04 | Exam and ranking rules differ from school practice | Low | High | All examination, grading, PASS/FAIL, promotion, absence, decimal, graduation, late-joiner and ranking rules confirmed (C-02, C-03, C-05 to C-11); boundary, decimal, ranking and overall-total fixtures validated with the school before Phase 6 sign-off |
| R-05 | Poor Somali translation quality | Medium | Medium | Native reviewer (Q-G6); glossary of school terms; context notes for translators |
| R-06 | PDF rendering resource use (Chromium) | Medium | Medium | Dedicated worker tier; queue with concurrency limits; warm browser pool; timeouts |
| R-07 | Scope creep across 25 modules | High | High | Phase gates; MoSCoW priorities; change requests reviewed against approved scope |
| R-08 | Data loss | Low | Critical | PITR + independent off-site dumps + quarterly restore drills |
| R-09 | Weak passwords / shared accounts among staff | High | Medium | Password policy, lockout, MFA for privileged roles, per-user accounts, audit, session listing |
| R-10 | Vendor lock-in before hosting is chosen | Low | Medium | Containers, S3-compatible storage, standard MySQL/Redis, adapters for email/SMS |
| R-11 | Dependency vulnerabilities (e.g. spreadsheet parsers) | Medium | High | Pinned, audited dependencies; parsing in an isolated worker with limits; Renovate updates |
| R-12 | Key-person dependency / knowledge loss | Medium | Medium | Documentation per phase, ADRs, runbooks, tests as specification |
| R-13 | Teachers struggling with the UI on small phones | Medium | Medium | Mobile-first flows for attendance and marks; usability test with real teachers in Phases 5–6 |
| R-14 | Confusion between the term named "Mid Term" and a mid-term exam | Low | Low | One examination per term, named "Mid Term Examination" / "Final Term Examination"; screens and PDFs always show the term label (AS-6) |
| R-15 | An unapproved annual calculation is introduced by mistake (e.g. Final Term treated as annual) | Low | High | Policy fixed to `NONE` in v1; explicit negative tests in Phase 6; any future policy requires recorded formal approval and an audit log |
| R-16 | A student is wrongly made to repeat a grade because of a calculation error | Low | High | Exact-total comparison (no rounding); hand-calculated fixtures; outcomes audit-logged; correction via audited unlock → re-publish, which also moves the next-year enrolment |
| R-17 | Absent students wrongly left out of totals, or ABSENT changed after publication without trace | Low | High | Fixed rule (ABSENT ⇒ score 0) enforced by database check constraints and service; no configuration to change it; changes after publication only via audited unlock → re-publish with versioned results (C-07) |

## 5. Architecture decision records
From Phase 2 onward, each approved decision (D-01 to D-09 and any later ones) is recorded as an ADR in `docs/adr/NNNN-title.md`: context, decision, consequences, status. That gives future maintainers a traceable history.

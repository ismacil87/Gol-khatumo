# GKSMS — Phase 1: Architecture & Planning

**Project:** Gol Khatumo School Management System (GKSMS)
**Phase:** 1 of 10 — requirements analysis, system architecture, database ERD, permissions, module relationships and technical specifications
**Status:** FINAL DRAFT. Decisions C-01 to C-47 are confirmed by the project owner. Open: school-specific information (Q-ST3, Q-AY2, Q-G6, Q-G7, Q-H2, Q-G9) and minor proposals (Q-M1 to Q-M10). No application code is written until the project owner separately authorises Phase 2.
**Date:** 2026-11-23

---

## Document set

| # | Document | Contents |
|---|----------|----------|
| 01 | [Requirements](01-requirements.md) | Project overview, scope, functional requirements by module, non-functional requirements |
| 02 | [System Architecture](02-architecture.md) | Technology stack, logical architecture, repository structure, API conventions, multi-school tenancy strategy, Excel and PDF architecture, localization, cloud deployment strategy |
| 03 | [Database Design & ERD](03-database-erd.md) | Data modelling conventions, entity-relationship diagrams by domain, key business invariants, indexing |
| 04 | [Roles & Permissions](04-roles-permissions.md) | Role definitions, permission model, permission matrix, data scoping rules, sensitive operations |
| 05 | [Modules & Dependencies](05-modules.md) | Purpose of each of the 25 modules, the entities each one owns, dependency graph, build order |
| 06 | [Security Strategy](06-security.md) | Authentication, authorization, OWASP controls, file uploads, secrets, audit logging, backup and recovery |
| 07 | [Milestones & Acceptance Criteria](07-milestones.md) | Phase 2–10 deliverables, definition of done, acceptance criteria |
| 08 | [Risks, Decisions & Open Questions](08-open-questions.md) | Decision register (C-01 to C-47), technical decisions, business rules, remaining open questions, gaps, risks |
| 09 | [Report Card Layout](09-report-card-layout.md) | Mid Term and Final Term report card layout. **Confirmed (C-20)** |

## Approved baseline and confirmed decisions

| Area | Decision |
|------|----------|
| First school | Gol Khatumo School |
| Tenancy | Multi-school (multi-tenant): one shared database, every record isolated by `school_id`, isolation enforced server-side in five layers; no school can access another school's records (confirmed, D-01 / C-21) |
| Hosting | Cloud, provider-independent; region chosen to suit users in East Africa (confirmed, C-47) |
| Languages | Somali and English, with a language switcher |
| Academic structure | Two **terms** per academic year (not semesters): **Mid Term** and **Final Term** (confirmed, C-01) |
| Grading | Mid Term and Final Term each calculated independently out of 100%, shown separately; no combined annual result (confirmed, C-02) |
| Grading scale | 91–100 A+, 85–90 A-, 80–84 B+, 75–79 B, 70–74 C+, 65–69 C, 60–64 D+, 50–59 D, 0–49 F. One exam per subject per term out of 100 (confirmed, C-03). PASS/FAIL per term as below; only the Final Term decides promotion (C-06, C-14) |
| PASS/FAIL | Both terms: overall percentage across all subjects (total obtained ÷ total possible × 100): ≥ 50% PASS, < 50% FAIL. One failed subject does not fail the student (C-05). Exact values decide; displayed percentages are truncated, never rounded (C-04). Mid Term PASS/FAIL is a progress assessment only (C-14) |
| Promotion | Automatic: PASS → next grade (GRADUATED in the highest grade, C-09), FAIL → repeat the same grade; no approval; outcome recorded and displayed. Mid Term does not decide promotion (C-06). Late joiners → PENDING PLACEMENT, placed by the School Admin with an audited reason (C-10) |
| Exam absence | ABSENT = automatic 0/100, included in the overall percentage; grades and PASS/FAIL calculated normally; no option to choose another policy (confirmed, C-07 revised) |
| Marks | 0.00–100.00 with up to two decimal places, e.g. 67.25, 78.50, 90.25; more decimals rejected; no rounding (confirmed, C-08) |
| Ranking | Exact overall term percentage; ties share a rank with competition ranking 1, 2, 2, 4 (confirmed, C-11) |
| Student name | Four mandatory parts: First, Father's, Grandfather's, Fourth Name (confirmed, C-12) |
| Parent/guardian | Full name and phone number mandatory for every student (confirmed, C-13) |
| Users (v1) | Staff only, logging in with email **or** phone number (confirmed, C-15). Parents and students have no login. |
| Currency | USD only in the initial release (confirmed, C-16) |
| FREE students | Exempt from tuition fees only; other configured charges (books, uniforms, transport, …) may apply (confirmed, C-17). Becoming FREE mid-year: the School Admin decides KEEP or WAIVE for existing unpaid charges, audit-logged; future tuition stops (confirmed, C-18) |
| Fee frequencies | MONTHLY, TERMLY, ONE_TIME; no ANNUAL (confirmed, C-19) |
| FREE fee status | FREE always displayed; other charges keep their own PAID / PARTIAL / UNPAID / CREDIT status; FREE never waives them (confirmed, C-22) |
| Report card | One-page A4, bilingual (Somali first), no photo, no guardian contact, Mid Term column "Reference Only – Not Combined", section and grade rank, parent signature, document number; QR deferred (confirmed, C-20) |
| Roles | Platform Super Admin, School Admin, Principal, Accountant, Teacher, plus per-school custom roles from Phase 3 (C-27). Super Admin sees school data only through read-only, time-limited, logged support sessions (C-25). School Admin self-approval allowed and audited; second approver optional (C-28) |
| Attendance | Daily per section; staff attendance by School Admin or Principal; teachers edit their own sections until midnight the same day (C-31) |
| Admission numbers | `GK-{YYYY}-{0000}` (C-30) |
| Late Mid Term joiners | "—" and "No Mid Term Result"; no artificial zero or FAIL; Final Term alone decides the outcome (C-23) |
| Finance (v1) | Fixed and percentage discounts with approval (C-33); mobile money recorded manually (C-34); expenses recognised after School Admin approval (C-35); report cards never withheld for fees (C-36); arrears stay on the previous year (C-37); no late penalties (C-38); A4 or half-page bilingual receipts, gapless numbering (C-39) |
| Out of v1 | School billing (C-26), payroll (C-43), offline mode (C-41), SMS/email (C-46), QR verification (C-20) |
| Capacity and retention | 100 schools × up to 3,000 students (C-42); financial records 7 years, audit logs 2 years, subject to law (C-44) |
| Pilot | Gol Khatumo School after Phase 7 (C-45) |
| Backend | Node.js + Express.js + TypeScript |
| Frontend | React + TypeScript + Tailwind CSS + Radix UI (C-24) |
| Database / ORM | MySQL 8 + Prisma (with migrations) |
| Files | Excel import/export (.xlsx, .xls, .csv); direct PDF generation |
| Devices | Mobile, tablet, desktop |

## Confirmed technical decisions and remaining open items

All decisions are listed in [08 — Open Questions](08-open-questions.md): **C-01 to C-47 are confirmed** (08 §0). Items still open are marked **[PROPOSED]** where they appear. The decisions that most affect Phase 2:

1. **Tenancy model (confirmed, D-01 / C-21):** one shared database and schema, a `school_id` column on every school-owned table, and isolation enforced in five layers: schema, composite foreign keys, a Prisma tenant extension, authorization checks and automated tests (02 — Architecture §6.2).
2. **One code repository (confirmed, D-02 / C-24):** pnpm workspaces with `apps/api`, `apps/web` and `packages/shared`.
3. **PDF engine (confirmed, D-05 / C-24):** HTML/CSS templates rendered to PDF by headless Chromium in a background worker.
4. **Background jobs (confirmed, D-04 / C-24):** Redis + BullMQ for PDF batches, imports and notifications.
5. **Excel, UI and validation (confirmed, D-06, D-08, D-09 / C-24):** SheetJS CE + ExcelJS; Tailwind CSS + Radix UI; Zod with generated OpenAPI. UUIDv7 primary keys (D-03).
6. **Login (confirmed, C-15, D-07, C-29):** email or phone number plus password; short-lived access token plus a rotating refresh token in an httpOnly cookie; two-factor authentication mandatory for Super Admin, optional for School Admin and Accountant.
7. **Fee model:** charges (invoices) are generated from fee structures. Free students never receive tuition charges; other configured charges still apply (C-17). Payments are allocated to charges and are never deleted, only voided. Retries are safe (C-41).

**Still open:** school-specific information only the school can give (Q-ST3 grade structure, Q-AY2 term dates, Q-G6 Somali reviewer, Q-G7 team and go-live date, Q-H2 domain name, Q-G9 first-line support) and ten minor proposals with recommended defaults (Q-M1 to Q-M10, 08 §2a).

## Next step

Provide the school-specific information listed above when available and accept or change the minor proposals (Q-M1 to Q-M10). Phase 2 (project setup, database schema and backend foundation) starts **only after the project owner's separate, explicit authorisation**.

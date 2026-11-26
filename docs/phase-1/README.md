# GKSMS — Phase 1: Architecture & Planning

**Project:** Gol Khatumo School Management System (GKSMS)
**Phase:** 1 of 10 — requirements analysis, system architecture, database ERD, permissions, module relationships and technical specifications
**Status:** FINAL DRAFT for approval. All critical decisions are confirmed (C-01 to C-22); the remaining open questions have proposed defaults. No application code is written until the project owner approves Phase 1 and authorises Phase 2.
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
| 08 | [Risks, Decisions & Open Questions](08-open-questions.md) | Technical decisions that need approval, business rules that need confirmation, gaps, risks |
| 09 | [Report Card Layout](09-report-card-layout.md) | Mid Term and Final Term report card layout. **Confirmed (C-20)** |

## Approved baseline (from the project brief)

| Area | Decision |
|------|----------|
| First school | Gol Khatumo School |
| Tenancy | Multi-school (multi-tenant): one shared database, every record isolated by `school_id`, isolation enforced server-side in five layers; no school can access another school's records (confirmed, D-01 / C-21) |
| Hosting | Cloud, provider-independent until a provider is chosen |
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
| Roles | Platform Super Admin, School Admin, Principal, Accountant, Teacher |
| Backend | Node.js + Express.js + TypeScript |
| Frontend | React + TypeScript + Tailwind CSS |
| Database / ORM | MySQL 8 + Prisma (with migrations) |
| Files | Excel import/export (.xlsx, .xls, .csv); direct PDF generation |
| Devices | Mobile, tablet, desktop |

## Decisions in this document that need your approval

The documents use proposed defaults where the brief leaves something open. Each one is marked **[PROPOSED]** where it appears and is listed with its number in [08 — Open Questions](08-open-questions.md). Confirmed decisions are listed as C-01 to C-22 in 08 §0. The ones that most affect Phase 2:

1. **Tenancy model (confirmed, D-01 / C-21):** one shared database and schema, a `school_id` column on every school-owned table, and isolation enforced in five layers: schema, composite foreign keys, a Prisma tenant extension, authorization checks and automated tests (02 — Architecture §6.2).
2. **Monorepo:** pnpm workspaces with `apps/api`, `apps/web` and `packages/shared`.
3. **PDF engine:** HTML/CSS templates rendered to PDF by headless Chromium in a background worker.
4. **Background jobs:** Redis + BullMQ for PDF batches, imports and notifications.
5. **Fee model:** charges (invoices) are generated from fee structures. Free students never receive tuition charges; other configured charges still apply (C-17). Payments are allocated to charges and are never deleted, only voided.
6. **Login identifier (confirmed, C-15):** email or phone number plus password. Short-lived access token, plus a rotating refresh token in an httpOnly cookie.

## Next step

Review the nine documents, answer the remaining open questions in [08 — Open Questions](08-open-questions.md) (every one has a proposed default you can accept), and approve Phase 1. Phase 2 (project setup, database schema and backend foundation) starts only after that.

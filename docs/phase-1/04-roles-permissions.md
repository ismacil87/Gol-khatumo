# 04 — User Roles & Permission Model

## 1. Model

GKSMS uses **role-based access control with granular permissions, plus data-scope rules**:

```
User ──< UserRole >── Role ──< RolePermission >── Permission ("module.action")
                                         +
                     Scope rules evaluated in services (tenant, assignment, ownership)
```

- **Permission** = `module.action`, e.g. `students.read`, `marks.enter`, `payments.void`. The catalogue is defined in code (`packages/shared/permissions.ts`), seeded into the database and versioned with the application.
- **Role** = a named set of permissions. The five **system roles** are seeded and cannot be deleted, and their permissions cannot be changed by schools. **[PROPOSED]** Schools may create **custom roles** (e.g. Librarian, Receptionist, Deputy Principal) from the permission catalogue, but cannot grant platform permissions (Q-R1).
- **Scope** = which records within a granted permission a user can reach. It is enforced in services, not only in the UI:
  - `TENANT`: all records of the user's school (default for school roles).
  - `ASSIGNED`: only sections/subjects linked through `teaching_assignment` or `section.class_teacher_id` (Teacher).
  - `OWN`: only the user's own records (own profile, own attendance, own sessions).
  - `PLATFORM`: cross-school platform objects (Super Admin).
- A user may hold several roles in one school (e.g. a Principal who also teaches). Effective permissions are the **union**; scopes are the **widest** granted per permission.
- The JWT carries `permission_version`. When a user's roles change, the version increments and the next refresh issues updated permissions. Permissions are always re-checked on the server; the frontend uses them only to hide UI.

## 2. Role definitions

| Role | Scope | Purpose |
|------|-------|---------|
| **Platform Super Admin** | Platform | Operates the platform: onboards and suspends schools, creates first School Admins, manages platform settings and system health, views platform audit logs. **No routine access to school data**; time-limited, audited support sessions only (Q-P1). |
| **School Admin** | Whole school | The school's owner and administrator: settings, users and roles, all modules, approvals for sensitive operations (free status, voids, waivers, exam unlocks). |
| **Principal** | Whole school, academic focus | Academic leadership: academic years, classes, subjects, timetables, exams (approve, lock, publish), results, report cards, attendance oversight, staff attendance. Read-only finance summaries. |
| **Accountant** | Whole school, finance focus | Fee structures, charges, payments, receipts, balances, income/expenses, financial reports. Can *request* free status and waivers; cannot approve their own requests. Read-only student and guardian contact data. |
| **Teacher** | Assigned sections/subjects | Takes attendance for assigned sections, enters marks for assigned subject × section during marks entry, sees own timetable and own students' academic data. **No access to fees or finance.** |

## 3. Permission catalogue (v1)

| Module | Permissions |
|--------|-------------|
| Platform | `platform.schools.manage`, `platform.settings.manage`, `platform.users.manage`, `platform.audit.read`, `platform.support.start` |
| School settings | `school.settings.read`, `school.settings.manage` |
| Academic years/terms | `academic.years.read`, `academic.years.manage`, `academic.rollover.run` |
| Classes/sections | `classes.read`, `classes.manage` |
| Subjects/assignments | `subjects.read`, `subjects.manage`, `assignments.manage` |
| Timetable | `timetable.read`, `timetable.manage` |
| Students | `students.read`, `students.create`, `students.update`, `students.status.change`, `students.transfer`, `students.medical.read`, `students.documents.manage`, `students.export` |
| Guardians | `guardians.read`, `guardians.manage` |
| Staff | `staff.read`, `staff.manage`, `staff.sensitive.read` (salary, national ID), `staff.export` |
| Student attendance | `attendance.read`, `attendance.take`, `attendance.edit_locked` |
| Staff attendance | `staff_attendance.read`, `staff_attendance.record`, `staff_leave.approve` |
| Exams | `exams.read`, `exams.manage`, `exams.lock`, `exams.publish`, `exams.unlock` |
| Grading scale | `grading.scale.read`; `grading.scale.manage` (reserved for future editing by an authorised administrator, inactive in v1: the official scale is read-only) |
| Marks/results | `marks.read`, `marks.enter`, `marks.import`, `results.read`, `rankings.read`, `results.annual_policy.manage` (reserved for the future annual result policy, School Admin only, inactive in v1) |
| Report cards | `report_cards.generate`, `report_cards.remarks.manage` |
| Fees | `fees.structures.read`, `fees.structures.manage`, `fees.charges.generate`, `fees.charges.adhoc`, `fees.charges.waive.request`, `fees.charges.waive.approve`, `fees.discounts.manage` |
| Free students | `billing.status.read`, `billing.status.request`, `billing.status.approve` |
| Payments | `payments.read`, `payments.create`, `payments.void.request`, `payments.void.approve`, `payments.refund` |
| Balances | `balances.read` |
| Income/expenses | `finance.read`, `finance.record`, `finance.approve`, `finance.accounts.manage` |
| Library | `library.read`, `library.manage`, `library.circulate` |
| Inventory | `inventory.read`, `inventory.manage`, `inventory.move` |
| Transport | `transport.read`, `transport.manage` |
| Notifications | `notifications.announce` (reading one's own notifications is implicit) |
| Reports | `reports.academic`, `reports.financial`, `reports.operational`, `reports.export` |
| Users/roles | `users.read`, `users.manage`, `roles.manage` |
| Audit | `audit.read` |
| Imports | `imports.students`, `imports.staff`, `imports.fees`, `imports.marks` |

## 4. Permission matrix

Legend: **F** = full (all actions in the module), **R** = read only, **A** = can approve, **Rq** = can request (needs approval), **S** = scoped to assigned sections/subjects, **O** = own records only, **—** = no access.

| Module / capability | Super Admin | School Admin | Principal | Accountant | Teacher |
|---|:---:|:---:|:---:|:---:|:---:|
| Schools (create, suspend, platform settings) | F | — | — | — | — |
| Platform audit, health, statistics | F | — | — | — | — |
| School data (any module) | Support session only | — | — | — | — |
| Dashboard | Platform | School | Academic | Finance | Own classes |
| School profile & settings | — | F | R | R | — |
| Academic years, terms, calendar | — | F | F | R | R |
| Year-end rollover / promotion | — | F | F | — | — |
| Grade levels & sections | — | F | F | R | S (R) |
| Subjects & teaching assignments | — | F | F | — | S (R) |
| Timetable | — | F | F | — | S (R, own) |
| Students: view | — | F | F | R (basic) | S (R) |
| Students: create/edit/transfer/status | — | F | F | — | — |
| Students: medical notes | — | F | F | — | S (R) **[PROPOSED]** |
| Guardians / parent info | — | F | F | R (contact) | S (R, contact) |
| Staff records | — | F | R | R (basic) | O |
| Staff sensitive data (salary, ID) | — | F | — | R | — |
| Student attendance: take | — | F | F | — | S |
| Student attendance: edit after lock window | — | F | F | — | — |
| Staff attendance & leave | — | F | F | — | O (R) |
| Exams: create, schedule | — | F | F | — | — |
| Exams: lock / publish / unlock | — | F | F | — | — |
| Marks entry | — | F | F | — | S |
| Marks import (Excel) | — | F | F | — | S |
| Results, rankings | — | F | F | — | S (R) |
| Grading scale: view | — | R | R | — | R |
| Grading scale: edit | — | — (reserved, future) | — | — | — |
| Report cards: generate | — | F | F | — | S (own sections) |
| Report card remarks | — | F | F (principal) | — | S (class teacher) |
| Fee categories & structures | — | F | R | F | — |
| Generate charges | — | F | — | F | — |
| Ad-hoc charges | — | F | — | F | — |
| Discounts | — | F | — | Rq | — |
| Waive charges | — | A | — | Rq | — |
| Free student status | — | A | Rq | Rq | — |
| Payments & receipts: record, print | — | F | R | F | — |
| Payments: void / refund | — | A | — | Rq | — |
| Outstanding balances | — | F | R | F | — |
| Income & expenses | — | F + A | R | F (record) | — |
| Library | — | F | R | — | R (catalogue) |
| Inventory | — | F | R | F | — |
| Transport | — | F | R | R (+ fees) | — |
| Announcements | F (platform-wide) | F | F | — | — |
| Notifications: receive | Own | Own | Own | Own | Own |
| Reports: academic | — | F | F | — | S |
| Reports: financial | — | F | R (summaries) | F | — |
| Reports: operational (library/inventory/transport) | — | F | R | R | — |
| Excel export of lists | — | F | F | Finance + basic student | S |
| Users & roles | Platform users + first school admin | F | — | — | — |
| Audit logs | Platform | School | R (academic events) **[PROPOSED]** | — | — |
| Own profile, password, sessions, language | O | O | O | O | O |

## 5. Sensitive operations (maker–checker and audit)

| Operation | Requester | Approver | Notes |
|-----------|-----------|----------|-------|
| Grant/revoke FREE status | Accountant, Principal | School Admin | Reason category + notes required. Choice of KEEP/WAIVE for existing charges. |
| Waive a charge | Accountant | School Admin | Reason required |
| Void a payment | Accountant | School Admin | Receipt stays, marked VOID; income transaction voided |
| Refund credit | Accountant | School Admin | Creates an expense/refund transaction |
| Unlock a locked/published exam | Principal or School Admin | — (self, with reason) | Snapshots invalidated; notification to School Admin |
| Edit attendance after lock window | Principal/School Admin | — | Audited with reason |
| Change a user's roles | School Admin | — | Audited; permission version bumped; sessions refreshed |
| Super Admin support session | Super Admin | — | Reason, time limit; visible in the school's audit log |

**[PROPOSED]** A School Admin can approve their own requests, because small schools may have only one admin; the audit log shows this. This is configurable per school (Q-R2).

## 6. Enforcement checklist (applies to every endpoint)
1. Route declares its required permission(s). A lint/test fails if a tenant route has none.
2. Service applies scope: tenant (automatic), assigned (teacher), own.
3. Financial fields (balances, fee status, payments) are stripped by the DTO mapper for users without `balances.read`, including inside student profile and export responses.
4. Out-of-scope record → `404 Not Found`. In-scope record but forbidden action → `403 Forbidden`.
5. Permission-denied events are logged at warning level, and repeated attempts are rate-limited.

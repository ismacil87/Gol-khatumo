# 09 — Report Card Layout

**Status: CONFIRMED (C-20).** The project owner reviewed and approved all nine design choices of Q-E6 (§6). The content rules this layout applies (four-part name, marks, percentages, grades, PASS/FAIL, outcomes, ABSENT, ranking, rounding, Mid Term late joiners) come from confirmed decisions C-01 to C-14 and C-23.

## 1. Design principles

- **One page, A4 portrait** (210 × 297 mm), margins 12 mm. It prints cleanly in black and white: status is never shown by colour alone.
- **Bilingual, Somali first and English second** (e.g. "Natiijada / Result"), on every card (C-20, Q-E6.7). The Somali wording is accepted **provisionally** and is reviewed by a Somali language specialist before the pilot (Q-E6.8, Q-G6). The term labels stay "Mid Term" and "Final Term" (C-32).
- **Two card types**: the **Mid Term report card** and the **Final Term report card**. There is no annual card, annual column, combined percentage or annual rank (C-02).
- **Only published results** appear (FR-RPC-05). Every figure comes from the term's result snapshot, so the card always matches the system.
- **Percentages are shown to 2 decimals, truncated, never rounded up.** Grades and PASS/FAIL are decided on exact values (C-04).
- **Never withheld for unpaid fees** in v1, and no fee information appears on the card (C-36).
- **No student photo and no parent/guardian contact details** are printed (Q-E6.2, Q-E6.9). Both stay in the student record.

### 1.1 Mandatory content

Every report card contains:

- **Separate Mid Term and Final Term results.** Each term's marks, percentage, grade and PASS/FAIL are calculated and shown on their own; they are never combined into an annual figure (C-02, C-14).
- The student's **four-part name**: First Name, Father's Name, Grandfather's Name, Fourth Name (C-12).
- The **overall term percentage** (2 decimals, truncated; C-04, C-05) and the **overall grade** (C-03).
- **PASS/FAIL** for the term (C-05, C-14). On the Mid Term card it is labelled as an academic progress assessment that does not affect promotion.
- On the **Final Term** card, the applicable **end-of-year status: Promoted to {next grade}, Repeats {grade}, or Graduated** (C-06, C-09).

## 2. Final Term report card: wireframe

```text
┌────────────────────────────────────────────────────────────────────────────────────────────┐
│ [LOGO]          DUGSIGA GOL KHATUMO  ·  GOL KHATUMO SCHOOL                                 │
│                 Cinwaan / Address · Tel. / Phone · Email · Reg. No.                        │
│                 "Halku-dhigga dugsiga / School motto"                                      │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│           WARQADDA NATIIJADA ARDAYGA  /  STUDENT REPORT CARD                               │
│           FINAL TERM  ·  Sannad-dugsiyeedka / Academic Year 2026–2027                      │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│ Magaca / Name: Amina Ahmed Ali Hassan         Lambarka / Admission No.: GK-2024-0137       │
│ Fasalka / Grade: Grade 5  ·  Qaybta / Section: A  ·  Roll No.: 12  ·  Jinsi / Sex: F       │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│                      │  FINAL TERM (this term)    │ MID TERM – REFERENCE ONLY              │
│ #  Maaddo / Subject  │ Marks/100 │   %    │ Grade │   – NOT COMBINED                       │
│                      │           │        │       │ Marks/100 │ Grade                      │
│ 1  Somali            │   78.50   │ 78.50  │  B    │   72.00   │  C+                        │
│ 2  English           │   67.25   │ 67.25  │  C    │   60.50   │  D+                        │
│ 3  Arabic            │   85.00   │ 85.00  │  A-   │   80.00   │  B+                        │
│ 4  Mathematics       │   90.25   │ 90.25  │  A-   │   88.50   │  A-                        │
│ 5  Science           │   49.50   │ 49.50  │  F    │   55.00   │  D                         │
│ 6  Islamic Studies   │   88.00   │ 88.00  │  A-   │   91.00   │  A+                        │
│ 7  Social Studies    │ ABSENT–0  │  0.00  │  F    │   64.75   │  D+                        │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│ Wadarta / Total            │ 458.50 / 700                 │ 511.75 / 700                   │
│ Boqolley / Overall %       │ 65.50 %                      │ 73.10 %                        │
│ Darajo / Overall grade     │ C                            │ C+                             │
│ Natiijada / Result         │ PASS                         │ PASS (progress only)           │
│ Kaalinta / Rank            │ Section: 4 of 32             │ (Mid Term ranks are on         │
│                            │ Grade:  11 of 95             │  the Mid Term card)            │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│ NATIIJADA / RESULT:   ██ PASS ██                                                           │
│ GO'AANKA SANADKA / END-OF-YEAR OUTCOME:   Promoted to Grade 6                              │
│ (Decided automatically from the Final Term overall percentage. Mid Term does not count.)   │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│ Joogitaanka / Attendance (Final Term): School days 96 · Present 90 · Late 2 ·              │
│                                        Absent 4 · Excused 0                                │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│ Faallada macallinka fasalka / Class teacher's remark: .............................        │
│ Faallada maamulaha / Principal's remark: ..........................................        │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│ Grading scale: A+ 91–100 · A- 85–90 · B+ 80–84 · B 75–79 · C+ 70–74 · C 65–69 ·            │
│                D+ 60–64 · D 50–59 · F 0–49.  PASS = overall ≥ 50%.                         │
│                ABSENT = 0/100, included in the total. Mid Term PASS/FAIL = progress only.  │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│ Macallinka fasalka /      Maamulaha /             Waalidka /             [SCHOOL           │
│ Class teacher             Principal               Parent/Guardian         STAMP]           │
│ ____________________      ____________________    ____________________                     │
├────────────────────────────────────────────────────────────────────────────────────────────┤
│ Issued: 15/06/2027  ·  Doc. No. RC-2027-000458  ·  Page 1 of 1                             │
└────────────────────────────────────────────────────────────────────────────────────────────┘
```

**Example figures (illustrative student and subjects; the real subject list comes from the curriculum):**

- Final Term total: 78.50 + 67.25 + 85.00 + 90.25 + 49.50 + 88.00 + 0 = **458.50 / 700 = 65.50% → C, PASS**. Science (F) and the ABSENT subject (F) do not cause FAIL, because the overall percentage is ≥ 50 (C-05, C-07).
- Mid Term reference: 511.75 / 700 = 73.107…% → **displayed 73.10%** (truncated) → C+, **PASS (progress assessment only)**. The column is headed **"Reference Only – Not Combined"** and has no effect on the Final Term result or the outcome (C-02, C-06, C-14, Q-E6.3).
- 90.25 → A- (A+ starts at 91), per the exact-value rule (C-04, C-08).

## 3. Mid Term report card: differences from the Final Term card

- Title: "FINAL TERM" is replaced by **"MID TERM"**.
- One results block only (Mid Term marks, %, grade). There is no reference column.
- Rank: the Mid Term section rank and grade rank.
- Result line: **"NATIIJADA / RESULT: PASS"** (or FAIL) with the note **"Academic progress assessment – does not affect promotion"** (C-14). There is **no** end-of-year outcome line.
- Attendance: Mid Term attendance counts.
- Grading legend, remarks, signatures and footer as on the Final Term card.

## 4. Field specification

| Block | Field | Source / rule |
|---|---|---|
| Header | Logo, school name (Somali first, then English), address, phone, email, registration number, motto | School profile (FR-SCH-03) |
| Title | Card type, term label, academic year | Term (C-01) |
| Student | Full four-part name (First, Father's, Grandfather's, Fourth Name), admission number, grade, section, roll number, sex | Student and enrolment (C-12) |
| Results | Subject, marks /100 (2 decimals), percentage, letter grade | `result_snapshot.subject_breakdown` (C-03, C-08) |
| Results | ABSENT subject | Printed as "ABSENT – 0/100", grade F (C-07) |
| Results | EXEMPT subject | Printed as "EXEMPT", excluded from totals (GP-13) |
| Results (Final Term card) | Mid Term marks and grade per subject, Mid Term total, %, grade and PASS/FAIL, in a column headed **"Reference Only – Not Combined"** | Current Mid Term snapshot of the same enrolment (Q-E6.3). Never added to the Final Term figures. |
| Summary | Total obtained / total possible, overall %, overall grade | Snapshot totals (C-05). % truncated to 2 decimals (C-04) |
| Summary | **Section Rank and Grade Rank** ("Section: 4 of 32 · Grade: 11 of 95") | Competition ranking 1, 2, 2, 4 (C-11, Q-E6.4). Omitted for unranked students |
| Result | PASS / FAIL on both cards. Mid Term labelled "academic progress assessment – does not affect promotion" | `result_status` (C-05, C-14) |
| Outcome (Final Term only) | "Promoted to {next grade}", "Repeats {grade}", "Graduated" | `enrollment.outcome` (C-06, C-09) |
| Attendance | School days, present, late, absent, excused for the term | Attendance records within the term (FR-SAT-06). An attendance-percentage line can be added once its formula is agreed. |
| Remarks | Class teacher's remark, principal's remark | `report_card_remark` (FR-RPC-04) |
| Legend | Official grading scale, PASS rule, ABSENT rule | Grading scale version used by the snapshot (GS-6) |
| Signatures | Class teacher, principal, **parent/guardian signature line**; school stamp area | Q-E6.5. Optional scanned signature and stamp images from school settings |
| Footer | Issue date, **document number**, page | Q-E6.6. The QR verification code (FR-PDF-07) is deferred to a future release; the footer leaves space for it. |
| Not printed | Student photo; parent/guardian name and phone | Q-E6.2, Q-E6.9. Kept in the student record (C-13) |

## 5. Special cases

| Case | What the card shows |
|---|---|
| Student in the configured final grade who passes | Outcome "Graduated" (C-09) |
| Student who fails the Final Term | Result FAIL, outcome "Repeats {grade}" |
| Late joiner without a Final Term result (C-10) | **No Final Term report card is issued.** A one-page "Placement notice" in the same style is printed after the School Admin's placement decision (approved with the layout, Q-E6.1). |
| Student who joined after the Mid Term examination (no active enrolment on the first Mid Term paper date) | **Confirmed (C-23):** no Mid Term report card is issued. On the Final Term card the Mid Term reference column shows **—** for each subject mark and **"No Mid Term Result"** in the summary. No artificial zero or FAIL is shown. The Final Term result is calculated normally, and the Final Term alone decides promotion, repetition or graduation. |
| Result corrected after publication | The re-published version is printed. The footer shows "Revised – version N" and the new issue date. |

## 6. Approved design choices (Q-E6, CONFIRMED as C-20)

All nine choices were approved by the project owner as recommended.

| ID | Question | Approved decision | Reason |
|----|----------|-------------------|--------|
| Q-E6.1 | Overall layout and section order, including the late-joiner placement notice | **Approved:** the one-page A4 layout in §2 | Usual report order, fits one A4 page, contains all mandatory content (§1.1) |
| Q-E6.2 | Student photo | **Not included** in the initial release | Photos are not on file for every student; less personal data on printed documents |
| Q-E6.3 | Mid Term column on the Final Term card | **Included**, labelled **"Reference Only – Not Combined"** | Both term results shown separately on the final document, never combined |
| Q-E6.4 | Ranks shown | **Both Section Rank and Grade Rank** | Both are already computed; the grade rank is fairer across sections |
| Q-E6.5 | Parent/guardian signature | **Signature line included** | Confirms the family received the card |
| Q-E6.6 | QR verification | **Deferred to a future release**; the **document number** is printed now | QR needs a public verification page; adding it later needs no layout change |
| Q-E6.7 | Language | **Bilingual, Somali first, English second** | One printout serves every family and receiving school |
| Q-E6.8 | Somali wording | **Current draft accepted provisionally**, subject to review by a Somali language specialist | Labels are text resources; corrections do not change the design |
| Q-E6.9 | Parent/guardian name and phone on the card | **Not printed** | Staff use the guardian contact list (FR-RPT-03); less personal data on a document that may be passed on |

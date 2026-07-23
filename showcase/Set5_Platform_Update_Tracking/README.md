# Platform Update Tracking

The instrumentation behind a legacy-to-new-product migration — a definition of "done," and a scoreboard that measures progress against it.

---

A platform was being rebuilt as a new product with a hard launch date, and the one question that mattered most — how much of the old system have we actually rebuilt, and does it work? — had no single answer. It lived in three disconnected places: a subject-matter expert's screen-by-screen verdicts, a feature catalog, and more than 500 development tickets.

I built one trustworthy, measurable view of migration state. First I cataloged the legacy admin surface exhaustively — every page's links, buttons, and interactive elements, with element counts and a page-complexity ranking — so "done" had a concrete definition to measure against. Then I built a parity tracker that joins the pieces together: feature to ticket to status to test result to screen, across 267 features, 17 functional areas, 123 screens, and 500-plus tickets, with a burndown and a projected completion date. Every parity claim landed on a ticket only after the mapping was signed off — never silently.

The result: leadership got a launch-readiness scoreboard, engineering got a map of where to aim effort, and QA and the subject-matter experts got a shared source of truth showing which features were *proven* working rather than merely claimed. Tribal knowledge became a measurable, self-service artifact.

## What's in this folder

| File | What it is |
|---|---|
| `Admin_UI_Element_Map.html` | The exhaustive legacy-admin inventory — global elements, search, customer, parcel, and billing pages, every interactive element, with per-page counts and a complexity ranking. This defines the migration's surface area. |
| `Legacy_UI_Parity_Dashboard.html` | The parity tracker — features joined to tickets, status, test results, and screens, with burndown and projected finish. The launch-readiness scoreboard. Open in any browser. |

---
*Figures reflect a point-in-time snapshot; client, vendor, and personal names anonymized. Produced with AI-assisted BA/PO workflows. — Nathaniel Genwright*

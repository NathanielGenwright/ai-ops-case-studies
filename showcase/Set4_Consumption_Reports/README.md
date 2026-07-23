# Consumption Reports

A complete requirements-to-delivery lifecycle on a single report — traceable, testable, and carried all the way into a dev-tested build.

---

A municipality needed a district-level meter-consumption summary that matched its old system's format. Producing it by hand meant assembling data across a meter-readings table with more than 20 million rows — slow, error-prone, and repeated every reporting cycle.

I owned the requirements from charter through dev-tested build. The requirements document traces every acceptance criterion back to a business outcome, and specifies the exact data-model join path, the read-method classification logic (estimated, manual, handheld), hard performance constraints (a single aggregate query, no table locks), the database records the report needs to even appear in the menu, and behavioral scenarios carrying real expected totals — down to the customer count and total gallons. It also carries a log of the bugs found and fixed during implementation and a pre-production checklist.

The result: engineering built it with no back-and-forth, QA had concrete numbers to assert against instead of "looks right," operations had a deploy script and checklist, and a recurring manual assembly task was replaced with a report anyone could run.

## What's in this folder

| File | What it is |
|---|---|
| `FRD_Consumption_Reports.md` | The requirements document — join path, read-method classification, user stories with conditions of acceptance, behavioral scenarios, performance and security concerns, implementation reference, and the bugs-fixed log. |
| `Consumption_By_District.html` | Rendered report output — water and sewer district sections with per-district subtotals and a grand total. Open in any browser. |

---
*Sample data throughout; municipality, vendor, and personal names anonymized. Produced with AI-assisted BA/PO workflows. — Nathaniel Genwright*

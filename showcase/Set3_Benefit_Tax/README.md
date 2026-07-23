# Benefit Tax Calculation Engine

A runnable, in-browser engine that reproduces a municipality's water/sewer benefit-tax math to the penny against its own data.

---

A town migrating off a legacy system needed to know whether its benefit-tax calculation could be reproduced on a new platform. The math is deceptively hard: the charged unit is the *maximum* of several competing drivers — road frontage, building units, consumption — unless a resolution field overrides everything. The source data mixes units with raw feet, distinguishes blank from zero in ways a spreadsheet can't natively hold, and carries real data-quality noise. The question was blunt: can this even be reproduced correctly?

I answered it deterministically — by reproducing the town's own already-computed answers rather than hand-waving a plausible-looking mockup. I reverse-engineered the rules from a calculation-rules document and a 2,250-row extract into a documented algorithm (frontage-to-units with its over-500-foot half-rate branch, the building-unit minimum, blank-versus-zero override semantics, parcel aggregation, prior-year rollover, and certification totals), then built a browser engine that runs eleven scenarios covering every rule branch and shows its computed answer against the extract's known values with a pass/fail on each row — fully editable, with a consumption explorer that makes the selection logic visible. I surfaced four data-quality findings rather than hiding them, and logged the open questions.

The result: a runnable proof that reconciles to the client's own numbers, plus an algorithm spec that doubles as a regression oracle — run the engine across all 2,250 rows and assert it matches — with the two traps that would silently break a reimplementation documented before they could.

## What's in this folder

| File | What it is |
|---|---|
| `Benefit_Tax_POC.html` | The working engine — eleven scenarios validated live against the extract, editable inputs, consumption explorer, certification totals, prior-year rollover. Runs in any browser; no server or build. |
| `FRD_Benefit_Tax.md` | The specification — data model and field mapping, the calculation algorithm in pseudocode, worked examples, data-quality findings, and open questions. |

---
*Sample and extract data anonymized; municipality, vendor, and personal names fictitious. Produced with AI-assisted BA/PO workflows. — Nathaniel Genwright*

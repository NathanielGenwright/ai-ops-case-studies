# Payment Gateway Integration

Onboarding a new payment vendor into a utility-billing platform — from business case to a working proof of the reconciliation model.

---

A utility-billing platform needed to add a new payment gateway. The low-effort option — an external redirect to the vendor's page — required no development but would have cost the finance team a full workday of manual reconciliation every cycle and left customers staring at "$0 paid" for 24+ hours after they'd actually paid, driving support calls and duplicate payments.

I made the case for an embedded integration instead, then wrote a platform-agnostic requirements document where every acceptance criterion traces back to a committed business outcome — covering idempotent transaction handling, the convenience-fee split-record model, gateway health monitoring, and automated end-of-day batch reconciliation. Rather than let "automated reconciliation" stay an abstract phrase in a spec, I built two interactive reconciliation dashboards on sandbox data — captured-vs-settled money flow, funding status, and an exceptions queue — so the mechanism could be seen working before a line of production code shipped.

The outcome: buildable, portable requirements (written to survive either a legacy or greenfield platform decision), a business case that retired the manual-reconciliation option on cost, and a proof-of-concept that de-risked the build up front.

## What's in this folder

| File | What it is |
|---|---|
| `PC_Payment_Gateway_Integration.docx` | Project Charter — business justification, the redirect-vs-embedded decision, scope, and phased roadmap (recurring cards → ACH/e-check → POS/IVR). |
| `FRD_Payment_Gateway_Integration.docx` | Functional Requirements Document — API contract, iframe event model, idempotency, convenience-fee split, and health monitoring, with every acceptance criterion traced to a charter outcome. |
| `Reconciliation_Dashboard_A.html` | Working reconciliation view (sandbox data) — money-flow trend, funding status, exceptions. Open in any browser. |
| `Reconciliation_Dashboard_B.html` | Working batch-driven reconciliation view (sandbox data) — same instrument, different settlement shape. |

---
*Sample and sandbox data throughout; client, vendor, and personal names anonymized. Produced with AI-assisted BA/PO workflows. — Nathaniel Genwright*

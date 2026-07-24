# Password Security Remediation

Turning a third-party security audit into a governed, phased, buildable remediation plan.

---

A key client commissioned a third-party audit of the customer portal that surfaced three Critical and three High authentication vulnerabilities — weak password validation, lenient requirements, no anti-automation on login, and account enumeration. Left unaddressed, the same findings would recur across every other client running its own audit, on top of PCI and SOC 2 exposure.

I converted the auditor's raw findings into a defensible, phased remediation — deciding what belonged in Phase 1 versus what to defer, and, critically, *where* enforcement had to live. The charter fixed the policy decisions with rationale (a hybrid-NIST approach, a 12-character minimum, and a 90-day grace period before hard enforcement). The requirements document pinned enforcement to the shared authentication service — the single layer both standard and white-label sign-in flows pass through, so one fix covers the white-label domains without per-client work — and covered all four authenticatable user types across three portal versions, with acceptance criteria written per finding. A working mockup demonstrates the real-time "which rules are met" feedback.

The outcome: a clean retest path on the Critical findings, requirements that already account for the white-label architecture (where enforcing in the wrong service would silently miss half the clients), and platform-wide hardening rather than a one-client patch.

## What's in this folder

| File | What it is |
|---|---|
| `PC_Complex_Passwords.docx` | Project Charter — audit findings, the operational-risk case, Phase 1 vs Phase 2+ scope, and expected outcomes. |
| `FRD_Complex_Passwords.docx` | Functional Requirements Document — enforcement architecture, the cross-domain white-label flow, the four user types, and per-finding acceptance criteria. |
| `Password_Complexity_Mockup.html` | Live mockup of the real-time password-complexity rule feedback. Open in any browser. |

---
*Sample data throughout; client, vendor, and personal names anonymized. Produced with AI-assisted BA/PO workflows. — Nathaniel Genwright*

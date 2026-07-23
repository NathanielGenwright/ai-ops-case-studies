# Interactive Showcase

> Self-contained, browser-runnable artifacts — each proving a single idea. Where the [written case studies](../README.md) explain a *process*, these show the *output*: working proofs, requirement-to-screen mockups, and a calculation engine you can edit and re-run.

Each set below is a small deliverable bundle in the shape I actually ship them — typically a **Project Charter** and a **Functional Requirements Document** (the "why" and the "what"), paired with a **working mockup or proof** (the "see it"). Open the folder for that set's own write-up; open the `.html` files in a browser to run them.

> **Viewing the HTML:** GitHub shows `.html` as source, not a live page. To run one, download the file and open it locally, or prefix its GitHub URL with `https://htmlpreview.github.io/?`. No server or build step is needed either way.

## The sets

| Set | What it demonstrates | Artifacts |
|---|---|---|
| [**1 · Payment Gateway Integration**](Set1_Payment_Gateway/) | A platform-agnostic requirements package for an embedded-vs-redirect payment integration, with two sandbox reconciliation dashboards that prove the model. | Charter · FRD · [Dashboard A](Set1_Payment_Gateway/Reconciliation_Dashboard_A.html) · [Dashboard B](Set1_Payment_Gateway/Reconciliation_Dashboard_B.html) |
| [**2 · Password Security Remediation**](Set2_Password_Security/) | A third-party security audit (Critical + High findings) converted into a governed, phased remediation plan, with enforcement pinned to the shared auth service so one fix covers white-label flows. | Charter · FRD · [Complexity mockup](Set2_Password_Security/Password_Complexity_Mockup.html) |
| [**3 · Benefit Tax Calculation Engine**](Set3_Benefit_Tax/) | An in-browser engine that reproduces a municipality's water/sewer benefit-tax math **to the penny** against its own extract — eleven scenarios validated live, editable inputs, pass/fail per row. A runnable proof, not a plausible-looking mockup. | [POC engine](Set3_Benefit_Tax/Benefit_Tax_POC.html) · [FRD](Set3_Benefit_Tax/FRD_Benefit_Tax.md) |
| [**4 · Consumption Reports**](Set4_Consumption_Reports/) | A full requirements-to-delivery lifecycle on one report — a traceable FRD carried into a dev-tested build, plus the rendered district-consumption output. | [Rendered report](Set4_Consumption_Reports/Consumption_By_District.html) · [FRD](Set4_Consumption_Reports/FRD_Consumption_Reports.md) |
| [**5 · Platform Update Tracking**](Set5_Platform_Update_Tracking/) | The instrumentation behind a legacy-to-new-product migration — an exhaustive admin-surface inventory that defines "done," and a parity tracker that scores progress against it. | [Element map](Set5_Platform_Update_Tracking/Admin_UI_Element_Map.html) · [Parity dashboard](Set5_Platform_Update_Tracking/Legacy_UI_Parity_Dashboard.html) |
| [**Standalone mockups**](Standalone/) | Four smaller artifacts, each proving one idea: a payment-reassurance dashboard, a reconciliation redesign, a reimagined meter-reading workflow, and an enhanced split-bill UI. | [Payment health](Standalone/Payment_Display_Health_Dashboard.html) · [Reconcile UX](Standalone/Reconcile_Payments_UX_Mockup.html) · [Meter reading](Standalone/Meter_Reading_Screen_Mockup.html) · [Split bill](Standalone/Split_Bill_Enhanced_Mockup.html) |

---

*Sanitization: all artifacts use synthetic and sandbox data; client, vendor, municipality, and personal names are fictitious throughout. The approaches, calculations, and requirements are real; the proprietary data is not.*

— Nathaniel Genwright

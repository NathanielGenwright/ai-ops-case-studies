# Rivervale Consumption Reports — Phase 1

## Functional Requirements Document

| Field | Detail |
|---|---|
| **Document ID** | FRD-2026-CONSUM-001 |
| **Version** | 1.0 — Implementation Complete (Dev) |
| **Date** | 2026-03-05 |
| **Implements** | [PC_Rivervale_Consumption.md — Project Charter](./PC_Rivervale_Consumption.md) |
| **Status** | Dev-tested, pending production deployment |

---

## 1. Purpose & Audience

This document translates the Project Charter (BA-2026-CONSUM-001) into buildable, testable requirements for the Engineering and QA teams. It defines **what** the Meter Consumption by District Summary report must do and **how we verify it works**.

**Guiding Principle:** Every acceptance criterion traces to a business outcome, metric, or risk mitigation committed to in the Project Charter.

---

## 2. Reference Architecture Context

**Data Model — Join Path:**

```
meter_readings (20M+ rows total, ~20K for CID 9003)
  → meters (via meter_id)
    → parcels (via parcel_id)
      → custom_field_values (via model_id = parcel.id)
        → custom_fields (label = 'WATER DISTRICT' or 'SEWER DISTRICT')
```

**Key Tables & Columns:**

| Table | Column | Purpose |
|---|---|---|
| `meter_readings` | `end_reading_date` | Read date (GROUP BY DATE) |
| `meter_readings` | `consumption` | decimal(16,6) — summed for total |
| `meter_readings` | `estimated_reading` | boolean — Estimated read method |
| `meter_readings` | `customer_id` | COUNT DISTINCT for customer count |
| `meter_readings` | `company_id` | Company filter |
| `meter_readings` | `meter_auto_read_type_id` | FK to read type codes |
| `meter_auto_read_types` | `code` | 'GEN' = Manual, other = Handheld |
| `custom_fields` | `label` | 'WATER DISTRICT' (id=8801) or 'SEWER DISTRICT' (id=8802) for CID 9003 |
| `custom_field_values` | `value` | District name (e.g., 'WATER', 'S-MAPLE_AVE') |

**Read Method Classification:**

| Method | SQL Condition |
|---|---|
| Estimated | `estimated_reading = 1` |
| Manual | `estimated_reading = 0 AND COALESCE(mart.code, 'GEN') = 'GEN'` |
| Handheld | `estimated_reading = 0 AND mart.code IS NOT NULL AND mart.code != 'GEN'` |

**Key Technical Constraints:**

- meter_readings has 20M+ rows — must use single aggregate SQL, avoid table locks
- Date parameters undergo `munid_strptime_rewrite!` → YAML serialization → ISO format conversion; `parse_date` must handle both MM/DD/YYYY and YYYY-MM-DD
- Rails `Date#to_s` returns MM/DD/YYYY (locale-dependent) — must use `.strftime('%Y-%m-%d')` for SQL interpolation
- Report visibility requires three DB records: `reports`, `reports_companies`, and `report_display_category_id`
- `billing_job_type_id = 24` routes to Sidekiq (not Jasper)

---

## 3. Epics & User Stories

### Epic 1: Meter Consumption by District Summary Report

> **Business Outcome:** Eliminate manual data assembly, match old system report format (PC Section 2.3)
> **Metrics Traceability:** Report availability, generation time < 60 seconds (PC Section 4)

---

#### Story 1.1 — Generate District Consumption PDF

**As a** billing administrator,
**I want** to generate a PDF report showing meter consumption grouped by water and sewer districts,
**so that** I can provide external stakeholders with district-level consumption data without manual assembly.

**Conditions of Acceptance:**

- Report generates a landscape PDF with page numbers
- Header shows company name, report title, date range, print timestamp, "Common Reporting Units: Gallons"
- Data is grouped into "Water Districts" section followed by "Sewer Districts" section
- Within each section, districts are listed alphabetically
- Within each district, rows are ordered by read date
- Each district row shows: District ID, Meter Read Date, Number Customers, Number Meter Reads, Estimated count, Manual count, Handheld count, Total Consumption
- Each district has a subtotal row
- Each type section (Water/Sewer) has a type subtotal row
- Grand total row at bottom sums all districts
- Consumption values are formatted with comma delimiters (e.g., "50,921,035")

**Behavioral Scenarios:**

```
Scenario: Generate report with all districts
  Given I am on the Meter Consumption by District Summary form
  And I set From Date to "06/12/2025" and To Date to "09/11/2025"
  And I leave district filters empty (all districts)
  When I click "Generate Report"
  Then a PDF is generated with Water Districts section (4 districts) and Sewer Districts section (16 districts)
  And the grand total shows 2,951 customers, 2,960 reads, 128,681,761 Gallons

Scenario: No meter readings in date range
  Given I set a date range with no meter readings
  When I click "Generate Report"
  Then a PDF is generated with headers but 0 in all totals

Scenario: Missing date parameters
  Given I leave From Date or To Date blank
  When the job executes
  Then it raises ArgumentError "From date and To date are required"
```

---

#### Story 1.2 — Filter by Water or Sewer Districts

**As a** billing administrator,
**I want** to filter the report by specific water districts and/or sewer districts,
**so that** I can run reports for just water consumption or just sewer consumption.

**Conditions of Acceptance:**

- Form provides "Water Districts" multi-select dropdown (Chosen.js) populated from custom field values
- Form provides "Sewer Districts" multi-select dropdown populated from custom field values
- When water districts are selected but sewer is left empty, only water data appears
- When sewer districts are selected but water is left empty, only sewer data appears
- When both are left empty, all districts are included
- When specific districts are selected in both, only those districts appear
- Placeholder text shows "All Water Districts" / "All Sewer Districts" when unfiltered

**Behavioral Scenarios:**

```
Scenario: Filter to water districts only
  Given I select "WATER" and "MASTER-WATER" in the Water Districts filter
  And I leave Sewer Districts empty
  When I click "Generate Report"
  Then only WATER and MASTER-WATER districts appear in the PDF
  And no Sewer Districts section appears

Scenario: Filter to sewer districts only
  Given I leave Water Districts empty
  And I select "S-MAPLE_AVE" in Sewer Districts
  When I click "Generate Report"
  Then only S-MAPLE_AVE appears in the Sewer Districts section
  And no Water Districts section appears
```

---

#### Story 1.3 — Report Accessible from Custom Reports Menu

**As a** billing administrator,
**I want** the report to appear in the REPORTS > Custom section,
**so that** I can find and run it without needing a direct URL.

**Conditions of Acceptance:**

- Report appears under "Custom" category on the Outputs page (`/dashboard/reports`)
- Report name displays as "Meter Consumption by District Summary"
- Clicking the link navigates to `/reports/rs/meter_consumption_detail_index`
- Report is linked to Rivervale (CID 9003) via `reports_companies`

---

## 4. Cross-Cutting Concerns

### 4.1 Audit and Observability

- Job logs report generation with company ID, date range, and row count
- `tag_output_metadata('Meter_Consumption_Detail', 'PDF')` sets friendly file name on billing_job record
- Failed jobs show red banner with error message in UI

### 4.2 Performance Expectations

| Metric | Requirement | Rationale |
|---|---|---|
| Query execution | < 5 seconds for 3-month range | Single aggregate SQL with indexed joins |
| PDF generation | < 30 seconds total | wkhtmltopdf on pre-rendered HTML |
| Table locking | None | All JOINs on indexed columns; no row-level locks |
| Result cap | LIMIT 500 grouped rows (dev) | Remove or increase for production |

### 4.3 Error Handling Standards

- `ArgumentError` raised if from_date or to_date cannot be parsed
- Empty result set produces valid PDF with 0 totals (no error)
- `parse_date` gracefully handles Date objects, Time/DateTime objects, MM/DD/YYYY strings, and ISO YYYY-MM-DD strings

### 4.4 Security

- Report requires admin authentication (inherits from PrintJob)
- SQL parameters sanitized via `ActiveRecord::Base.connection.quote()`
- No user-supplied strings interpolated directly into SQL

---

## 5. Implementation Reference

### 5.1 Files Created

| File | Purpose |
|---|---|
| `lib/muni_billing/jobs/printing/meter_consumption_detail_job.rb` | Job class — SQL query, data aggregation, PDF generation |
| `app/views/reports/templates/meter_consumption_detail.html.erb` | PDF HTML template — table layout, styling |
| `app/views/reports/rs/meter_consumption_detail_index.html.erb` | Form view — date pickers, district multi-selects |

### 5.2 Files Modified

| File | Change |
|---|---|
| `app/controllers/reports_controller.rb` | Added `"meter_consumption_detail_index"` to `when` clause for district dropdown population |

### 5.3 Database Records Required

```ruby
# 1. Report record
Report.create!(
  name: 'Meter Consumption by District Summary',
  key_value: 'METER_CONSUMPTION_DETAIL',
  index_name: 'rs/meter_consumption_detail_index',
  job_class: 'MetroBill::Jobs::Printing::MeterConsumptionDetailJob',
  billing_job_type_id: 24,
  report_type_id: 5,
  report_version_id: 4,
  report_display_category_id: 1,  # Custom
  show_on_menu: true,
  pdf: true, csv: false, xls: false
)

# 2. Company link
ReportsCompany.create!(report_id: <report_id>, company_id: 9003)
```

### 5.4 Production Deployment Script

Location: `legacy/tmp/insert_meter_consumption_detail_report.rb`

Run on production: `bundle exec rails runner tmp/insert_meter_consumption_detail_report.rb`

### 5.5 Pre-Production Checklist

- [ ] Remove `LIMIT 500` from `build_consumption_sql` or increase to appropriate value
- [ ] Remove `.limit(100)` from `fetch_parcel_custom_fields` in `benefit_tax_base_job.rb` (existing reports)
- [ ] Run insert script on production database
- [ ] Verify report appears in Rivervale's Custom reports menu
- [ ] Test with production date range, verify data matches expectations
- [ ] Confirm with customer that report format meets requirements

---

## 6. Out of Scope — Explicitly Deferred

| Capability | Earliest Phase | Notes |
|---|---|---|
| CSV/Excel export | Phase 2 | Add `csv: true` to report, implement CSV renderer |
| Water loss analysis | Separate project | Requires well meter vs. master meter vs. sold comparison |
| Dual account categories | Separate project | Data model change — each account needs two category fields |
| "Final" read method column | Phase 2 | No data source identified in current schema |
| Dynamic read type columns | Phase 2 | `GROUP_CONCAT(DISTINCT mart.code)` available but unused |

---

## 7. Glossary

| Term | Definition |
|---|---|
| CID | Company ID in MetroBill |
| Custom Field | Company-configurable metadata field on parcels (e.g., WATER DISTRICT) |
| CFV | Custom Field Value — the actual district name stored per parcel |
| Estimated Reading | Meter reading flagged as estimated (not actual) |
| Manual Reading | Non-estimated reading with `meter_auto_read_type.code = 'GEN'` |
| Handheld Reading | Non-estimated reading with any code other than 'GEN' |
| GEN | General read type code — maps to Manual reads |
| munid_strptime_rewrite! | Controller method that converts date strings to Date objects before job serialization |
| RetrofitJobProcessor | Job concern providing 3-hook pattern (prepare, process, finalize) |
| BenefitTaxBaseJob | Shared module with district custom field constants and utility methods |

---

## 8. Bugs Found & Fixed During Implementation

| Bug | Root Cause | Fix |
|---|---|---|
| "From date and To date are required" on every run | `munid_strptime_rewrite!` converts dates to Date objects; YAML serializes as ISO "2025-06-12"; `Date.strptime` expects MM/DD/YYYY | Added `Date.parse(str)` fallback in `parse_date` |
| SQL returns 0 rows despite data existing | Rails `Date#to_s` returns "06/12/2025" (locale format); MySQL can't parse this as a date | Changed to `.strftime('%Y-%m-%d')` for SQL interpolation |
| Report not visible in UI menu | Missing `reports_companies` junction record and null `report_display_category_id` | Added both records; documented in insert script |

---

## 9. Document Control

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-03-05 | Nathaniel Genwright | Initial draft — implementation complete in dev |

---

*This document implements Project Charter **BA-2026-CONSUM-001**. Scope changes that affect business outcomes must be reflected back to the Project Charter and re-approved by stakeholders.*

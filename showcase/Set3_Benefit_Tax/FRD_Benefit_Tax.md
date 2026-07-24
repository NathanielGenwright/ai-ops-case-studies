# FRD — Town of Rivervale Water/Sewer Benefit Tax Engine

| | |
|---|---|
| **Document ID** | FRD-2026-BENTAX-001 |
| **Status** | Proof of Concept (POC) — for feasibility demonstration |
| **Author** | Nathaniel (Product Owner) |
| **Date** | 2026-06-19 |
| **Source rules** | [reference/Calculation_Rules_QA.md](reference/Calculation_Rules_QA.md) (transcribed from `Calculation Rules.pdf`) |
| **Source data** | `2026_BENEFIT_TAX_SUMMARY.csv` — full extract, ~2,250 parcel/account rows, SWIS `990001` (Town of Rivervale) |
| **Sample data** | [reference/demo_parcels.csv](reference/demo_parcels.csv) — 14 curated rows (11 scenarios) |
| **Working POC** | [Benefit_Tax_POC.html](Benefit_Tax_POC.html) — runnable calculation engine + live validation |

---

## 1. Purpose

Demonstrate — with a **runnable engine, not a static mockup** — that MetroBill can reproduce the Town of Rivervale's annual **water/sewer benefit-tax** calculation currently performed in their legacy *HydroWorks* system. The POC proves the calculation rules can be implemented deterministically by **reproducing the town's own already-computed answers** (`new_wbt` / `new_sbt`) from the source extract.

This is a feasibility artifact. It is **not** production-ready code and does not yet integrate with MetroBill's billing/levy pipeline.

## 2. Scope

**In scope (POC):**
- The per-account unit calculation for **Water Benefit Tax (WBT)** and **Sewer Benefit Tax (SBT)**.
- The four calculation drivers: **road frontage**, **building units**, **consumption**, and the **resolution override**.
- Parcel-level **aggregation** for tax maps with multiple accounts (Q8).
- **Prior-year rollover** behavior (Q5) — demonstrated as a one-click action.
- **Certification totals** — total water + total sewer units (Q6).
- A 11-scenario sample chosen to exercise every rule branch.

**Out of scope (POC):** the County levy file format, live meter-reading integration (consumption is read from the precomputed `usage_units` value; §5.3 explains the production path), UI inside the MetroBill app, the Town Code legal text (Q7, pending), and writing back to any database.

## 3. Background — why this is hard

The town is migrating off *HydroWorks*. Two structural mismatches drive most of their questions:

1. **Frontage is stored as FEET in HydroWorks but as derived UNITS in MetroBill.** Account #1000 is 166.00 ft in HydroWorks and shows as `1.66` in MetroBill (166 ÷ 100). The values are *consistent*, but the town reads them in different units, which looks like an error. The over-500-ft rule (Q1) compounds this: Account 0734 is 645.00 ft → `5.73` units, which is not a simple ÷100.
2. **The benefit-tax unit is not a single field — it is the maximum of several competing drivers, unless overridden.** A parcel is charged for the *largest* of its frontage units, building units, or consumption units — but a populated **Resolution** field overrides everything (including forcing zero). This selection logic is the core of the engine.

The source extract also contains **real data-quality inconsistencies** (the same `frontage` column holds units on most rows and raw feet on others). The POC surfaces these rather than hiding them (§9).

## 4. Data model & field mapping

Each row in the extract is one **billing account**. Accounts are grouped by **tax map** (`taxmap`) into **parcels**. The CSV columns map to MetroBill customer custom fields as follows:

| Extract column | Meaning | Proposed MetroBill field | Notes |
|---|---|---|---|
| `custno` | Account number | Customer / account # | Account-level key |
| `swiss_code` | SWIS code `990001` | Company constant | Identifies Town of Rivervale |
| `taxmap` | Tax-map parcel ID | Parcel identifier | **Aggregation key** for Q8 |
| `wtr_dist` | Water district (e.g. `WD001`) | Service flag | **Blank = not in water district → no WBT** |
| `sew_dist` | Sewer district (e.g. `SD002`) | Service flag | **Blank = not in sewer district → no SBT** |
| `frontage` | Road frontage | **Stored as FEET** (recommended) | Today inconsistent (units vs feet) — see §9 |
| `bld_units` | Count of developed building units | Building units | Source of the "1.0 minimum" (§5.4) |
| `usageunits` | Consumption units | Computed value (§5.3) | `Σ(last 4 reads) ÷ 75,000` |
| `resol_wbt` | Water resolution override | Resolution (Water) | **Blank = no override; `0` = force zero** |
| `resol_sbt` | Sewer resolution override | Resolution (Sewer) | Same semantics |
| `prior_wbt` / `prior_sbt` | Prior-year charged units | Prior WBT / SBT | Set by rollover (§6) |
| `new_wbt` / `new_sbt` | **Current-year charged units** | New WBT / SBT | **The output the engine computes** |
| `hw_active` | HydroWorks-active flag | Status | Housekeeping |

> **Critical distinction — blank vs zero in the Resolution field.** Per Q3, a *populated* resolution (including `0`) is a hard override; a *blank* resolution means "no override, use the calculated value." CSV cannot natively distinguish empty-string from zero, so the engine treats `""`/`null` as **no override** and a numeric `0` as **force zero**. This must be preserved in the real schema (nullable column, not `DEFAULT 0`).

## 5. The calculation algorithm

For a given account and a given tax type (`water` or `sewer`), the charged units are:

```
function unitsFor(account, taxType):
    district   = account.district[taxType]      # wtr_dist  | sew_dist
    resolution = account.resolution[taxType]    # resol_wbt | resol_sbt

    # (a) District gate — not in the district means no charge column at all.
    if isBlank(district):
        return N/A          # render blank, exclude from that tax's total

    # (b) Resolution override — a populated value (INCLUDING 0) wins outright.   [Q2, Q3]
    if isPopulated(resolution):
        return round2(resolution)

    # (c) Otherwise charge the LARGEST of the three calculated drivers.
    frontageUnits    = frontageUnitsFromFeet(account.frontageFeet)   # [Q1]
    buildingUnits    = account.bldUnits                              # [Q2 — 1.0 minimum]
    consumptionUnits = account.usageUnits                            # [Q4]

    return round2( max(frontageUnits, buildingUnits, consumptionUnits) )
```

### 5.1 Road frontage → units (Q1)

```
function frontageUnitsFromFeet(feet):
    if isBlank(feet): return 0
    if feet <= 500:   return round2(feet / 100)                  # 1 unit per 100 ft
    else:             return round2(5 + (feet - 500) / 200)      # first 500 ft = 5 units;
                                                                  # remainder at half-rate
```

The over-500 branch encodes the town's "first 500 ft = 5 units, everything thereafter divided in half" rule. Algebraically: `5 + ((feet − 500) ÷ 2) ÷ 100` = `5 + (feet − 500) ÷ 200`.

| Frontage | Branch | Calculation | Units |
|---|---|---|---|
| 166 ft | ≤ 500 | 166 ÷ 100 | **1.66** |
| 270 ft | ≤ 500 | 270 ÷ 100 | **2.70** |
| 645 ft | > 500 | 5 + (645−500)/200 = 5 + 0.725 | **5.73** |
| 800 ft | > 500 | 5 + (800−500)/200 = 5 + 1.5 | **6.50** |

### 5.2 Rounding

All unit values round to **2 decimals, half-up** (0.725 → 0.73, matching the town's worked example). See the floating-point caveat in §11 — naïve `Math.round(0.725*100)/100` returns `0.72` in JavaScript and must be corrected.

### 5.3 Consumption → units (Q4)

`consumptionUnits = Σ(last 4 meter reads) ÷ 75,000`.

In the POC, consumption is read directly from the extract's precomputed `usageunits` value (the town's legacy system already applied this formula). **In production**, the engine sums the last four `meter_readings` within the operator-specified date range and divides by 75,000 — i.e. `usage_units` is *derived, never hand-entered*. This matches the town's explicit statement that they do not enter usage units manually.

### 5.4 Building units & the "1.0 minimum" (Q2)

A developed parcel carries at least one building unit, and `buildingUnits` (typically `1`) is what realizes the town's "minimum charge is 1.0 unit." There is **no separate hard-coded 1.0 floor** in the engine: vacant/no-building accounts (`bld_units = 0`) with no resolution legitimately compute below 1.0 (e.g. a sub-metered child account billed only on its small consumption). This interpretation reproduces the extract exactly; see **Open Question OQ-1 (§10)** for confirmation that a developed parcel can never fall below 1.0 by another path.

### 5.5 Resolution override (Q2, Q3) — the decision that wins

The resolution field is the master switch:
- **Blank** → no override; use `max(frontage, building, consumption)`.
- **Populated with any number** (`0`, `0.5`, `1`, `2.70`, …) → **force that exact value**, ignoring all drivers. This covers both developer resolutions below 1.0 (undeveloped lots at 0.75) and full exemptions (0.00).

## 6. Parcel aggregation for multiple accounts (Q8)

Accounts are computed **independently**, then **summed by tax map** for the County levy:

```
parcelWaterUnits(taxmap) = Σ over accounts on taxmap of unitsFor(account, water)   # N/A treated as 0
parcelSewerUnits(taxmap) = Σ over accounts on taxmap of unitsFor(account, sewer)
```

Example — condo parcel `096.06-01-15.100`: 44 child unit accounts each resolution-forced to `1.0`, plus master/common rows resolution-forced to `0.0`. Parcel rollup = **44.00 water / 44.00 sewer**. (The POC embeds a representative subset of children + one master row and computes the rollup of that subset; the full-parcel figure is noted.)

## 7. Prior-year rollover (Q5)

At year start, before the new calculation runs, current values become prior values:

```
for each account: prior_wbt ← new_wbt ;  prior_sbt ← new_sbt
```

Today this is a manual "Update Prior Benefit Tax Information in WW" step. The POC exposes it as a single **"Roll over to prior year"** button to demonstrate the automation the town requested. The engine keeps `prior` visible alongside `new` so year-over-year movement (e.g. Acct 1508: 61.52 → 86.92) is auditable.

## 8. Certification report & totals (Q6)

The POC produces the two figures the town's certification needs:
- **Total Water Benefit Tax units** = Σ of all current-year WBT across accounts.
- **Total Sewer Benefit Tax units** = Σ of all current-year SBT across accounts.

A production certification report would add the operator's certification block and per-parcel detail; the totals math is what the POC proves.

## 9. Worked examples (the 11 demonstration scenarios)

Each row was hand-validated against the extract's `new_wbt`/`new_sbt`, then re-validated by the POC engine. `feet` is the canonical road-frontage input (reconstructed from the extract — under-500 unit values × 100; true feet where the extract stored feet).

| # | Acct | Scenario | Feet | Bld | Cons | Resol W/S | → WBT | → SBT | Extract | Match |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 45 | Building floor | 70 | 1 | 0.11 | – / – | 1.00 | 1.00 | 1 / 1 | ✅ |
| 2 | 963 | Frontage-driven, water-only | 220 | 1 | 0.23 | – / – | 2.20 | n/a | 2.2 / — | ✅ |
| 3 | 1000 | Feet↔units display (PDF #1000) | 166 | 1 | 0.83 | – / – | 1.66 | n/a | 1.66 / — | ✅ |
| 4 | 734 | Over-500 frontage (PDF 0734) | 645 | 3 | 0 | – / – | 5.73 | 5.73 | 5.73 / 5.73 | ✅ |
| 5 | 1508 | Consumption-driven + YoY jump | 87 | 1 | 86.92 | – / – | 86.92 | 86.92 | 86.92 / 86.92 | ✅ |
| 6 | 1459 | Resolution → specific value | 0 | 1 | 0.17 | 2 / 2 | 2.00 | 2.00 | 2 / 2 | ✅ |
| 7 | 2185 | Developer resolution < 1.0 (Q2) | 0 | 1 | — | 0.5 / 0.5 | 0.50 | 0.50 | 0.5 / 0.5 | ✅ |
| 8 | 897 | Partial exemption (sewer=0) | 446 | 1 | 2.8 | – / 0 | 4.46 | 0.00 | 4.46 / 0 | ✅ |
| 9 | 2042 | Full exemption despite usage | 0 | 0 | 34.89 | 0 / 0 | 0.00 | 0.00 | 0 / 0 | ✅ |
| 10 | 1665 | Data quality: feet in units field | 159.64 | 1 | — | – / – | 1.60 | 1.60 | 1.6 / 1.6 | ✅ |
| 11 | 1922-1924, 243 | Multi-account condo (Q8) | 0 | 1 | — | forced | Σ children | Σ children | 1 each | ✅ |

> "–" = blank resolution (no override). "0" = populated zero (force exempt). "n/a" = not in that district.

## 10. Data-quality findings (surfaced, not hidden)

The full extract contains issues the production import must normalize:

- **DQ-1 — Frontage units vs feet.** The `frontage` column holds *units* on most rows but *raw feet* on others (e.g. Acct 1665 = `159.64`, Acct 0734 dup = `645`, Acct 191 = `343.61`, Acct 721 = `1585.77`, Acct 908 = `2200`). **Recommendation: store frontage in FEET and let the engine derive units.** This single change resolves Q1, the #1000 display question, and the inconsistency in one stroke.
- **DQ-2 — Duplicate / DNU rows.** Rows tagged `DNU` ("Do Not Use", e.g. `0733 DNU`, `1534 DNU`, `0223Z`) and exact taxmap duplicates exist. The aggregation step must dedupe by account, honoring DNU markers.
- **DQ-3 — Sentinel rows.** Rows like `custno 999999` and a row whose `custno` is itself a tax map are extract artifacts (likely footer/diagnostic) and must be filtered.
- **DQ-4 — Misaligned columns.** A few rows have a tax map sitting in the wrong column (e.g. `custno 2258` has the taxmap in the `frontage` position) — import validation must reject/flag these.

## 11. Open questions / confirmations needed

- **OQ-1 (1.0 minimum):** Confirm the floor is realized purely through `building_units` (engine has no separate hard floor). Is there any developed-parcel path where a sub-1.0 result is wrong? *(Q2 wording is ambiguous; data supports the no-hard-floor reading.)*
- **OQ-2 (Town Code, Q7):** The Town Code benefit-tax section was referenced but not received. Needed to confirm the over-500 rule and any tiers beyond 500.
- **OQ-3 (Aggregation semantics):** Confirm the County levy wants **summed units per tax map**, vs. per-account billing. POC assumes sum-per-taxmap.
- **OQ-4 (Rollover trigger):** Should rollover run automatically at year start, or stay a deliberate operator action after the levy file is finalized? (The town's process is iterative.)
- **OQ-5 (Rounding):** Confirm half-up to 2 decimals (matches the 0.725 → 0.73 example).

## 12. How to run the POC

Open [Benefit_Tax_POC.html](Benefit_Tax_POC.html) in any browser (no server, no build). It:
1. Loads the 11 demonstration scenarios.
2. Runs the calculation engine **in the browser** and shows computed WBT/SBT next to the extract's known values, with a ✅/❌ per row and an overall match count.
3. Lets you **edit any input** (frontage feet, building units, consumption, resolution) and recomputes live — try changing Acct 0734's frontage to 800 ft to see 6.50.
4. Shows **certification totals** and a **prior-year rollover** button.
5. Includes a **Consumption Explorer** — pick a parcel, drag consumption (shown in both units and ≈gallons, units × 75,000), and watch New WBT/SBT respond. A status banner makes the `max()` behavior explicit: it reports whether consumption is *driving* the charge or whether the building/frontage **baseline** still wins, and names the breakeven (consumption must exceed the baseline units to have any effect). This is the clearest way to see why consumption changes are often invisible on low-volume parcels.

---

## Engineering Notes
*Engineering reference, not stakeholder-facing.*

- **Floating-point rounding bug.** `Math.round(0.725 * 100) / 100 === 0.72` in JavaScript because `0.725` stores as `0.7249999…`. The engine uses `Math.round(x*100 + 1e-9)/100` (sign-aware) so 0.725 → 0.73 as the town expects. Any production implementation (Ruby `BigDecimal`, SQL `ROUND`) must pick an explicit half-up mode; do not rely on language defaults.
- **Selection vs. sum.** Per-account logic is a `max()` (pick the largest driver); parcel logic is a `Σ` (sum across accounts). Keeping these two operations distinct is the most common place a reimplementation will go wrong.
- **Nullable resolution column.** Do **not** model `resol_wbt/resol_sbt` as `NOT NULL DEFAULT 0` — that would silently exempt every parcel. `0` and `NULL` are semantically different (force-zero vs no-override).
- **Match rate as a regression metric.** The strongest production test is to run this engine over all ~2,250 extract rows and assert it reproduces `new_wbt`/`new_sbt`; residual mismatches isolate either an unmodeled rule (e.g. an over-500 tier from the Town Code) or a DQ row to filter.

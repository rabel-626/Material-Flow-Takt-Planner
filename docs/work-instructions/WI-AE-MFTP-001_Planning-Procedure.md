---
documentId: WI-AE-MFTP-001
title: Material Flow & Takt Planner — Planning Procedure
tool: Material-Flow-Takt-Planner
documentVersion: 1.1
toolVersion: 1.5.1
status: Review Draft
updated: 2026-09-09
---

# Material Flow & Takt Planner — Planning Procedure

## 1. Purpose

This work instruction defines the recommended method for converting a reviewed Cycle Time & Labor Study into a documented **Material Flow & Takt Plan**.

The procedure is intended to maintain traceability between:

```text
measured source study
→ planning assumptions
→ takt/capacity analysis
→ labor analysis
→ material demand
→ staging requirements
→ saved planning revision
```

---

## 2. Scope

Use this procedure for planning activities involving:

- required production rate and takt,
- paced-process capacity,
- machine and belt-rate constraints,
- labor sizing at a target rate,
- material consumption,
- supply-skid frequency,
- line-side staging,
- finished-pallet throughput,
- downstream finished-goods buffer space, and
- repeat-study timing evidence.

This procedure does **not** establish machine safety requirements, warehouse inventory policy, ergonomic standards, fire-code clearances, forklift routes, or final engineered layout approval.

---

## 3. Required Inputs

### 3.1 Primary source

A compatible Cycle Time & Labor Study **Report JSON** is required.

The current planner recognizes these report schema names:

```text
Abel-Engineering.CartonerPowerBIReport
MPG.CartonerPowerBIReport
```

The current source-shape validation expects:

- a populated `Study` array,
- a `Stations` array, and
- valid object-array structure for supported optional tables.

Current schema-17 multi-material/statistical exports are read directly; compatible older reports use fallback logic where available.

---

### 3.2 Recommended source-study readiness

Before planning, verify the source study includes or adequately documents:

- stable Study ID,
- reference unit,
- required/observed throughput basis,
- paced processes,
- process events/reference unit,
- current positions,
- selected timing statistics,
- recurring labor,
- physical material quantities,
- material identity,
- supply pieces/skid,
- timing quality/readiness status, and
- reviewed accuracy/outlier conditions.

> A planner can calculate from incomplete information in some cases, but missing source structure must not be interpreted as a validated zero.

---

## 4. Repository Connection

### 4.1 Preferred environment

Use current desktop **Microsoft Edge** or **Google Chrome**.

### 4.2 Connect

1. Open the planner.
2. Select **CONNECT / RECONNECT**.
3. Select the approved Abel Engineering Study Repository root.
4. Grant read/write permission.

The repository workflow uses:

```text
Source:
01_CYCLE_TIME

Planner output:
03_TAKT_MATERIAL_FLOW
```

If repository access is unavailable, **Legacy File Load** may be used for source analysis, but the normal controlled save workflow requires supported repository access.

---

## 5. Load the Primary Cycle Time Study

1. Select **LOAD CYCLE TIME STUDY**.
2. Search or browse the available verified Cycle Time reports.
3. Open the required source.
4. Confirm displayed:
   - Study ID,
   - plant,
   - line,
   - job/SKU,
   - shift,
   - study date,
   - source app version, and
   - source schema version.

### 5.1 Source change behavior

When a different Study ID is loaded, the planner clears report-linked station overrides and manual-material assumptions that could otherwise be carried into the wrong source.

Review all planner assumptions after any source change.

---

## 6. Define the Required Rate

### 6.1 Direct entry

Enter the required production rate in the displayed reference unit/minute.

The planner calculates:

```text
T_required = 60 / R_required
```

Where:

- `T_required` = required takt, sec/reference unit
- `R_required` = required reference units/min

---

### 6.2 Demand / available-time calculator

Use the Takt Rate Calculator when the required rate should be derived from production demand.

Enter:

- required good-unit demand,
- production days,
- shifts/day,
- scheduled hours/shift,
- planned unavailable minutes/shift,
- first-pass yield, and
- rate application mode.

The planner calculates:

```text
Scheduled min/shift
= Hours/shift × 60
```

```text
Net min/shift
= Scheduled min/shift
  − Planned unavailable min/shift
```

```text
Net min/day
= Net min/shift × Shifts/day
```

```text
Net min/period
= Net min/day × Production days
```

```text
Good demand rate
= Required good units / Net min/period
```

```text
Classic customer takt
= 60 / Good demand rate
```

If yield is applied:

```text
Gross run rate
= Good demand rate / First-pass yield fraction
```

```text
Gross production interval
= 60 / Gross run rate
```

Use **Apply** only after confirming the calculator assumptions.

---

## 7. Set Net Production Hours / Day

Enter the daily hours during which the selected production-rate assumption is intended to apply.

Examples of planned non-production time that may already be removed:

- breaks,
- meetings,
- sanitation,
- preventive maintenance,
- planned changeover,
- setup,
- planned line clearance.

Do not subtract the same planned time twice.

---

## 8. Select Timing Basis

Choose one:

### Current-study mean
Use for the central measured estimate when the primary study is sufficiently representative.

### Current-study P90
Use when a conservative within-run percentile is desired.

### Operator-weighted mean
Use when operator representation is important and valid operator data exist.

### Evidence-weighted best estimate
Use when compatible independent Cycle Time studies have been added and pooled evidence is appropriate.

### Conservative future-setting bound
Use when repeat-study evidence is sufficient and the planning question calls for a conservative future-setting timing basis.

> The conservative basis is not a worst-case guarantee.

---

## 9. Configure Reference Evidence — Optional

Reference studies are used for timing evidence only.

### 9.1 Define comparison scope

Select the intended applicability scope:

- Same process family
- Same product family
- Exact product & placement context

Populate:

- Primary Product Family
- Primary Line Configuration
- Design Efficiency Fallback %

when applicable.

### 9.2 Add studies

Use:

**SELECT REFERENCE STUDIES**

or the legacy report option when necessary.

### 9.3 Review each evidence source

Verify:

- Study ID,
- product/SKU,
- product family,
- line configuration,
- process coverage,
- valid cycles, and
- source inclusion.

### 9.4 Review Process Timing Evidence

For each process, review:

- applied sec/action,
- timing basis,
- evidence grade,
- contributing StudyIDs,
- observation count,
- pooled mean confidence interval,
- future-setting prediction interval when available,
- heterogeneity / Welch diagnostic,
- product/placement context, and
- limitations.

Reference evidence should be rejected or narrowed when process comparability is not credible.

---

## 10. Review Process Capacity

Open **Takt / Capacity**.

For each active paced process:

1. Confirm the process relationship.
2. Confirm current positions.
3. Confirm applied timing basis.
4. Review effective sec/reference unit.
5. Review process capacity.
6. Compare capacity with required rate.
7. Review required positions and additional positions.

The planner uses the controlling constraint between:

- touch-labor capacity, and
- explicit complete-sequence elapsed capacity when available.

A process can be elapsed constrained even when touch labor appears below takt.

---

## 11. Review the Bottleneck and Constrained Maximum

### 11.1 Process bottleneck

The process bottleneck is the lowest positive modeled paced-process capacity.

### 11.2 Primary machine rate

Measured machine-rate checks are preferred when available. Otherwise a configured source-rate signal may be used as a labeled proxy.

### 11.3 Belt / line rate

Measured belt evidence is preferred. A configured speed/pitch calculation can be used as fallback.

### 11.4 Constrained maximum

```text
Constrained max
= min(
    process bottleneck,
    primary machine rate,
    belt / line rate
  )
```

Only available positive constraints participate.

If a required paced-process model is incomplete, the line constrained rate should not be treated as validated.

---

## 12. Review Labor Analysis

Verify:

- Current Labor
- Optimal Labor
- Labor Delta to Optimal
- Labor Content
- Source-Rate PPLH
- Optimal PPLH @ Target
- Labor Load @ Target Rate
- Labor Hours Opportunity / Day

The source-study design efficiency is preferred. The planner fallback efficiency applies only when the source does not provide one.

### 12.1 Support work

Support tasks are recomputed at the target rate according to their frequency model.

- per-reference / consumption work scales with target demand,
- fixed events/hour remain fixed hourly burden.

Do not treat a pooled average labor result as proof that bursty support work is physically feasible.

---

## 13. Choose the Material Supply Rate Basis

Use:

### Takt requirement rate
For material supply sized to planned demand.

### Process bottleneck rate
For material flow sized to the modeled paced-process bottleneck.

Record the selected basis in the planning record and downstream reports.

---

## 14. Verify Station Material Inputs

Open **Material Flow**.

For each included material row, verify:

- station,
- segment,
- material/component name,
- physical quantity/reference unit,
- refill quantity,
- pieces/supply skid,
- calculated flow metrics.

### 14.1 Quantity hierarchy

Treat these fields as separate physical concepts:

```text
Components / Reference Unit
≠ Refill Quantity
≠ Pieces / Supply Skid
```

Do not use refill quantity as a supply-skid quantity unless they are physically the same and this has been independently confirmed.

---

## 15. Add Manual Materials — If Required

Use **ADD MANUAL MATERIAL** only for material demand not represented in the source report.

For each manual row, document:

- material name,
- quantity/reference unit,
- pieces/supply skid,
- inclusion status.

Manual materials are planner assumptions and should be traceable to a drawing, BOM, packaging specification, inventory standard, or other controlled source.

---

## 16. Resolve Shared-Material Skid Conflicts

The planner aggregates materials by normalized material name.

If the same material is represented with multiple positive pieces/skid values, the material is flagged as conflicting.

Do not use that material's:

- skids/hour,
- minutes/skid,
- equivalent skids/day,
- whole skids/day, or
- line-side positions

until the conflict is resolved.

---

## 17. Enter Inbound Coverage

Set the line-side material coverage period.

The calculation is:

```text
Line-side positions per material
= CEILING(
    Skids/hour × Inbound coverage hours
  )
```

Review the result against:

- pallet/skid dimensions,
- material presentation,
- aisle space,
- line access,
- lift-truck access,
- empty-container return,
- ergonomic access,
- fire protection,
- egress, and
- local site standards.

The planner calculates quantity-derived footprints only.

---

## 18. Enter Finished-Pallet Planning Inputs

Enter:

- reference units/finished pallet,
- finished-goods coverage hours.

Review:

- finished units/min,
- pallets/hour,
- minutes/pallet,
- pallets/day,
- finished pallet buffer positions.

If the required rate was applied as a **yield-adjusted gross run rate**, the planner applies the recorded yield to finished-output calculations while material supply continues to follow the selected gross planning flow.

---

## 19. Review Storage & Handling Snapshot

Confirm:

- inbound equivalent skids/day,
- whole incoming supply skids/day,
- inbound line-side positions,
- finished pallets/day,
- finished-pallet buffer positions, and
- total staging positions.

> **Total staging positions** is a simple concurrent footprint count. It does not calculate square footage or travel paths.

---

## 20. Review Data Confidence and Reliability

Before finalizing a high-impact plan:

1. Review the controlling-process evidence plot.
2. Check the pooled mean confidence interval.
3. Check future-setting uncertainty when available.
4. Review evidence grades.
5. Read all displayed limitations.
6. Confirm process relationship and staffing assumptions.
7. Confirm recurring-task assumptions.

Do not use ANOVA p-values as a reliability score.

---

## 21. Export Review Outputs

Available report outputs include:

- Material CSV
- Reliability CSV
- HD / Vector PDF Report

These are reporting outputs.

They are not alternate resumable plan files.

---

## 22. Save the Authoritative Plan

Select **SAVE PLAN**.

A current v1.5.1 save requires:

- loaded Cycle Time report,
- stable source Study ID,
- valid repository connection.

The save creates a new immutable revision, including:

```text
TP-...__REVISION__SOURCE_REPORT.json
TP-...__REVISION__MATERIAL.csv
TP-...__REVISION__RELIABILITY.csv
TP-...__REVISION__MANIFEST.json
TP-...__REVISION__PLAN.json
```

The exact timestamp/random revision token varies by save.

The plan JSON is written as the final commit marker after companion outputs and manifest information have been prepared and verified.

---

## 23. Reopen a Saved Plan

Select **OPEN SAVED PLAN**.

The current workflow:

1. locates a supported plan revision,
2. verifies the package manifest,
3. verifies file sizes and SHA-256 hashes,
4. verifies the source-report snapshot lineage,
5. restores the exact source report,
6. restores planner assumptions and overrides,
7. restores evidence configuration, and
8. recalculates the planner.

Do not manually move or separate files within one immutable revision package.

---

## 24. Reset Planner Settings

**RESET PLANNER SETTINGS** resets planning assumptions, skid inputs, and manual material rows while retaining the loaded source report and reference-study evidence set.

Use this when starting a new planning scenario from the same measured study.

---

## 25. Final Approval Review

At minimum, the reviewer should verify:

- source-study identity and revision,
- required-rate basis,
- net-time basis,
- yield assumption,
- timing basis,
- evidence scope,
- process model completeness,
- bottleneck plausibility,
- labor feasibility,
- material quantities,
- skid quantities,
- material aggregation,
- staging coverage,
- finished-pallet packout,
- physical-layout feasibility,
- report exports, and
- successful repository save.

---

## 26. Related Documents

- **QS-AE-MFTP-001** — Quick Start
- **REF-AE-MFTP-001** — Takt, Capacity & Labor Calculations
- **REF-AE-MFTP-002** — Material Flow & Staging Calculations
- **LIM-AE-MFTP-001** — Known Limitations
- **DATA-AE-MFTP-001** — Data Handling & Repository Model

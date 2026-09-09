---
documentId: QS-AE-MFTP-001
title: Material Flow & Takt Planner — Quick Start
tool: Material-Flow-Takt-Planner
documentVersion: 1.1
toolVersion: 1.5.1
status: Review Draft
updated: 2026-09-09
---

# Material Flow & Takt Planner — Quick Start

## Purpose

The **Abel Engineering Material Flow & Takt Planner** converts a validated Cycle Time & Labor Study **Report JSON** into a downstream engineering planning model for:

- required takt and production-rate comparison,
- process capacity and bottleneck analysis,
- target-rate labor analysis,
- material consumption and incoming supply-skid demand,
- line-side staging requirements,
- finished-pallet throughput and downstream buffer positions,
- timing-evidence and repeat-study reliability review, and
- controlled, resumable planning records in the Abel Engineering Study Repository.

> **Important:** This planner is not a time-study capture tool. The Cycle Time & Labor Study remains authoritative for raw timing observations, process relationships, positions, material definitions, recurring-task definitions, and source-study quality review.

---

## Preferred Data Flow

```text
Cycle Time & Labor Study
        |
        | verified Report JSON
        v
Study Repository / 01_CYCLE_TIME
        |
        | LOAD CYCLE TIME STUDY
        v
Material Flow & Takt Planner
        |
        | SAVE PLAN
        v
Study Repository / 03_TAKT_MATERIAL_FLOW
```

A current repository save creates an immutable planner revision with an exact source-report snapshot, planner state, material CSV, timing/reliability CSV, and package manifest.

---

## Before You Start

Confirm that the Cycle Time source study has been reviewed for:

- correct reference unit,
- stable Study ID,
- correct process-event relationships,
- representative timing observations,
- current-position counts,
- reviewed timing outliers and exclusions,
- recurring-task frequency models,
- physical material quantities,
- material names/components, and
- pieces per incoming supply skid when skid planning is required.

For the repository workflow, use a current desktop version of **Microsoft Edge or Google Chrome** because the Study Repository uses browser folder-access capabilities.

---

## Quick Workflow

### 1. Load the Cycle Time source

From **Overview → Study Repository**:

1. Select **LOAD CYCLE TIME STUDY**.
2. Choose the appropriate verified report from `01_CYCLE_TIME`.
3. Confirm:
   - Study ID,
   - plant,
   - line,
   - job/SKU,
   - study date,
   - source app version, and
   - source schema information.

Use **LEGACY FILE LOAD** only when the compatible Cycle Time **Report JSON** is outside the Study Repository.

> Do not use the editable Cycle Time **Study JSON** as the planner source.

---

### 2. Set the required production rate

You may either enter the required rate directly or use **CALCULATE FROM DEMAND / AVAILABLE TIME**.

The basic takt relationship is:

```text
Required Takt
= 60 / Required Reference Units per Minute
```

Example:

```text
Required rate = 20 units/min

Required takt
= 60 / 20
= 3.000 sec/unit
```

If using the Takt Rate Calculator, enter:

- required good-unit demand,
- production days,
- shifts per day,
- scheduled hours per shift,
- planned unavailable minutes per shift,
- first-pass yield, and
- whether the planner should use the classic good-unit rate or the yield-adjusted gross run rate.

---

### 3. Enter net production hours per day

Use **net planned production time**, not automatically the paid shift duration.

```text
Daily Reference Units
= Rate per Minute × 60 × Net Production Hours per Day
```

If planned breaks, sanitation, meetings, changeovers, or other non-production windows are already removed from the operating plan, the net-hours input should reflect that.

---

### 4. Choose the Takt Timing Basis

The current planner offers:

| Timing basis | Intended use |
| --- | --- |
| **Current-study mean** | Central estimate from the primary source study. |
| **Current-study P90** | More conservative within-run cycle-time basis. |
| **Operator-weighted mean** | Gives each represented operator balanced influence when available. |
| **Evidence-weighted best estimate** | Uses compatible multi-study timing evidence when available. |
| **Conservative future-setting bound** | Uses the future-setting uncertainty model when sufficient independent study evidence exists. |

Reference studies affect capacity only when an evidence-based timing mode is selected.

---

### 5. Choose the Material Supply Rate Basis

The current normal workflow provides two material-flow bases:

| Basis | Meaning |
| --- | --- |
| **Takt requirement rate** | Size material supply and staging to the required production rate. |
| **Process bottleneck rate** | Size flow to the calculated paced-process bottleneck capacity. |

This selection drives:

- pieces/minute,
- pieces/hour,
- daily pieces,
- incoming skids/hour,
- minutes/skid,
- line-side skid positions,
- finished-pallet throughput, and
- finished-pallet buffer positions.

> Use **Takt requirement rate** for demand-driven supply planning. Use **Process bottleneck rate** when evaluating material flow at the current modeled process constraint.

---

### 6. Set staging assumptions

Enter:

- **Inbound Line-Side Coverage (hours)**
- **Finished-Goods Buffer Coverage (hours)**
- **Reference Units / Finished Pallet**

These are floor-planning windows.

They are not warehouse safety-stock calculations.

---

## Review the Overview

### Rate & Daily Maximum Comparison

Compare:

- required rate,
- process bottleneck,
- primary machine rate,
- belt/line rate,
- constrained maximum, and
- daily equivalents.

At a high level:

```text
Constrained Maximum Rate
= minimum positive value of:
    Process Bottleneck Rate
    Primary Machine Rate
    Belt / Line Rate
```

A daily maximum assumes the rate is sustainable for every entered net production minute. Downtime/OEE losses are not automatically added.

---

## Review the Takt / Capacity Tab

The process chart compares controlling effective process time against required takt.

For a process with current staffed positions:

```text
Displayed Effective Time
= max(
    Touch-Labor Workload per Position,
    Explicit Sequence-Elapsed Workload per Position
  )
```

A process above the takt line cannot mathematically sustain the required rate with the current configured positions.

Review:

- timing basis seconds/action,
- events/reference unit,
- current positions,
- effective seconds/reference unit,
- process capacity,
- target load,
- required positions,
- additional positions, and
- status.

> Required positions are mathematical timing results. They do not prove that additional workers can physically fit, safely access the work, or divide the tasks as assumed.

---

## Review Labor Analysis

Key outputs include:

- Current Labor
- Optimal Labor
- Labor Delta to Optimal
- Labor Content
- Source-Rate PPLH
- Optimal PPLH @ Target
- Labor Load @ Target Rate
- Labor Hours Opportunity / Day

Examples:

```text
Optimal PPLH
= Required Units per Hour / Optimal Labor
```

```text
Target Labor Load %
= Labor sec/reference unit
  × Required units/min
  / (Current labor × 60)
  × 100
```

Optimal labor is constraint-aware and uses the source study's design efficiency when available. The planner uses its visible fallback efficiency only when the source does not provide one.

---

## Review Material Flow

### Station Consumption & Supply Skids

Verify each material row:

- source station,
- material name,
- quantity/reference unit,
- imported refill quantity,
- pieces/supply skid,
- pieces/hour,
- daily pieces,
- skids/hour,
- minutes/skid, and
- line-side positions.

The key distinction is:

| Quantity | Meaning |
| --- | --- |
| **Qty / Reference Unit** | Physical consumption per finished reference unit. |
| **Imported Refill Qty** | Typical line-side service/refill quantity. |
| **Pieces / Supply Skid** | Usable incoming pieces on one supply skid. |

**Refill quantity is not supply-skid quantity.**

---

### Material-Level Skid Summary

The planner combines rows with the same material name.

If the same material is assigned conflicting **Pieces / Supply Skid** values, the planner flags the conflict instead of calculating misleading skid demand.

Resolve the conflict before using staging outputs.

---

## Core Material Equations

```text
Pieces / Minute
= Planning Rate / Minute × Qty / Reference Unit
```

```text
Pieces / Hour
= Pieces / Minute × 60
```

```text
Daily Pieces
= Pieces / Hour × Net Production Hours / Day
```

```text
Skids / Hour
= Pieces / Hour / Pieces per Supply Skid
```

```text
Minutes / Skid
= 60 / Skids per Hour
```

```text
Equivalent Skids / Day
= Daily Pieces / Pieces per Supply Skid
```

```text
Whole Skids / Day
= CEILING(Equivalent Skids / Day)
```

```text
Line-Side Skid Positions
= CEILING(Skids / Hour × Inbound Coverage Hours)
```

---

## Review Finished Throughput

When finished-pallet packout is known:

```text
Finished Pallets / Hour
= Finished Reference Units / Minute
  × 60
  / Reference Units per Pallet
```

```text
Minutes / Finished Pallet
= 60 / Finished Pallets per Hour
```

```text
Finished Pallet Buffer Positions
= CEILING(
    Finished Pallets / Hour
    × Finished-Goods Coverage Hours
  )
```

A buffer position means one concurrent physical pallet footprint downstream of the line.

---

## Optional: Add Reference Studies

Under **Data / Export → Reference Studies & Model Evidence**, compatible Cycle Time reports can be added as timing evidence.

The primary study remains authoritative for:

- line configuration,
- material relationships,
- staffing,
- source-rate signals, and
- planner source lineage.

Reference studies contribute timing evidence only after processes are matched.

Use the evidence views to distinguish:

- precision of the pooled average,
- repeat-study reproducibility,
- applicability,
- data integrity,
- timing uncertainty, and
- future-setting uncertainty.

Do not interpret the displayed reliability grades as a probability that the estimate is correct.

---

## Save the Plan

Use **SAVE PLAN**.

The current repository save writes an immutable revision under:

```text
03_TAKT_MATERIAL_FLOW
  / PLANT
  / LINE
  / YYYY
  / YYYY-MM
  / TP-RECORD-ID
```

The revision contains:

- exact Cycle Time source-report snapshot,
- planner-state JSON,
- material summary CSV,
- timing/reliability CSV, and
- package manifest with integrity metadata.

Use **OPEN SAVED PLAN** to restore a verified saved revision.

---

## Final Review Checklist

Before issuing or relying on a plan, confirm:

- [ ] Correct source Study ID and line
- [ ] Correct required-rate source
- [ ] Correct net production hours
- [ ] Appropriate timing basis
- [ ] No incomplete paced-process timing model
- [ ] Current positions are correct
- [ ] Reference-study scope is appropriate if used
- [ ] Material supply basis matches the planning question
- [ ] Qty/reference unit is physical consumption
- [ ] Pieces/skid is verified
- [ ] No unresolved shared-material skid conflict
- [ ] Inbound coverage is intentional
- [ ] Finished pallet packout is current
- [ ] Finished-goods coverage is intentional
- [ ] Labor results have been reviewed for physical feasibility
- [ ] Repository plan saved successfully
- [ ] Material CSV / Reliability CSV / PDF exported if required

---

## Related Documents

- **WI-AE-MFTP-001** — Material Flow & Takt Planning Procedure
- **REF-AE-MFTP-001** — Takt, Capacity & Labor Calculations
- **REF-AE-MFTP-002** — Material Flow & Staging Calculations
- **LIM-AE-MFTP-001** — Known Limitations
- **DATA-AE-MFTP-001** — Data Handling & Repository Model

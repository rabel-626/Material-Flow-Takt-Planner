---
documentId: LIM-AE-MFTP-001
title: Material Flow & Takt Planner — Known Limitations
tool: Material-Flow-Takt-Planner
documentVersion: 1.1
toolVersion: 1.5.1
status: Review Draft
updated: 2026-09-09
---

# Material Flow & Takt Planner — Known Limitations

## Purpose

This document defines the principal engineering and software boundaries of **Material Flow & Takt Planner v1.5.1**.

> The planner is an engineering analysis aid. It does not replace professional engineering judgment, applicable standards, manufacturer requirements, site procedures, safety review, or validation on the physical production system.

---

## 1. Source-Study Dependency

The planner is only as reliable as the imported Cycle Time & Labor Study.

Incorrect source values can directly affect:

- takt comparison,
- process capacity,
- labor sizing,
- material consumption,
- skid demand,
- staging,
- finished-pallet throughput.

Particular attention is required for:

- process-event relationships,
- current positions,
- timing selection,
- recurring-task frequency,
- material quantity,
- supply pieces/skid.

---

## 2. Missing Data Is Not Zero Capacity

A process can lack a usable timing model because of:

- missing cycle timing,
- missing events/reference-unit relationship,
- missing current positions,
- required complete-sequence elapsed timing not available,
- raw rows present but none eligible under current source-selection rules.

The planner surfaces these conditions as incomplete.

Do not interpret a blank or unavailable metric as zero work content.

---

## 3. Required Rate Can Be a Proxy

When a Cycle Time report is loaded, a source rate may populate the required-rate field as a **configured source-rate proxy**.

A configured or observed source rate is not necessarily customer demand.

Verify the required rate against:

- schedule,
- demand,
- net production time,
- yield,
- operating strategy.

---

## 4. Daily Maximum Is Not Expected Output

Daily maximum calculations assume the selected rate is sustained for every entered net production minute.

The planner does not automatically apply:

- OEE,
- downtime loss,
- scrap,
- starvation,
- blocking,
- startup loss,
- minor stops,
- speed loss,
- changeover loss

unless those losses are explicitly represented by the planning assumptions such as net time or yield.

---

## 5. Constrained Maximum Is Only as Complete as Its Constraints

The line constrained maximum uses available positive:

- paced-process bottleneck,
- machine rate,
- belt/line rate.

It is not a complete physics model of every possible line constraint.

Unmodeled constraints may include:

- accumulation,
- controls logic,
- downstream equipment,
- upstream starvation,
- quality inspection,
- conveyor transfer limits,
- palletizer constraints,
- labor sharing,
- material availability.

---

## 6. Machine-Rate Interpretation

Measured machine-rate checks are preferable to a configured rate.

However, even a measured short-duration machine check does not prove sustainable production across a full shift.

The configured fallback rate is explicitly a proxy and should not be represented as actual production attainment.

---

## 7. Belt-Rate Interpretation

A belt-derived rate assumes the configured or measured:

- linear speed,
- pitch distance,
- reference units/pitch

correctly represent the production relationship.

Belt geometry alone may not represent the true line constraint.

---

## 8. Current Positions Are a Mathematical Parallelism Assumption

Capacity calculations use current positions as parallel processing capacity.

That does not automatically prove that:

- work can be evenly divided,
- workers can access the process,
- workers do not interfere,
- travel can be shared,
- skill requirements are interchangeable,
- ergonomic constraints permit the calculated staffing.

---

## 9. Required Positions Are Not an Automatic Rebalance

The planner calculates timing-based position requirements.

It does not automatically redesign:

- standard work,
- task ownership,
- work-zone boundaries,
- walking paths,
- operator handoffs,
- ergonomic sequence,
- automation interactions.

Required-position outputs require physical validation.

---

## 10. Recurring Work Is Averaged

Recurring tasks are normalized into target-rate labor burden.

Averaged labor can hide burst behavior.

For example, a task may be only 10% average load but still require immediate response during a short production window.

The planner does not currently perform queueing or simultaneous-call simulation for support tasks.

---

## 11. Recurring-Task Uncertainty Is Not Fully Propagated

Timing confidence intervals are conditional on the entered/modelled recurring-task durations, frequency relationships, and staffing.

Uncertainty in those recurrence assumptions is not fully propagated into process-capacity intervals.

---

## 12. P90 Is Not a Confidence Bound

P90 is the 90th percentile of a within-run timing population.

It should not be described as:

- 90% confidence,
- a guaranteed capacity,
- future-setting reliability,
- a worst-case bound.

---

## 13. Evidence Pooling Requires Comparability

Reference studies may differ in:

- product,
- line configuration,
- quantity,
- placement pattern,
- material presentation,
- operator population,
- shift,
- batch,
- machine condition,
- standard work.

The planner provides applicability controls, but metadata cannot prove that two studies are truly exchangeable.

---

## 14. Distinct Study IDs Are Assumed Independent

Repeat-study evidence assumes distinct Study IDs can be treated as separate study-level observations.

Shared:

- batches,
- runs,
- operators,
- source observations,
- equipment conditions

may violate this assumption if not represented in metadata.

---

## 15. Small Number of Studies

The multi-study model uses DerSimonian–Laird between-study variance with a modified Hartung–Knapp interval.

With very few studies:

- between-study variance is uncertain,
- prediction intervals can be unstable,
- DL variance may be optimistic.

Interpret small-`k` results conservatively.

---

## 16. Prediction Interval Meaning

The future-setting prediction interval estimates the possible **underlying mean** in a comparable future setting.

It is not:

- an individual-cycle interval,
- a guaranteed next-study mean,
- an observed finite-sample next-study range.

---

## 17. Capacity Intervals Are Marginal

Exported capacity uncertainty ranges are component-wise marginal envelopes.

They are not a simultaneous line-wide 95% confidence region.

Do not claim joint 95% coverage for all processes at once.

---

## 18. Serial Correlation May Be Unmeasured

The planner can adjust effective sample size when session/run metadata support a lag-1 autocorrelation check.

If appropriate grouping identifiers are absent, serial dependence may remain unidentified.

Closely spaced stopwatch cycles should not automatically be treated as statistically independent.

---

## 19. Operator Effects May Dominate

Where between-operator or operator-cluster uncertainty is larger than simple cycle-level uncertainty, the planner uses a conservative safeguard.

Additional cycles from the same limited operator group may not materially improve generalizability.

---

## 20. Welch ANOVA Is Exploratory

The Welch study-mean diagnostic is an exploratory difference screen.

Its p-values:

- are not the reliability grade,
- are not adjusted for multiple process screens,
- do not establish causal differences,
- do not prove practical interchangeability.

---

## 21. Material Quantity Must Represent Physical Consumption

A process timing relationship is not automatically the same as material consumption.

Verify:

```text
Qty / Reference Unit
```

as physical demand.

Incorrect quantity relationships directly scale material-flow results.

---

## 22. Refill Qty Is Not Pieces / Skid

The planner intentionally separates:

- refill quantity,
- supply-skid quantity.

Using refill quantity as incoming skid quantity can materially distort:

- skids/hour,
- minutes/skid,
- daily skids,
- line-side positions.

---

## 23. Material Names Drive Aggregation

Material-level aggregation is name based.

Spelling, punctuation, abbreviations, or naming conventions can create:

- accidental duplicate material streams,
- unintended combinations.

Standardize names before final planning.

---

## 24. Skid Quantity Conflicts

If the same material contains conflicting positive pieces/skid values, the planner withholds skid calculations for that combined material.

The user must resolve the conflict.

The planner does not choose an arbitrary source value.

---

## 25. Clear Skid Inputs Behavior

Planner-side skid overrides and manual-row skid quantities can be set to zero/blank through the clear control.

Because explicit planner overrides take precedence over imported values, a cleared override may continue to mask an imported skid quantity until the override is removed or the source context is refreshed.

Verify imported pieces/skid after using **Clear Skid Inputs**.

---

## 26. Staging Is Quantity-Based, Not Geometry-Based

The planner calculates **position counts**, not physical floor dimensions.

It does not automatically include:

- pallet dimensions,
- aisle width,
- lift-truck turning area,
- machine clearance,
- rack geometry,
- empty return,
- dunnage,
- overhang,
- fire protection,
- egress,
- electrical clearances.

A layout review remains required.

---

## 27. Coverage Is Not Inventory Policy

Inbound and finished-goods coverage hours are simultaneous staging assumptions.

They are not automatically:

- safety stock,
- reorder point,
- warehouse min/max,
- supplier lead-time stock,
- economic order quantity,
- Kanban count.

---

## 28. Whole Skids / Day Is Rounded

Whole incoming skids/day rounds material-equivalent demand upward.

This is useful for logistical planning but can overstate actual consumption if the next day's partially used skid is carried over.

---

## 29. Finished-Pallet Output Can Be Fractional

Pallets/day represents equivalent completed-pallet throughput.

A finite production run can end with a partial pallet.

---

## 30. Yield Handling Is Conditional

Automatic finished-output yield treatment is applied when the required rate is created by the Takt Rate Calculator in **gross** mode.

A manually entered required rate is treated as the rate entered by the user and does not automatically infer yield semantics.

Document whether a manual rate is:

- good-unit rate,
- gross attempt rate,
- another planning rate.

---

## 31. Material Flow Basis Is Not Independent by Material

All active material rows use the selected planner material-supply rate basis.

The current standard workflow does not independently assign a different production-rate basis to each material.

---

## 32. Manual Materials Are Assumptions

Manual rows are not measured source-study data.

They should be supported by:

- BOM,
- specification,
- packaging standard,
- inventory record,
- engineering drawing,
- validated production record.

---

## 33. Browser Autosave Is Not the Authoritative Record

Browser autosave supports recovery.

It is not the controlled engineering record.

Browser state can be affected by:

- browser profile,
- site/origin,
- IndexedDB availability,
- storage limits,
- site-data clearing,
- private browsing,
- device changes.

Use **SAVE PLAN** for the authoritative repository record.

---

## 34. Repository Workflow Requires Supported Desktop Folder Access

Current direct repository save/open requires File System Access API behavior available in supported desktop Chromium browsers.

Other browsers/devices may still support analysis and browser recovery but not the full repository workflow.

---

## 35. Immutable Revision Packages Must Remain Together

Current saved revisions use:

- source snapshot,
- material CSV,
- reliability CSV,
- manifest,
- plan JSON.

Moving, renaming, editing, or deleting one companion file can cause integrity verification to fail.

This is intentional.

---

## 36. PDF Depends on Browser Print

The report uses vector text and SVG where possible, but final PDF behavior still depends on the browser/print driver.

Use the browser's direct **Save as PDF** for best fidelity.

---

## 37. Engineering Use Disclaimer

Abel Engineering tools are provided as engineering analysis and planning aids.

Results depend on source observations, user-entered data, assumptions, classification, and configuration and should be independently reviewed before use for:

- production commitments,
- staffing decisions,
- capital decisions,
- equipment design,
- safety decisions,
- material-storage design,
- regulatory decisions,
- financial decisions.

These tools do not replace professional engineering judgment, applicable standards, manufacturer requirements, site procedures, or required safety reviews.

Users are responsible for verifying calculations, inputs, outputs, and suitability for the intended application.

The application is not intended to function as a machine safety system, safety PLC, protective device, or real-time process-control system.

---

## Related Documents

- **QS-AE-MFTP-001** — Quick Start
- **WI-AE-MFTP-001** — Planning Procedure
- **REF-AE-MFTP-001** — Takt, Capacity & Labor Calculations
- **REF-AE-MFTP-002** — Material Flow & Staging Calculations
- **DATA-AE-MFTP-001** — Data Handling & Repository Model

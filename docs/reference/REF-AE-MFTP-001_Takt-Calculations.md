---
documentId: REF-AE-MFTP-001
title: Material Flow & Takt Planner — Takt, Capacity & Labor Calculations
tool: Material-Flow-Takt-Planner
documentVersion: 1.1
toolVersion: 1.5.1
status: Review Draft
updated: 2026-09-09
---

# Takt, Capacity & Labor Calculations

## 1. Purpose

This reference defines the principal rate, takt, process-capacity, labor, and timing-evidence calculations used by **Material Flow & Takt Planner v1.5.1**.

The planner uses the source Cycle Time & Labor Study as the measured engineering basis and applies user-entered planning assumptions downstream.

---

## 2. Core Terms

| Term | Definition |
| --- | --- |
| **Reference unit** | The finished unit used to normalize rate, labor, and material relationships. |
| **Required rate** | Planned demand rate in reference units/min. |
| **Takt** | Maximum average time available per required reference unit. |
| **Process event** | One occurrence of a paced process action. |
| **Events/reference unit** | Normalized process frequency relative to the finished reference unit. |
| **Current positions** | Parallel staffed positions configured for a paced process. |
| **Touch labor** | Manual labor content required to perform the process. |
| **Sequence elapsed** | Total parent elapsed cycle time when a complete sequence is explicitly measured. |
| **Process capacity** | Maximum modeled reference-unit rate for one paced process. |
| **Process bottleneck** | Lowest modeled capacity among active paced processes. |
| **Constrained maximum** | Lowest positive available process, machine, or belt rate. |

---

# Part A — Demand and Takt

## 3. Direct Required Rate

When a rate is directly entered:

```text
R_req = required reference units/min
```

Then:

```text
T_takt = 60 / R_req
```

Where:

- `T_takt` = sec/reference unit
- `R_req` = reference units/min

---

## 4. Daily Required Output

```text
Q_day
= R_req × 60 × H_net
```

Where:

- `Q_day` = required reference units/day
- `H_net` = net production hours/day

This is a planning conversion, not an attainment forecast.

---

## 5. Takt Rate Calculator

Let:

- `H_shift` = scheduled hours/shift
- `U_shift` = planned unavailable minutes/shift
- `S_day` = shifts/day
- `D_prod` = production days in planning period
- `Q_good` = required good units in planning period
- `Y` = first-pass yield fraction, 0 < Y ≤ 1

### 5.1 Scheduled minutes per shift

```text
M_sched_shift
= H_shift × 60
```

### 5.2 Net minutes per shift

```text
M_net_shift
= M_sched_shift − U_shift
```

### 5.3 Net minutes per day

```text
M_net_day
= M_net_shift × S_day
```

The current calculator rejects a combination that exceeds 1,440 net scheduled minutes/day.

### 5.4 Net minutes in the planning period

```text
M_net_period
= M_net_day × D_prod
```

### 5.5 Good-unit demand rate

```text
R_good
= Q_good / M_net_period
```

### 5.6 Classic customer takt

```text
T_classic
= 60 / R_good
```

Equivalently:

```text
T_classic
= M_net_period × 60 / Q_good
```

### 5.7 Yield-adjusted gross run rate

```text
R_gross
= R_good / Y
```

### 5.8 Yield-adjusted gross interval

```text
T_gross
= 60 / R_gross
```

Yield does **not** change classic customer takt. It changes the gross process rate needed to produce the required good output.

---

## 6. Yield Treatment in Downstream Flow

When the calculator's **gross** mode is applied:

- material supply follows the selected gross planning rate,
- finished-output calculations apply the recorded yield.

Conceptually:

```text
Gross material-flow rate
= R_gross
```

```text
Expected finished good-unit rate
= R_gross × Y
```

This distinction prevents raw material consumption from being understated when first-pass yield is below 100%.

---

# Part B — Timing Basis

## 7. Current-Study Mean

The source timing mean is the central average of the selected source timing population.

When raw eligible timing rows exist, the current planner treats those raw observations as authoritative rather than silently substituting an incompatible exported summary.

---

## 8. P90

P90 is the 90th percentile of the selected within-study cycle-time population.

It is a **within-run percentile**.

It is not:

- a confidence limit,
- a prediction interval,
- a guaranteed worst case, or
- a service-level probability.

---

## 9. Operator-Weighted Mean

When operator identity is available, the planner can use a timing basis that balances operator representation rather than allowing an operator with more captured samples to dominate the overall mean.

Conceptually:

```text
Operator-weighted mean
= average of represented operator means
```

The exact source or raw operator structure available determines whether this basis can be calculated.

---

## 10. Evidence-Weighted Best Estimate

When compatible reference studies exist, the planner can construct a study-aware pooled estimate.

The primary study remains authoritative for:

- physical process structure,
- positions,
- process relationships,
- materials,
- source-rate signals.

Reference studies contribute timing evidence only.

---

## 11. Conservative Future-Setting Bound

The conservative basis uses the future-setting uncertainty model when it is estimable.

It is intended to represent a more conservative process timing basis for a **comparable future setting**.

It is not a deterministic upper bound and should not be described as a guaranteed worst case.

---

# Part C — Process Workload

## 12. Core Process Labor

Let:

- `t` = selected sec/process event
- `e` = process events/reference unit

Then the core touch-labor burden is:

```text
L_core
= t × e
```

Units:

```text
sec labor / reference unit
```

---

## 13. Recurring Labor

The planner distinguishes recurring work that scales with production from fixed hourly work.

Let:

- `L_stable` = recurring sec/reference unit for per-reference or consumption-driven tasks
- `H_fixed` = recurring labor sec/hour for fixed hourly tasks
- `R_target` = required reference units/min

The fixed hourly burden may be represented per reference unit at the target rate as:

```text
L_fixed_equiv
= H_fixed / (R_target × 60)
```

Therefore:

```text
L_recurring
= L_stable + L_fixed_equiv
```

when a positive target rate exists.

---

## 14. Total Touch-Labor Workload

```text
L_total
= t × e + L_recurring
```

For some compatible source configurations, a direct source labor-content value may be used when appropriate instead of reconstructing it from detailed timing.

---

# Part D — Process Capacity

## 15. Current Positions

Let:

```text
n = current staffed positions
```

A process with no current staffed position is not treated as a normal positive-capacity process.

---

## 16. Touch-Labor Capacity with Fixed Hourly Work

For detailed recurring-task modeling:

```text
Available worker sec/min
= 60n − H_fixed / 60
```

Let:

```text
L_stable_total
= t × e + L_stable
```

Then:

```text
R_labor
= (60n − H_fixed / 60)
  / L_stable_total
```

Where:

- `R_labor` = reference units/min

This formulation keeps fixed events/hour fixed instead of incorrectly scaling them as a constant per-unit burden at all rates.

---

## 17. Sequence-Elapsed Capacity

When explicit complete-sequence parent elapsed timing exists:

Let:

- `t_elapsed` = applied elapsed sec/process event
- `e` = events/reference unit
- `n` = positions

Then:

```text
R_elapsed
= 60n / (t_elapsed × e)
```

Sequence elapsed and touch labor are separate constraints.

---

## 18. Overall Process Capacity

When both labor and elapsed constraints are valid:

```text
R_process
= min(R_labor, R_elapsed)
```

If only the labor constraint is applicable:

```text
R_process
= R_labor
```

The planner deliberately does not substitute touch-labor sum for missing explicit sequence elapsed when a complete-sequence process requires elapsed timing.

---

## 19. Effective Seconds / Reference Unit

The takt chart displays the controlling per-position equivalent:

```text
Effective sec/reference unit
= max(
    Touch-labor workload / positions,
    Sequence elapsed workload / positions
  )
```

This is the time-equivalent form of the capacity constraint.

---

## 20. Takt Compliance

A process is mathematically capable of meeting required takt when:

```text
R_process ≥ R_req
```

Equivalent time view:

```text
Effective sec/reference unit ≤ T_takt
```

---

## 21. Required Positions — Labor Constraint

At target rate `R_req`:

```text
Required labor positions
= CEILING(
    [R_req × L_stable_total + H_fixed / 60]
    / 60
  )
```

This is the detailed recurring-task form.

---

## 22. Required Positions — Sequence Constraint

```text
Required elapsed positions
= CEILING(
    t_elapsed × e / T_takt
  )
```

---

## 23. Overall Required Positions

```text
Required positions
= max(
    Required labor positions,
    Required elapsed positions
  )
```

The planner's required-position result is an arithmetic timing requirement, not an automatic work-rebalance design.

---

# Part E — Line-Level Rate Signals

## 24. Process Bottleneck

```text
R_bottleneck
= min(R_process_i)
```

across active paced processes with usable modeled capacity.

If a required process model is incomplete, the planner surfaces that limitation instead of treating missing timing as infinite capacity.

---

## 25. Primary Machine Rate

When valid machine-rate check trials exist, the current planner prefers the pooled physical measurement:

```text
R_machine
= Total measured units × 60
  / Total measured seconds
```

An approximate 95% t-based interval is also calculated from trial-rate dispersion when multiple trials exist.

If measured checks are unavailable, a configured source-rate field may be used as a labeled proxy.

A configured proxy is not independent observed attainment.

---

## 26. Belt / Line Rate

Measured belt checks are preferred when available.

For configured speed/pitch fallback:

```text
Belt reference rate
= Linear belt speed
  / Pitch distance
  × Reference units per pitch
```

after unit conversion.

---

## 27. Constrained Maximum

```text
R_constrained
= min(
    positive R_bottleneck,
    positive R_machine,
    positive R_belt
  )
```

If the paced-process model is incomplete, the constrained rate is not treated as validated.

---

# Part F — Labor KPIs

## 28. Current Labor

The planner prefers the best available source headcount signal, using source total line headcount where provided, otherwise configured source-study labor/positions.

---

## 29. Design Efficiency

Let:

```text
η = design efficiency fraction
```

The planner prefers the source study's design efficiency. If unavailable, it uses the visible planner fallback.

---

## 30. Constraint-Aware Optimal Labor

For each paced role:

```text
N_labor
= CEILING(
    Labor sec/reference unit
    / (T_takt × η)
  )
```

Sequence elapsed can impose a larger position requirement.

The role requirement is therefore:

```text
N_role
= max(N_labor, N_elapsed)
```

Shared support pools are modeled separately and then added to paced-role requirements.

---

## 31. Theoretical Minimum Labor

When labor content and takt are available:

```text
N_theoretical
= Total labor sec/reference unit
  / T_takt
```

No per-role integer rounding is applied to the theoretical minimum.

---

## 32. Source-Rate PPLH

```text
PPLH_source
= Source rate/min × 60
  / Current labor
```

If the source rate is only a configured proxy, this KPI is not an attainment/OEE metric.

---

## 33. Optimal PPLH at Target

```text
PPLH_optimal
= Required rate/min × 60
  / Optimal labor
```

---

## 34. Target Labor Load

```text
Labor Load %
= Labor sec/reference unit
  × Required units/min
  / (Current labor × 60)
  × 100
```

---

## 35. Labor Hours Opportunity / Day

When current labor exceeds modeled optimal labor:

```text
Labor hours opportunity/day
= (Current labor − Optimal labor)
  × Net production hours/day
```

This is an arithmetic opportunity indicator, not an automatic staffing recommendation.

---

# Part G — Timing Evidence & Reliability

## 36. Raw Timing Authority

When usable raw timing observations are present for a process, v1.5.1 recalculates statistics from the selected raw population and can flag mismatches against exported summary values.

The raw population is filtered using the source study's timing-condition, sequence-revision, completeness, exclusion, and accuracy lineage.

---

## 37. Effective Sample Size for Serial Correlation

Where session/run grouping can be identified, positive lag-1 serial correlation is used to reduce the nominal sample size.

For a group of `n` observations and positive lag-1 correlation `ρ`:

```text
n_eff
≈ n × (1 − ρ) / (1 + ρ)
```

The implementation clamps the adjusted value to a valid range.

If serial correlation cannot be checked because grouping metadata are missing, the planner reports that limitation.

---

## 38. Within-Study Mean Uncertainty

The planner considers multiple uncertainty components when available:

- cycle-level variance,
- between-operator variance,
- operator-cluster variance.

For a component with mean-variance estimate `V` and degrees of freedom `ν`:

```text
95% half-width
= t_0.975,ν × sqrt(V)
```

The planner uses the most conservative available component for the displayed study mean interval.

---

## 39. Multi-Study Pooled Timing

For multiple compatible Study IDs, the current evidence model uses:

- study-level means,
- within-study uncertainty,
- DerSimonian–Laird between-study variance, and
- a modified Hartung–Knapp interval for the pooled mean.

Conceptually:

```text
w_i*
= 1 / (V_i + τ²)
```

```text
Pooled mean
= Σ(w_i* × mean_i) / Σ(w_i*)
```

Where:

- `V_i` = within-study mean variance
- `τ²` = estimated between-study variance

The precise small-sample interval treatment is handled by the application.

---

## 40. Future-Setting Prediction Interval

When enough distinct compatible studies exist, the planner can estimate an approximate range for the **underlying process mean in a comparable future setting**.

Conceptually:

```text
Future-setting PI
≈ pooled mean
  ± critical value
  × sqrt(
      pooled-mean uncertainty
      + between-study variance
    )
```

This interval does **not** describe:

- individual future cycles,
- a guaranteed next-study average,
- a finite-sample observation range, or
- simultaneous line-wide coverage.

---

## 41. Welch Diagnostic

The planner retains a Welch study-mean difference screen as an exploratory diagnostic.

Its p-value is:

- unadjusted across multiple process screens,
- not the reliability grade,
- not proof of interchangeability, and
- not proof that studies are practically different.

---

## 42. Reliability Interpretation

Reliability is presented as separate dimensions such as:

- precision,
- replication,
- integrity,
- reproducibility,
- applicability.

Do not collapse these into “X% reliable.”

---

## 43. Capacity Uncertainty

Capacity intervals are derived from timing uncertainty while holding other planning inputs fixed.

They are conditional on:

- process relationship,
- positions,
- recurring-task durations/frequencies,
- source structure, and
- available elapsed timing.

The exported capacity envelopes are **component-wise marginal bounds**, not joint or simultaneous 95% intervals for the complete line.

---

# Part H — Worked Example

Assume one process has:

```text
Required rate             = 18 units/min
Selected touch time       = 4.20 sec/event
Events/reference unit     = 1.00
Stable recurring labor    = 0.30 sec/unit
Fixed recurring burden    = 180 sec/hour
Current positions         = 2
Sequence elapsed          = 5.00 sec/event
Design efficiency         = 85%
```

### Required takt

```text
T_takt
= 60 / 18
= 3.333 sec/unit
```

### Stable labor burden

```text
L_stable_total
= 4.20 × 1.00 + 0.30
= 4.50 sec/unit
```

### Available labor sec/min

```text
Available
= 2 × 60 − 180/60
= 120 − 3
= 117 sec/min
```

### Labor capacity

```text
R_labor
= 117 / 4.50
= 26.0 units/min
```

### Sequence capacity

```text
R_elapsed
= 60 × 2 / (5.00 × 1.00)
= 24.0 units/min
```

### Process capacity

```text
R_process
= min(26.0, 24.0)
= 24.0 units/min
```

The process can mathematically support 18 units/min with two positions.

### Required labor positions

```text
N_labor
= CEILING(
    [18 × 4.50 + 180/60] / 60
  )

= CEILING(84 / 60)
= 2
```

### Required elapsed positions

```text
N_elapsed
= CEILING(5.00 / 3.333)
= 2
```

### Overall required positions

```text
N_required
= max(2, 2)
= 2
```

This does not prove the work can actually be divided between two operators without interference; that remains an engineering validation task.

---

## 44. Related Documents

- **QS-AE-MFTP-001** — Quick Start
- **WI-AE-MFTP-001** — Planning Procedure
- **REF-AE-MFTP-002** — Material Flow & Staging Calculations
- **LIM-AE-MFTP-001** — Known Limitations
- **DATA-AE-MFTP-001** — Data Handling & Repository Model

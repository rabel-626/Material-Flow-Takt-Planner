---
documentId: REF-AE-MFTP-002
title: Material Flow & Takt Planner — Material Flow & Staging Calculations
tool: Material-Flow-Takt-Planner
documentVersion: 1.1
toolVersion: 1.5.1
status: Review Draft
updated: 2026-09-09
---

# Material Flow & Staging Calculations

## 1. Purpose

This reference defines how Material Flow & Takt Planner v1.5.1 converts source-study physical material relationships into:

- pieces/minute,
- pieces/hour,
- daily pieces,
- incoming skids/hour,
- minutes/skid,
- equivalent skids/day,
- whole skids/day,
- line-side skid positions,
- finished pallets/hour,
- minutes/finished pallet,
- finished pallets/day,
- finished-pallet buffer positions, and
- total staging positions.

---

## 2. Material Supply Rate Basis

The current normal workflow allows:

### Takt requirement rate

```text
R_flow = R_required
```

Use when material supply should satisfy planned demand.

### Process bottleneck rate

```text
R_flow = R_bottleneck
```

Use when evaluating material flow at the modeled paced-process constraint.

A legacy custom-rate field may exist for compatibility with older planner state, but the current visible workflow is centered on **Takt requirement** or **Process bottleneck**.

---

## 3. Physical Material Quantity

For each source station-material relationship, the planner seeks physical material demand per finished reference unit.

The preferred current source field is:

```text
ComponentsPerReferenceUnit
```

If needed, it can derive:

```text
Qty / Reference Unit
= Events / Reference Unit
  × Components / Process Event
```

This is the physical quantity consumed by production.

---

## 4. One-to-Many Materials

Current Cycle Time reports can provide multiple material rows for one process/station.

The planner retains separate station-material relationships before aggregating by material name.

This prevents one process with several components from being collapsed into one generic material quantity.

---

## 5. Refill Quantity Is Not Supply-Skid Quantity

These quantities must remain distinct:

| Field | Physical meaning |
| --- | --- |
| **Qty / Reference Unit** | Consumption per finished unit. |
| **Imported Refill Qty** | Quantity typically replenished during one line-side service event. |
| **Pieces / Supply Skid** | Total usable pieces on one incoming supply skid. |

A line-side refill may use only part of a supply skid.

Therefore:

```text
Refill Qty
≠ Pieces / Supply Skid
```

unless independently verified to be the same physical quantity.

---

## 6. Planner Overrides

The planner can override:

- material name,
- physical quantity/reference unit,
- pieces/supply skid,
- inclusion status.

Overrides are planning assumptions and should be traceable.

Imported source values remain part of the source report; the planner does not modify the original Cycle Time file.

---

## 7. Manual Material Rows

Manual rows are appropriate for items not represented in the source study.

A manual row can define:

- material,
- station/description,
- quantity/reference unit,
- pieces/supply skid,
- inclusion.

Manual material demand should be supported by controlled engineering or production documentation.

---

# Part A — Consumption

## 8. Pieces / Minute

Let:

- `R_flow` = selected material-supply rate, reference units/min
- `q` = material quantity/reference unit

Then:

```text
Pieces / Minute
= R_flow × q
```

---

## 9. Pieces / Hour

```text
Pieces / Hour
= R_flow × q × 60
```

---

## 10. Daily Pieces

Let:

- `H_net` = net production hours/day

```text
Daily Pieces
= R_flow × q × 60 × H_net
```

---

# Part B — Supply Skids

## 11. Skids / Hour

Let:

- `S` = usable pieces/supply skid

Then:

```text
Skids / Hour
= Pieces / Hour / S
```

provided `S > 0`.

---

## 12. Minutes / Skid

```text
Minutes / Skid
= 60 / Skids per Hour
```

This is the average production time represented by one supply skid at the selected flow rate.

It is not necessarily the forklift delivery interval if:

- multiple skids are delivered at once,
- milk-run batching is used,
- decanting occurs,
- material is staged in a supermarket,
- partial skids are exchanged.

---

## 13. Equivalent Skids / Day

```text
Equivalent Skids / Day
= Daily Pieces / Pieces per Supply Skid
```

This value can be fractional.

It represents material-equivalent consumption.

---

## 14. Whole Skids / Day

```text
Whole Skids / Day
= CEILING(Equivalent Skids / Day)
```

The planner rounds individual material supply requirements to a whole-skid planning quantity.

A daily whole-skid quantity is not the same as simultaneous line-side staging.

---

# Part C — Line-Side Staging

## 15. Line-Side Positions per Material

Let:

- `C_in` = inbound coverage hours

```text
Positions_material
= CEILING(
    Skids / Hour × C_in
  )
```

This represents the number of simultaneous supply-skid footprints required to hold the selected coverage.

---

## 16. Total Inbound Positions

```text
Total Inbound Positions
= Σ Positions_material
```

The planner sums the rounded material-level footprint requirements.

---

## 17. Why Rounding Occurs Per Material

Suppose:

```text
Material A = 0.6 positions
Material B = 0.6 positions
```

The planner requires:

```text
A = CEILING(0.6) = 1
B = CEILING(0.6) = 1

Total = 2 positions
```

not:

```text
CEILING(0.6 + 0.6)
= 2
```

In this example the result is the same, but in general material-level rounding preserves the physical need for a discrete footprint for each separately staged material stream.

---

## 18. Coverage Is a Floor-Planning Window

Inbound coverage answers:

> “At this consumption rate, how many supply-skid footprints are needed simultaneously if the line should hold approximately this many hours of material?”

It does **not** answer:

- reorder point,
- safety stock,
- days-on-hand,
- warehouse inventory requirement,
- supplier lead-time inventory,
- emergency buffer,
- Kanban card count without additional assumptions.

---

# Part D — Material Aggregation

## 19. Material-Level Summary

Station rows are grouped by material name using case-insensitive matching.

For one material used by several stations:

```text
Total Qty / Reference Unit
= Σ Station Qty / Reference Unit
```

```text
Total Pieces / Hour
= Σ Station Pieces / Hour
```

```text
Total Daily Pieces
= Σ Station Daily Pieces
```

This allows material handling to see one combined supply stream.

---

## 20. Material Naming Control

Because aggregation is name based, inconsistent naming can create artificial separation.

For example:

```text
"Foil Pack"
"FOIL PACK"
```

will combine.

But:

```text
"Foil Pack"
"Foil Packs"
```

may remain separate.

Standardize material names before issuing the plan.

---

## 21. Conflicting Pieces / Skid

If grouped rows for the same material contain different positive pieces/skid values, the planner flags a **skid quantity conflict**.

The current model then withholds downstream skid calculations for that material rather than choosing one value silently.

Affected outputs include:

- skids/hour,
- minutes/skid,
- equivalent skids/day,
- whole skids/day,
- line-side positions.

Resolve the source or planner override before use.

---

# Part E — Finished Goods

## 22. Finished-Output Rate

Normally:

```text
R_finished = R_flow
```

When the planner required rate came from the **yield-adjusted gross** takt calculator mode:

```text
R_finished
= R_flow × Yield
```

while material supply continues to use the selected gross flow basis.

This preserves the distinction between gross process attempts and expected good finished output.

---

## 23. Finished Pallets / Hour

Let:

- `P` = reference units/finished pallet

```text
Pallets / Hour
= R_finished × 60 / P
```

provided `P > 0`.

---

## 24. Minutes / Finished Pallet

```text
Minutes / Pallet
= 60 / Pallets per Hour
```

---

## 25. Finished Pallets / Day

```text
Finished Pallets / Day
= Pallets / Hour × H_net
```

This is an equivalent output quantity; physical operations may create partial final pallets at the end of a run.

---

## 26. Finished Pallet Buffer Positions

Let:

- `C_out` = finished-goods coverage hours

```text
Finished Buffer Positions
= CEILING(
    Pallets / Hour × C_out
  )
```

A finished buffer position means one downstream pallet footprint reserved for completed product awaiting movement, QA, stretch-wrap, labeling, or warehouse transfer.

---

## 27. Total Staging Positions

```text
Total Staging Positions
= Total Inbound Positions
  + Finished Buffer Positions
```

This is a footprint count only.

It does not convert positions into square feet.

---

# Part F — Worked Example

Assume:

```text
Required flow rate       = 18 cartons/min
Net production time      = 8 hr/day
Inbound coverage         = 1.25 hr
Finished coverage        = 1.00 hr
Finished pallet          = 720 cartons
```

Material A:

```text
Qty/carton               = 6 pieces
Pieces/supply skid       = 4,000
```

Material B:

```text
Qty/carton               = 1 piece
Pieces/supply skid       = 1,200
```

---

## 28. Material A

### Pieces/min

```text
18 × 6
= 108 pieces/min
```

### Pieces/hr

```text
108 × 60
= 6,480 pieces/hr
```

### Daily pieces

```text
6,480 × 8
= 51,840 pieces/day
```

### Skids/hr

```text
6,480 / 4,000
= 1.620 skids/hr
```

### Minutes/skid

```text
60 / 1.620
= 37.04 min/skid
```

### Equivalent skids/day

```text
51,840 / 4,000
= 12.96 skid-equivalents/day
```

### Whole skids/day

```text
CEILING(12.96)
= 13 whole skids/day
```

### Line-side positions

```text
CEILING(1.620 × 1.25)
= CEILING(2.025)
= 3 positions
```

---

## 29. Material B

### Pieces/hr

```text
18 × 1 × 60
= 1,080 pieces/hr
```

### Skids/hr

```text
1,080 / 1,200
= 0.900 skids/hr
```

### Line-side positions

```text
CEILING(0.900 × 1.25)
= CEILING(1.125)
= 2 positions
```

---

## 30. Total Inbound Positions

```text
3 + 2
= 5 inbound skid positions
```

---

## 31. Finished Pallets

### Pallets/hr

```text
18 × 60 / 720
= 1.50 pallets/hr
```

### Minutes/pallet

```text
60 / 1.50
= 40.0 min/pallet
```

### Pallets/day

```text
1.50 × 8
= 12 pallet-equivalents/day
```

### Finished buffer positions

```text
CEILING(1.50 × 1.00)
= 2 positions
```

---

## 32. Total Staging Positions

```text
5 inbound
+ 2 finished
= 7 total staging positions
```

This result means **seven concurrent pallet/skid footprints** under the stated coverage assumptions.

It does not mean seven pallet moves per hour or seven operators.

---

# Part G — Physical Layout Review

## 33. Convert Positions to Real Space Separately

The planner does not know:

- pallet length/width,
- skid overhang,
- rack geometry,
- dunnage footprint,
- forklift turning radius,
- aisle width,
- pedestrian separation,
- machine clearance,
- electrical-panel clearance,
- fire extinguisher access,
- sprinkler/storage restrictions,
- egress,
- ergonomic reach,
- empty-container return.

After determining position counts, a physical layout review is required.

---

## 34. Material Handling Cadence

`Minutes / Skid` is useful for planning replenishment workload.

However, actual material-handler routes should account for:

- delivery batch size,
- travel distance,
- pickup time,
- drop time,
- scan/transaction time,
- empty return,
- congestion,
- queueing,
- simultaneous calls,
- multiple material types.

The planner currently provides demand cadence, not a complete milk-run simulation.

---

## 35. Related Documents

- **QS-AE-MFTP-001** — Quick Start
- **WI-AE-MFTP-001** — Planning Procedure
- **REF-AE-MFTP-001** — Takt, Capacity & Labor Calculations
- **LIM-AE-MFTP-001** — Known Limitations
- **DATA-AE-MFTP-001** — Data Handling & Repository Model

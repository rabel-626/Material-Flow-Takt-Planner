# Abel-Engineering Machine Rate, Material Flow & Takt Planner

**Current app version:** 1.4  
**Application file:** `Abel-Engineering_Material-Flow-Takt-Planner_v1.4.html`  
**Type:** Standalone browser-based engineering planning tool  
**Network requirement:** None  
**Primary companion tool:** Abel-Engineering Cycle Time & Labor Study

## Overview

The **Abel-Engineering Machine Rate, Material Flow & Takt Planner** converts a Cycle Time & Labor Study report into a production-planning view for:

- process takt and capacity comparison,
- labor bottleneck identification,
- material consumption rates,
- incoming supply-skid demand,
- line-side material staging requirements,
- finished-pallet throughput,
- downstream pallet-buffer requirements, and
- downloadable planning/report outputs.

The application is intended to bridge **observed cycle-time data** with **production-rate and material-flow planning**. It does not replace the source timing study. Instead, it imports the flattened **Power BI Report JSON** exported by the Cycle Time & Labor Study and applies planning assumptions to that data.

The tool runs locally in the browser and does not require a server or internet connection.

---

## Core Workflow

1. Complete or load a study in the **Abel-Engineering Cycle Time & Labor Study**.
2. Export the study's **Power BI Report JSON**.
3. Open the Material Flow & Takt Planner HTML file in a modern browser.
4. Select **Import Report JSON** and load the exported Cycle Time report.
5. Enter or verify the planning basis:
   - required machine rate,
   - net production hours per day,
   - material-flow rate basis,
   - inbound line-side coverage,
   - finished-goods buffer coverage,
   - units per finished pallet, and
   - takt timing basis.
6. Review the **Takt / Capacity** results.
7. Review and complete the **Material Flow** inputs.
8. Export the planning JSON, material CSV, or HD/vector PDF report as required.

---

## Cycle Time → Takt Planner Handshake

Version 1.4 supports the optional station-level field:

`SupplyPiecesPerSkid`

When this value exists in the Cycle Time & Labor Study report, the Takt Planner automatically uses it as the default **Pieces / Supply Skid** quantity for the corresponding station/material.

### Skid Quantity Precedence

The planner uses the following precedence:

1. **Planner override**
2. **Imported `SupplyPiecesPerSkid` value**
3. **Manual entry if no imported value exists**

An imported value is identified in the Material Flow table as:

`↳ Cycle Time import`

A value changed in the Takt Planner is identified as:

`↳ planner override`

### Important Quantity Distinction

The following fields represent different physical concepts and should not be substituted for one another:

| Field | Meaning |
|---|---|
| `QtyPerCarton` / Qty per Unit | Physical or repeated quantity required per finished production unit |
| `RefillQty` | Typical quantity replenished during one line-side refill/service event |
| `SupplyPiecesPerSkid` | Total usable pieces contained on one incoming supply skid |

**Do not use `RefillQty` as Pieces / Supply Skid.** A refill event and an incoming supply skid are different logistics units.

---

## Application Tabs

## 1. Overview

The Overview tab establishes the planning basis and provides a high-level comparison of production capability and staging demand.

### Source Report

Displays identifying information from the imported Cycle Time report, including:

- Study ID,
- Study / line identity,
- job or SKU,
- study date,
- source app version, and
- source schema version.

### Planning Basis

#### Required Machine Rate

Required production demand in cycles or units per minute.

**Required takt:**

`Takt sec/unit = 60 / Required CPM`

#### Net Production Hours / Day

The amount of actual production time available during the planning day.

Use **net production hours**, rather than paid shift length, when breaks, scheduled changeovers, or other excluded time are intentionally removed from the production window.

**Daily production:**

`Units/day = CPM × 60 × Net Production Hours`

#### Material Flow Rate Basis

Material consumption and finished-pallet calculations can be driven by:

- Required production rate
- Constrained maximum rate
- Labor bottleneck rate
- Machine rate
- Line speed / belt-derived rate
- Custom material rate

Use **Required production rate** when material storage should be sized to demand rather than theoretical maximum capacity.

#### Inbound Line-Side Coverage

Defines how many hours of incoming material should be staged at the production line at one time.

#### Finished-Goods Buffer Coverage

Defines how many hours of completed pallets may accumulate downstream before removal.

#### Units / Finished Pallet

Converts finished-unit throughput into pallet frequency and buffer-space requirements.

#### Takt Timing Basis

Process-capacity calculations can use:

- Mean cycle time
- P90 cycle time
- Operator-weighted mean

---

## Rate & Daily Maximum Comparison

The planner compares the available production-rate signals:

- Required Rate
- Labor Bottleneck
- Machine Rate
- Line Speed / Belt Rate
- Constrained Maximum

### Labor Bottleneck

For each timed process:

`Effective sec/unit = Timing sec/action × Qty/unit / Current Positions`

`Process Capacity CPM = 60 / Effective sec/unit`

The **labor bottleneck** is the lowest positive process capacity among eligible timed process rows.

### Constrained Maximum

The constrained maximum is the lowest available positive capacity signal among:

- labor bottleneck,
- machine/cartoner rate, and
- belt/line-speed rate.

Conceptually:

`Constrained Max CPM = MIN(available positive capacity signals)`

This represents a theoretical rate limit. It does not automatically account for downtime or OEE losses.

---

## Storage & Handling Snapshot

The Overview tab converts production flow into estimated concurrent staging positions.

### Incoming Material

For each material:

`Line-Side Positions = CEILING(Skids/hr × Inbound Coverage Hours)`

The planner then sums material-level positions.

### Finished Goods

`Finished Buffer Positions = CEILING(Pallets/hr × Finished-Goods Coverage Hours)`

A **position** represents one physical skid or pallet footprint. It is not:

- a worker,
- a forklift move,
- total daily pallet movement, or
- a warehouse safety-stock quantity.

These values are intended primarily for **line-side and floor-layout planning**.

---

## 2. Takt / Capacity

The Takt / Capacity tab compares each eligible process step with the required production pace.

### Process Takt Chart

Each bar represents:

`Timing statistic × Qty/unit / Current Positions`

The horizontal takt line represents:

`60 / Required CPM`

A process above the takt line cannot sustain the required rate with the current number of parallel positions.

### Process Capacity Detail

For each timed process, the planner reports:

- process step,
- segment,
- timing basis sec/action,
- quantity per unit,
- current positions,
- effective sec/unit,
- capacity CPM,
- required CPM,
- load percentage,
- required positions,
- additional positions required, and
- takt status.

### Required Positions

`Required Positions = CEILING(Timing sec/action × Qty/unit / Takt sec/unit)`

This is a mathematical capacity result only. Engineering review is still required for:

- ergonomics,
- physical access,
- reach,
- work-zone length,
- worker interference,
- shared tasks, and
- practical station layout.

Archived rows and support-segment rows are excluded from paced process-capacity calculations.

---

## 3. Material Flow

The Material Flow tab converts production rate into material demand and supply-skid frequency.

### Station Consumption & Supply Skids

Each eligible station can provide:

- station,
- segment,
- material/component,
- quantity per unit,
- imported refill quantity,
- pieces per supply skid,
- pieces/min,
- pieces/hr,
- daily pieces,
- skids/hr,
- minutes/skid,
- equivalent skids/day,
- whole skids/day, and
- line-side skid positions.

### Material Consumption

`Pieces/min = Material Planning CPM × Qty/unit`

`Pieces/hr = Pieces/min × 60`

`Daily Pieces = Pieces/hr × Net Production Hours`

### Supply Skid Demand

`Skids/hr = Pieces/hr / Pieces per Supply Skid`

`Minutes/skid = 60 / Skids/hr`

`Equivalent Skids/day = Daily Pieces / Pieces per Supply Skid`

`Whole Skids/day = CEILING(Equivalent Skids/day)`

### Line-Side Skid Positions

`Positions = CEILING(Skids/hr × Inbound Coverage Hours)`

### Material-Level Aggregation

Rows using the same material name are combined into a material-level summary.

If the same material is assigned conflicting **Pieces / Supply Skid** quantities, the planner flags a:

`⚠ skid qty conflict`

The conflicting values are not silently combined into one skid-demand calculation.

For reliable aggregation across repeated studies, consistent material naming should be used. A future material master key such as `MaterialID` or `ComponentSKU` would further improve cross-study reliability.

---

## Manual Materials

Use **Add Manual Material** when a consumed item is not represented by an imported Cycle Time station.

Manual rows can define:

- material name,
- quantity per finished unit,
- pieces per supply skid, and
- whether the row is included in calculations.

Manual materials are stored in planner state and in the Planning JSON.

---

## Clear Skid Inputs

**Clear Skid Inputs** removes planner-entered skid quantity overrides.

For imported stations:

- the planner override is removed, and
- the original Cycle Time `SupplyPiecesPerSkid` value is restored when available.

For manual material rows:

- the skid quantity is reset to blank/zero.

This makes it possible to return to the source study's logistics assumptions without reimporting the report.

---

## Finished Throughput Pallets

Finished-goods calculations use the selected **Material Flow Rate Basis**.

### Pallets per Hour

`Pallets/hr = Planning CPM × 60 / Units per Finished Pallet`

### Minutes per Pallet

`Minutes/pallet = 60 / Pallets/hr`

### Pallets per Day

`Pallets/day = Pallets/hr × Net Production Hours`

### Finished Pallet Buffer

`Buffer Positions = CEILING(Pallets/hr × Finished-Goods Coverage Hours)`

---

## 4. Data / Export

The Data / Export tab provides compatibility checks and output tools.

### Import Compatibility

The app checks the imported report for:

- recognized report schema,
- Study data,
- Station data,
- Qty / Unit,
- Material / Component,
- timing statistics,
- Current Positions,
- machine-rate data,
- belt-derived rate data, and
- optional `SupplyPiecesPerSkid` handshake values.

A warning does not necessarily mean the source study is invalid. It indicates that one or more downstream planner calculations may not have the desired source information.

---

## Planning JSON

### Export

**Download Planning JSON** saves the planning session without duplicating the source timing observations.

Planning JSON includes:

- source study identity,
- planning assumptions,
- rate and capacity outputs,
- process results,
- material results,
- finished-goods results,
- station overrides, and
- manual materials.

Current planning schema:

`Abel-Engineering.MaterialFlowTaktPlan`

Planning schema version:

`1.2`

### Import

**Import Planning JSON** restores saved planning assumptions and overrides.

A complete resumed analysis normally uses:

**Matching Cycle Time Report JSON + Planning JSON**

The planning file may be loaded before the source report. Its Study ID is retained so the planner can recognize the matching report when it is later imported.

If the currently loaded source report belongs to a different Study ID, the app warns before applying report-linked overrides.

---

## Material CSV

**Download Material CSV** exports the material-level flow summary for use in spreadsheets, Power BI, engineering analysis, or downstream reporting.

Exported fields include:

- Material
- SourceStations
- QtyPerCarton
- PiecesPerSupplySkid
- PiecesPerHour
- DailyPieces
- SkidsPerHour
- MinutesPerSkid
- EquivalentSkidsPerDay
- WholeSkidsPerDay
- LineSidePositions

---

## HD / Vector PDF

Use **Print / Save HD PDF** to generate the printable planning report through the browser print system.

The report includes:

- planning basis,
- rate/capacity KPIs,
- storage and material-handling snapshot,
- vector takt chart,
- process-capacity detail,
- material-flow detail, and
- finished-goods planning information.

The report uses vector text, tables, and a vector SVG takt chart so the output remains sharp when saved as PDF.

The print layout is designed for **US Letter landscape**.

---

## Local Storage and Session Behavior

The tool runs entirely in the browser.

Planner state is stored using browser `localStorage`, including:

- planning assumptions,
- station overrides,
- manual materials, and
- source Study ID.

The imported Cycle Time report itself is not intended to replace the original source file. Keep the source **Power BI Report JSON** with the planning output so timed station data can be reloaded when needed.

No network connection is required for normal operation.

---

## Expected Source Report

The primary import is the **Power BI Report JSON** generated by the Abel-Engineering Cycle Time & Labor Study.

Expected report schema:

`Abel-Engineering.CartonerPowerBIReport`

The planner expects the report to contain, at minimum, Study and Stations arrays. Additional timing, machine-rate, belt/pace, and material fields expand the available calculations.

For the automatic supply-skid handshake, use **Cycle Time & Labor Study v1.23 or later** and populate **Pieces / Supply Skid** at the relevant role/station.

---

## Recommended Engineering Use

A typical use case is:

1. Observe and time the production process.
2. Establish station quantities and current positions in the Cycle Time app.
3. Identify the material/component handled at each role.
4. Enter `SupplyPiecesPerSkid` when the incoming pack quantity is known.
5. Export the Cycle Time Power BI Report JSON.
6. Import it into the Takt Planner.
7. Set required production rate and net production hours.
8. Confirm material quantities represent **physical consumption per finished unit**.
9. Review bottleneck and takt results.
10. Review incoming skid frequency and line-side storage.
11. Enter finished pallet quantity and review downstream staging.
12. Export the Planning JSON and PDF for the engineering record.

---

## Engineering Notes and Limitations

- **Qty / Unit must be validated as physical consumption.** A labor-study repetition quantity may not always equal material pieces consumed.
- **Refill quantity and skid quantity are separate.**
- **Storage-position outputs are planning estimates**, not warehouse inventory-policy calculations.
- **Required positions are mathematical timing results**, not a substitute for layout or ergonomic validation.
- **Daily maximum calculations assume the selected rate is sustained through all entered net production time.**
- Downtime, OEE, planned stops, scrap, yield loss, and changeover losses are not automatically added unless represented through the planning assumptions.
- Material aggregation currently depends on material display names.
- Conflicting pieces-per-skid values for the same material are flagged rather than automatically reconciled.
- The app is intended as an engineering decision-support tool; final staffing, floor-space, inventory, and production commitments should be validated against actual operating conditions.

---

## Suggested File Set

For a traceable study package, retain:

```text
Cycle-Time-Labor-Study.html
Cycle-Time-PowerBI-Report.json
Material-Flow-Takt-Planner.html
Material-Flow-Takt-Planning.json
Material-Flow.csv
Material-Flow-Takt-Report.pdf
README.md
```

Keeping the source study/report and planner output together preserves both the observed data and the assumptions used for planning.

---

## Version Notes

### v1.4

- Added automatic `SupplyPiecesPerSkid` handshake from Cycle Time & Labor Study reports.
- Added imported-versus-override source indicators for skid quantities.
- Planner overrides take precedence over imported skid quantities.
- Clearing skid inputs restores imported Cycle Time quantities where available.
- Added supply-skid handshake status to Import Compatibility.
- Retained manual skid entry for older reports and materials without imported skid quantities.
- Maintained standalone local-browser operation.
- Retained Planning JSON import/export, Material CSV export, and HD/vector PDF reporting.

---

## Ownership

**Abel-Engineering**  
Machine Rate, Material Flow & Takt Planner

Standalone engineering planning utility for cycle-time, takt, material-consumption, and staging analysis.

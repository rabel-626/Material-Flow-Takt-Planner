---
documentId: DATA-AE-MFTP-001
title: Material Flow & Takt Planner — Data Handling & Repository Model
tool: Material-Flow-Takt-Planner
documentVersion: 1.1
toolVersion: 1.5.1
status: Review Draft
updated: 2026-09-09
---

# Data Handling & Repository Model

## 1. Purpose

This document describes how **Material Flow & Takt Planner v1.5.1** reads source data, maintains working state, saves controlled planner revisions, restores saved plans, and produces reporting outputs.

---

## 2. Data Architecture

The preferred workflow is:

```text
01_CYCLE_TIME
   |
   | verified Cycle Time Report JSON
   v
Material Flow & Takt Planner
   |
   | calculations + planner assumptions
   v
03_TAKT_MATERIAL_FLOW
```

The planner does not overwrite the Cycle Time source study.

---

## 3. Accepted Source Report Schema Names

The current planner recognizes:

```text
Abel-Engineering.CartonerPowerBIReport
MPG.CartonerPowerBIReport
```

It also validates report shape.

At minimum the source must contain:

- `Study` array with at least one object,
- `Stations` array.

Supported optional arrays are validated as arrays of objects when present, including current structures such as:

- `StationMaterials`
- `RoleTimingSummary`
- `PlacementSamples`
- `SupportTasks`
- `MachineRateChecks`
- `BeltSpeedChecks`
- `ProcessPaceSummary`
- `DataAccuracy`
- `DataQuality`
- `SequenceDefinitions`
- `WorkElementSamples`
- `SequenceReconciliation`
- `Operators`
- `ObservationSessions`
- `StationObservationSessions`

---

## 4. Current Source Behavior

Current schema-17 Cycle Time exports can supply:

- study identity,
- reference unit,
- one-to-many station materials,
- process relationships,
- current positions,
- raw timing observations,
- timing summaries,
- operator information,
- recurring support work,
- sequence elapsed timing,
- machine-rate checks,
- belt checks,
- source quality/readiness,
- pieces/supply skid.

Compatible older reports are read using fallbacks where implemented.

---

## 5. Primary vs Reference Studies

### Primary study

The primary report supplies:

- physical line configuration,
- stations/processes,
- current staffing/positions,
- material relationships,
- source rate signals,
- Study ID lineage.

### Reference studies

Reference reports contribute compact timing evidence after process matching.

They do not replace the primary source's:

- material structure,
- process quantities,
- line configuration,
- current positions,
- source identity.

This distinction is fundamental to the evidence model.

---

## 6. Process Matching for Evidence

Equivalent processes may be matched using current source identifiers and reviewed process identity, including stable station/process keys and normalized process labels.

Reference-study use should still be reviewed by an engineer for actual operational comparability.

---

## 7. Browser Working State

The planner retains local working state under the legacy-compatible key:

```text
mpg_material_flow_takt_planner_v1
```

The legacy key name is retained so existing local state is not disconnected by a branding change.

Working state includes items such as:

- required rate,
- required-rate source,
- net hours,
- timing basis,
- material-supply basis,
- coverage assumptions,
- finished-pallet quantity,
- station overrides,
- manual materials,
- evidence configuration,
- source Study ID,
- repository plan identity.

---

## 8. Browser Autosave

v1.5.1 adds a fuller browser recovery payload:

```text
fileType:
AbelEngineering.MaterialFlowTaktBrowserAutosave

schemaVersion:
2
```

The autosave contains:

- planner state,
- a copy of the current source report when available,
- app version,
- save timestamp.

The preferred browser autosave store uses **IndexedDB**.

If that fails, the application can use a localStorage compatibility fallback when the serialized payload is below its practical size guard.

Browser autosave is for recovery, not controlled recordkeeping.

---

## 9. Browser Autosave Timing

The application queues autosave shortly after state changes and also attempts to flush the recovery state when:

- the page is hidden,
- the page is unloaded.

The interface displays whether full autosave or compatibility-mode autosave is active.

---

## 10. Repository Requirements

The repository root must be an approved Abel Engineering Study Repository and contain the expected repository schema.

The planner expects repository routes including:

```text
01_CYCLE_TIME
02_DOWNTIME
03_TAKT_MATERIAL_FLOW
04_MULTI_STUDY_DOWNTIME
90_TEMPLATES
```

The planner uses:

```text
01_CYCLE_TIME
```

as its source route and:

```text
03_TAKT_MATERIAL_FLOW
```

as its authoritative plan route.

---

## 11. Repository Browser Compatibility

Direct repository access relies on browser directory access.

Use current desktop:

- Microsoft Edge, or
- Google Chrome.

When that capability is unavailable:

- browser autosave can remain available,
- Legacy File Load can remain available,
- repository Save/Open is disabled.

---

## 12. Legacy File Load

**LEGACY FILE LOAD** accepts a compatible Cycle Time **Report JSON** outside `01_CYCLE_TIME`.

It is a source migration/compatibility path only.

A plan created from a legacy source can still be saved only through **SAVE PLAN** to:

```text
03_TAKT_MATERIAL_FLOW
```

---

## 13. Current Plan Schema

The current v1.5.1 planner writes:

```text
schema:
MPG.MaterialFlowTaktPlan

schemaVersion:
1.4

appVersion:
1.5.1
```

The legacy `MPG` schema name is retained for compatibility.

The current open routine accepts plan schema versions:

```text
1.1
1.2
1.3
1.4
```

provided the plan otherwise passes compatibility checks.

---

## 14. Plan Contents

The current planner JSON contains major sections including:

- `source`
- `assumptions`
- `capacity`
- `processes`
- `materials`
- `labor`
- `finishedGoods`
- `plannerInputs`
- evidence/reliability content
- repository metadata

The plan stores planner assumptions and calculated results; the current repository revision also stores an exact source snapshot separately.

---

## 15. Repository Plan ID

A current plan receives a stable record ID similar to:

```text
TP-YYYYMMDD-STUDY-XXXX
```

The Study ID prefix and random token vary.

A plan can then receive multiple immutable revisions under the same planner record folder.

---

## 16. Repository Folder Structure

The current path is organized approximately as:

```text
03_TAKT_MATERIAL_FLOW
  / PLANT
  / LINE
  / YYYY
  / YYYY-MM
  / TP-RECORD-ID
```

The saved plan retains the repository relative path.

---

## 17. Immutable Revision ID

Each save creates a new revision token based on:

- save timestamp,
- random suffix.

A prior revision is not overwritten by the normal save workflow.

This provides a more auditable revision history.

---

## 18. Current Saved Revision Package

A current v1.5.1 save produces five companion outputs:

### Exact source snapshot

```text
...__SOURCE_REPORT.json
```

Role:

```text
source_report_snapshot
```

### Material summary

```text
...__MATERIAL.csv
```

Role:

```text
material_summary
```

### Timing/reliability summary

```text
...__RELIABILITY.csv
```

Role:

```text
timing_evidence
```

### Package manifest

```text
...__MANIFEST.json
```

### Planner state

```text
...__PLAN.json
```

Role:

```text
planner_state
```

The plan file is the final commit marker for the revision.

---

## 19. Package Manifest

Current manifest identity:

```text
fileType:
AbelEngineering.StudyPackageManifest

schemaVersion:
2

recordType:
TAKT_MATERIAL_FLOW
```

The manifest records:

- record ID,
- revision ID,
- source Study ID,
- source snapshot filename,
- source snapshot SHA-256,
- reference Study IDs,
- plant,
- line,
- job,
- save time,
- relative path,
- output metadata.

---

## 20. Integrity Verification

Current revision outputs carry:

- file size,
- SHA-256 hash,
- role.

When a current plan is reopened, the application checks:

- manifest type,
- manifest version,
- record type,
- immutable-revision flag,
- record ID,
- revision ID,
- source Study ID,
- source snapshot identity,
- required output roles,
- file size,
- SHA-256 hash.

A mismatch stops the normal restore path.

---

## 21. Source Snapshot Lineage

Current schema-1.4 repository revisions include the exact Cycle Time source-report snapshot used for the plan.

On reopen:

1. the snapshot hash is checked,
2. the snapshot is parsed,
3. the report shape/schema is validated,
4. its Study ID is checked against plan lineage.

This prevents the planner from silently attaching a later report revision with the same Study ID.

---

## 22. Older Plan Source Resolution

Older supported plans may not contain an exact source snapshot.

For those plans, the application searches `01_CYCLE_TIME` by Study ID.

If multiple compatible source reports share the Study ID and the older plan cannot identify a unique revision, reopening stops rather than silently choosing one.

This is intentional lineage protection.

---

## 23. Save Order

The current save sequence conceptually is:

```text
1. Verify source and Study ID
2. Create new immutable revision names
3. Verify files do not already exist
4. Write exact source snapshot
5. Write Material CSV
6. Write Reliability CSV
7. Build Plan JSON + repository metadata
8. Build package manifest
9. Write manifest
10. Write Plan JSON as final commit marker
11. Verify final Plan JSON
```

A successful UI message indicates the revision was saved and verified.

---

## 24. Material CSV

The repository material CSV includes fields such as:

- Material
- SourceStations
- QtyPerReferenceUnit
- PiecesPerSupplySkid
- SkidQtyConflict
- PiecesPerHour
- DailyPieces
- SkidsPerHour
- MinutesPerSkid
- EquivalentSkidsPerDay
- WholeSkidsPerDay
- LineSidePositions
- MaterialSupplyRateBasis
- MaterialSupplyRatePerMin

This is a report output, not the authoritative resumable planner state.

---

## 25. Reliability CSV

The reliability output provides timing/evidence information suitable for external review.

It should be retained with the plan when evidence-based timing is material to a decision.

It is not a replacement for the Cycle Time source observations.

---

## 26. PDF Report

**EXPORT HD PDF REPORT** uses the browser print system and a dedicated vector takt chart.

The report includes current views such as:

- planning basis,
- rate/capacity KPIs,
- data-confidence information,
- labor analysis,
- storage snapshot,
- process takt chart,
- process capacity detail,
- material summary,
- station consumption,
- finished-goods planning.

PDF is a presentation/report record, not a resumable planner package.

---

## 27. Reset Planner Settings

Resetting planner settings:

- resets planning assumptions,
- resets skid planner inputs,
- removes manual materials,

while retaining:

- loaded source report,
- normalized reference evidence set.

This supports multiple planning scenarios from the same measured evidence.

---

## 28. Clearing the Source Study

Clearing the source study removes the active Cycle Time report context.

Do not clear or replace a source before saving required planning records.

---

## 29. Privacy and Network Behavior

The application is a standalone browser application.

Normal calculations occur client-side.

The application does not require an Abel Engineering backend server or external analysis API for calculations.

Data leaves browser memory only through user-directed mechanisms such as:

- Study Repository save,
- report download,
- browser print,
- local/synchronized folder access.

If the Study Repository is stored in a OneDrive-synchronized folder, OneDrive performs the external synchronization outside the planner.

---

## 30. Sensitive Data

Avoid unnecessary personal or sensitive information in:

- study names,
- notes,
- material descriptions,
- planner metadata,
- exported reports.

Use controlled engineering identifiers where possible.

---

## 31. Retention Guidance

For an approved planning record, retain together:

- immutable planner revision package,
- source Cycle Time study/report according to site retention rules,
- issued PDF if used for approval,
- supporting BOM/material specifications where manual assumptions were used,
- physical layout documentation where staging positions were converted to real floor space.

---

## 32. Do Not Edit Saved Revision Files Manually

Editing a current immutable revision file after save can cause the SHA-256 or file-size integrity check to fail.

If a planning assumption changes:

1. open the valid revision,
2. make the change in the planner,
3. save a new immutable revision.

---

## 33. Related Documents

- **QS-AE-MFTP-001** — Quick Start
- **WI-AE-MFTP-001** — Planning Procedure
- **REF-AE-MFTP-001** — Takt, Capacity & Labor Calculations
- **REF-AE-MFTP-002** — Material Flow & Staging Calculations
- **LIM-AE-MFTP-001** — Known Limitations

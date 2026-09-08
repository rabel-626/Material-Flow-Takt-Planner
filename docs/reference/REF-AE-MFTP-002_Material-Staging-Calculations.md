---
documentId: REF-AE-MFTP-002
title: Material Staging Calculations
tool: Material-Flow-Takt-Planner
documentVersion: 0.1
status: Testbench
---


# Material Flow & Takt Planner — Material Staging Calculations

## Material Consumption

`pieces/min = planning rate × pieces/reference unit`

`pieces/hr = pieces/min × 60`

## Supply Skids

`skids/hr = pieces/hr / pieces per supply skid`

`minutes/skid = 60 / skids/hr`

`line-side positions = ceil(skids/hr × inbound coverage hours)`

## Finished Pallets

`finished pallets/hr = planning rate × 60 / cartons per pallet`

`minutes/pallet = 60 / finished pallets/hr`

`finished pallet buffer positions = ceil(pallets/hr × finished-goods coverage hours)`

These are floor-planning estimates, not inventory-policy calculations.

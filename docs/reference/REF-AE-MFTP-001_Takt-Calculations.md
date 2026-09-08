---
documentId: REF-AE-MFTP-001
title: Takt and Capacity Calculations
tool: Material-Flow-Takt-Planner
documentVersion: 0.1
status: Testbench
---


# Material Flow & Takt Planner — Takt & Capacity Calculations

## Required Takt

`Required takt (sec/reference unit) = 60 / required units per minute`

## Effective Process Time

A simplified paced-process relationship is:

`effective sec/reference unit = workload sec/reference unit / current parallel positions`

## Process Capacity

`capacity units/min = 60 / effective sec/reference unit`

## Required Positions

`required positions = ceil(workload sec/reference unit / takt sec/reference unit)`

The mathematical result still requires physical and operational review.

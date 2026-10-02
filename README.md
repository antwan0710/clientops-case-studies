# ClientOps Case Studies

Real-world data reconciliation case studies built with **ClientOps**.

This public repository documents practical data-cleaning, entity-reconciliation, validation, and human-review workflows tested on real datasets.

> **Source-code policy:** This repository contains case studies and results only. The ClientOps application source code, internal implementation details, and proprietary processing logic are kept private.

## Case Studies

### 01 — Chicago Business Data Reconciliation

A real-world master-to-activity reconciliation using City of Chicago public data:

- **2,002** current-active business-license records
- **11,635** food-inspection activity records
- exact reconciliation using License Number
- one-to-many master → activity relationships
- ambiguous and invalid keys preserved for review
- **0 records silently dropped**

**Final reconciliation**

| Status | Records |
|---|---:|
| Linked | 5,122 |
| Unmatched | 6,444 |
| Ambiguous | 53 |
| Invalid key | 16 |
| **Total** | **11,635** |

➡️ [Read the full Chicago case study](cases/chicago-business-data.md)

## About ClientOps

ClientOps is a data-operations project focused on multi-source ingestion, mapping, normalization, duplicate/conflict review, entity reconciliation, provenance, human review, and cleaned export workflows.

The goal of these case studies is to document what happens when the workflow is tested against real data rather than only synthetic examples.

## Repository Scope

Public:
- case-study writeups
- real-world validation results
- high-level workflow descriptions
- non-sensitive screenshots and supporting documentation

Private:
- ClientOps application source code
- internal reconciliation implementation
- dedupe rules and thresholds
- test suite and proprietary processing logic

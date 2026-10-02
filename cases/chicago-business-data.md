# ClientOps Real-World Reconciliation Case #1 — Chicago Business Data

## Overview

This project is a real-world validation of **ClientOps**, a data operations workflow designed to clean, reconcile, validate, and prepare multi-source business data.

Instead of relying only on synthetic test files, this case used two public datasets from the **City of Chicago**:

- **Business Licenses – Current Active** (dataset ID: `uupf-x98q`)
- **Food Inspections** (dataset ID: `4ijn-s7e5`)

The analysis focused on records in ZIP code **60647**.

The goal was to test whether ClientOps could handle a realistic master-to-activity reconciliation workflow without artificially inserting errors, duplicates, missing values, or conflicts.

> **Public case-study note:** This repository documents the workflow and results. The ClientOps application source code, internal reconciliation implementation, and proprietary processing logic are not included.

---

## Data Used

### Business License Master

The current-active business-license dataset contained:

- **2,002 business-license records**

It served as the master source for currently active licenses.

Relevant fields included:

- Business name
- Address
- License Number
- License type
- Location information

### Food Inspection Activity Data

The Food Inspections dataset contained:

- **11,635 inspection records**

It represented historical activity associated with licensed establishments.

Relevant fields included:

- Inspection ID
- Business name
- Address
- License Number
- Inspection date
- Inspection type
- Inspection result

The shared reconciliation key was:

- Business Licenses: `LICENSE NUMBER`
- Food Inspections: `License #`

---

## Why the Reconciliation Was Non-Trivial

This was not a simple one-record-to-one-record match.

The real Chicago data contained structures that commonly appear in production data operations work:

- One business or license can have many inspection events.
- A business at the same address can legitimately hold multiple licenses.
- Historical inspections can reference licenses no longer present in the current-active master.
- A business key can sometimes correspond to multiple current master records.
- Some activity records contain invalid or unusable keys.
- Similar-looking records are not always true duplicates.

Automatically merging records based only on name and address similarity would therefore risk false positives.

---

## What the Real Workload Exposed

### 1. Customer-Oriented Validation Did Not Fit Registry Data

The original workflow assumed CRM-style customer records and expected a contact method such as email or phone.

That assumption does not fit public business-registry and inspection records.

ClientOps was adjusted to distinguish record roles such as:

- customer
- business master
- activity/event

This allowed legitimate registry and event records to be processed without incorrectly rejecting them for missing CRM contact information.

### 2. CRM-Style Duplicate Detection Produced False Positives

The original duplicate logic relied heavily on business-name and address similarity.

The Chicago data showed why this can be unsafe:

- The same business at the same address can hold different valid licenses.
- The same licensed business can have multiple inspection events on different dates.

Role-aware duplicate gates were added so legitimate master and event records would remain distinct.

On the Food Inspections workload, duplicate-review candidate pairs were reduced from:

**90,793 → 987**

without automatically merging records.

### 3. Multi-File Processing Needed Persistent State

The real workload also exposed a workflow issue: processing one structured file and then switching to another cleared information needed for cross-file reconciliation.

The processing state was updated so multiple processed datasets could coexist during reconciliation.

---

## Final Business-Key Reconciliation

After addressing the real-world blockers, ClientOps reconciled all **11,635 Food Inspection records** against the **2,002 current-active Business License records** using exact License Number matching.

| Status | Records |
|---|---:|
| Linked | **5,122** |
| Unmatched | **6,444** |
| Ambiguous | **53** |
| Invalid key | **16** |
| **Total** | **11,635** |

Every inspection record was classified.

**0 records were silently dropped.**

---

## What the Results Mean

### Linked — 5,122

These inspection records had an exact License Number match to a single current-active business master record.

The output also confirmed legitimate one-to-many relationships:

**one business/license master → many inspection activity records**

Multiple inspections for the same licensed business remained separate activity records rather than being incorrectly collapsed into duplicate customers.

### Unmatched — 6,444

These inspection records contained a non-zero License Number but did not have an exact match in the **current-active** Business License master.

They are **not automatically data-quality errors**.

Because the Food Inspections dataset contains historical activity while the master contains current active licenses, possible explanations include:

- expired licenses
- closed businesses
- historical license records
- license changes
- businesses no longer active

The correct output is therefore `unmatched`, not automatically `bad data`.

### Ambiguous — 53

These activity records used a License Number that could not be resolved to exactly one master record.

ClientOps did not silently choose one candidate or automatically merge the master records.

Instead, the records were classified as **ambiguous** for review.

### Invalid Key — 16

Sixteen Food Inspection records contained:

`License # = 0`

ClientOps preserved the source value but did not create a false relationship.

They were classified as **invalid key**.

---

## Data Integrity Principles

Throughout this case:

- No synthetic errors were inserted.
- No duplicates were artificially created.
- No missing values were fabricated.
- No unmatched records were deleted.
- No historical records were automatically labeled as incorrect.
- No ambiguous relationships were silently resolved.
- No invalid business keys were rewritten.
- Original source values were preserved.
- Human-review cases remained visible rather than being automatically merged.

---

## Outcome

The Chicago case demonstrated that ClientOps can:

- ingest real public business data;
- distinguish master records from activity events;
- preserve provenance;
- perform exact business-key reconciliation;
- support one-to-many relationships;
- separate deterministic matches from unresolved cases;
- keep ambiguous and invalid records visible for review.

The final reconciliation classified all **11,635 inspection records** into:

- **5,122 linked**
- **6,444 unmatched**
- **53 ambiguous**
- **16 invalid-key**

This case also showed why testing data tools against real workloads matters: assumptions that looked reasonable in synthetic CRM-style testing did not hold when applied to real registry and historical activity data.

The resulting product changes were targeted responses to concrete real-world blockers rather than broad feature expansion.

---

## Scope of the Claim

This case validates **technical execution on one real-world public-data workload**.

It does **not** by itself demonstrate commercial demand, customer adoption, or revenue. Those are separate validation questions.

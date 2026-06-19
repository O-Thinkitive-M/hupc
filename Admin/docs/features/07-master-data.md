# Master Data — Admin Portal Feature Spec

## Purpose
Manage the shared reference catalogs the EMR depends on: bulk **Data Import**, **ICD-10** diagnosis codes, **CPT** procedure codes, and **Payer Master** — consumed by appointments, billing, templates, and clinical charting.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin | Manage all catalogs, run imports |
| Billing Admin | Maintain CPT, Payer, Allowed-Amount linkages |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| MD-1 | admin | bulk import master data (codes, payers, patients) | migration from Medent is fast |
| MD-2 | admin | search/add/edit ICD-10 codes | diagnoses are codeable |
| MD-3 | admin | search/add/edit CPT codes incl. add-on/variant codes | procedures bill correctly |
| MD-4 | admin | manage the Payer Master incl. mailing address & contact | claims route to the right payer |

## Primary Workflow
1. Admin opens **Master** (screen 15-39-21).
2. **Data Import:** upload a file (CSV/migration), map columns, validate, import; errors reported per row.
3. **ICD-10:** search/add/edit codes; codes are dropdown-only downstream (no free text).
4. **CPT:** add/edit codes; configure add-on/variant codes (e.g., 90853 group therapy) and per-appointment-type auto-attach.
5. **Payer Master:** add/edit payer (Name, Payer ID, Client Type, Status, **Mailing Address**, **Contact Number**).
6. Changes audit-logged; downstream config refreshes.

## Screens
- Master: `Screenshot from 2026-06-18 15-39-21.png`

## Data Entities
| Entity | Key Fields |
|--------|------------|
| ImportJob | jobId, type, fileRef, status, rowErrors[] |
| Icd10Code | code, description, status |
| CptCode | code, description, isAddOn, variantOf, status |
| Payer | payerId, name, clientType, status, **mailingAddress**, **contactNumber** |

## Business Rules
- **ICD-10 and CPT entered via dropdown only — not free text** (prevents accidental deletion) (SRS §7331).
- CPT add-on/variant codes auto-attach per appointment type config (SRS §3.x; module 09).
- Payer Master is the source for Allowed Amount + Network Status payer references.
- Catalog changes are audit-logged.

## MoM Decisions / Deltas (authoritative)
- [x] **Payer Master adds Mailing Address + Contact Number** ("Contact number is must" — Robert) [MoM §14].
- [x] Payer columns: Payer Name / Payer ID / Client Type / Status (+ new address/contact) [MoM §14].
- [x] New insurance + new CPT codes can be added **inline from the Allowed Amount tab** [MoM §13] (module 10).
- Migration source is **Medent**; Data Import supports the legacy migration.

## Dependencies
- Module 10 (Allowed Amount / Fee Schedule use CPT + Payer) · Module 05 (network status per payer) · Module 09 (CPT per appointment type) · Module 08 (codes referenced in notes) · Module 13 (audit).

## Open Items
- Import file formats/templates and field mappings not finalized.
- ICD-10/CPT catalog source (CMS load vs Medent export) TBD.
- "Client Type" value set undefined.

## Acceptance Criteria
- [ ] Bulk Data Import with per-row validation and error reporting.
- [ ] ICD-10 and CPT searchable, add/editable, dropdown-only downstream.
- [ ] Payer Master includes Payer ID, Client Type, Status, **Mailing Address**, **Contact Number**.
- [ ] CPT supports add-on/variant codes for appointment-type auto-attach.
- [ ] All catalog changes audit-logged.

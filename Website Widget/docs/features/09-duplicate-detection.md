# Duplicate Detection — Website Widget Feature Spec

## Purpose
- Define duplicate detection that runs **at submission, before any Lead or Patient record is created**, preventing existing patients from creating duplicate records and routing them appropriately.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Submits the new-patient flow |
| Existing Patient | Matched → redirected to Portal |
| HUPC staff | Review lead touchpoints (lead-match path) |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| DD-1 | Existing patient who picked "New" | to be caught | I don't create a duplicate |
| DD-2 | HUPC | duplicates prevented pre-creation | data stays clean |
| DD-3 | Staff | repeat submissions logged on the lead | I see all touchpoints |
| DD-4 | New prospect | a clean new lead when truly new | my journey starts |

## Primary Workflow
1. Patient submits the new-patient flow.
2. Before any record creation, all three match keys run **simultaneously** against the matching scope.
3. **Existing Patient match** → no record created; redirect to Portal login with message; audit log entry; no staff task.
4. **Existing Lead match** → no new lead; logged as additional touchpoint; "Last Activity" updated; visible in lead activity log.
5. **No match** → create new Lead in Lead Management; enters standard new-patient lead workflow.

## Matching Scope
| Target | Statuses |
|--------|----------|
| Existing Patient records | Active, Inactive, Archived |
| Open Lead records | Not yet converted to Patient |

## Match Keys
| Key | Fields compared | Purpose |
|-----|-----------------|---------|
| A | Full Name + DOB + Mobile | Standard exact-match on full identity |
| B | Mobile + DOB | Catches name changes (e.g., post-marriage surname) |
| C | First Name + Mobile + DOB | Catches differently-entered last names (typo, hyphenation) |

- Any single key matching → submission treated as duplicate.

## Outcome Handling
| Outcome | Behavior |
|---------|----------|
| Patient match | No record created; redirect to Portal ("We found an existing account for you. Please log in to book your appointment."); audit log (timestamp, match key, target Patient ID); no staff task |
| Lead match | No new lead; logged as touchpoint; Last Activity updated; staff sees it on next review |
| No match | New Lead created; standard new-patient lead workflow |

## Screens
- No screen capture provided; per SRS/MoM, expected UI: Convert-to-Patient style comparison view showing matching fields when a potential duplicate exists; Portal redirect message for patient matches.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Match evaluation | Keys run, match found?, matched record type/ID |
| Audit log entry | Timestamp, match key, target Patient ID |
| Lead touchpoint | Submission payload, timestamp |

## Business Rules
- Detection occurs **prior to** account/lead creation, not after full form submit-and-save.
- Patient records are never hard-deleted; duplicates created in error are flagged and merged.
- Comparison view shows existing vs incoming on potential duplicate.
- Full duplicate logic specified in Lead Management.

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) **Convert-to-Patient duplicate-check** locked: on matching record, show matching fields + two options: **Create New Patient Account** OR **Update Existing Patient Account** (staff-side, in Lead Management).
- Duplicate detection runs **pre-creation**; comparison/merge, never hard-delete.
- Widget self-resolves patient matches via Portal redirect (no staff task on that path).

## Dependencies
- Patient Identification (file 02) match keys.
- Lead Management (lead creation, touchpoints, convert-to-patient).
- Patient Portal (redirect target).
- EHR patient records (match targets).

## Open Items
- Merge-conflict resolution UX edge cases — handled in Lead Management spec.

## Acceptance Criteria
- A submission matching any key against a Patient redirects to Portal with no record created + audit entry.
- A submission matching an open Lead adds a touchpoint without creating a new lead.
- A truly new submission creates a new Lead.
- Inactive/Archived patients are included in matching scope.

# Provider Selection — Website Widget Feature Spec

## Purpose
- Define **Step 6**: present eligible providers (cards), apply filtering rules (hard + soft), surface eligibility-aware cost, and expose the provider detail screen.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Browses, filters, selects a provider (or skips) |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| PS-1 | Prospect | only providers I'm eligible for | I don't pick someone I can't see |
| PS-2 | Prospect | copay/self-pay cost per provider | I choose by cost |
| PS-3 | Prospect | to filter by language/therapy focus | I find the right fit |
| PS-4 | Prospect | a detail screen | I learn about a provider before booking |
| PS-5 | Prospect | to submit without choosing | staff assigns later |

## Primary Workflow
1. After OTP, widget shows eligible providers for the patient's service, payer, age, mode, location.
2. All hard filters run simultaneously; failing providers are hidden.
3. Soft filters re-rank (do not hide).
4. Eligibility-aware display: copay/deductible (insurance) or self-pay rate (self-pay) shown inline per card.
5. Patient taps **View Details** → provider detail screen → **Book Appointment** returns with provider pre-selected.
6. Patient selects provider → time-slot selection (file 08); or submits without provider (file 11).

## Provider Card
| Element | Notes |
|---------|-------|
| Name + credential | |
| Specialty / sub-specialty tags | |
| Accepted insurance plans | |
| Accepted age ranges | |
| Practice locations | |
| Telehealth / In-Person availability | |
| Accepting New Patients flag | |
| Patient reviews summary | Aggregated rating |
| View Details button | Opens detail screen |
| Cost line | Copay/deductible (insurance) or self-pay rate (self-pay) |

## Filtering Rules
| # | Filter | Type | Rule |
|---|--------|------|------|
| 1 | Service / credential match | Hard | Med Mgmt→MD/APRN/PA; Psychotherapy→LMHC/LMFT/RMHCI/LCSW/PsyD/PhD; Group Therapy→group-flag (telehealth only); NCT→PsyD/PhD, in-person only; TMS→TMS-certified, in-person, Winter Garden only |
| 2 | Appointment mode | Hard | Telehealth/In-Person match; NCT+Telehealth and TMS+Telehealth blocked |
| 3 | Age range | Hard | DOB vs accepted ranges: Pediatric 0–11, Adolescent 12–17, Adult 18–64, Geriatric 65+ |
| 4 | Payer enrollment | Hard | Hide providers not credentialed/enrolled with patient's payer; Self-Pay bypasses |
| 5 | Accepting-new flag | Hard | New-patient appts require Accepting New Patients = true; full providers hidden |
| 6 | Therapy sub-specialty | Soft | Matching focus ranked higher; non-matching shown below; re-ranks on change |
| 7 | Language | Soft | Matching language ranked higher; non-matching not hidden |

## Provider Detail Screen
- Full name, credentials, professional title; profile photo; education/training; specialty + sub-specialty tags; therapy modes (Telehealth/In-Person); practice locations + working days; full insurance-accepted list; patient reviews + aggregated rating; **Book Appointment** (returns with provider pre-selected).

## Screens
- No screen capture provided; per SRS/MoM, expected UI: filter bar, sortable provider card list with inline cost, View Details, and a rich detail page.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Provider profile | Name, credential, specialties, age ranges, locations, modes, payers, accepting-new, language, reviews |
| Filter context | Service, mode, payer, DOB/age, location, therapy focus, language |
| Cost overlay | Copay/deductible or self-pay rate |

## Business Rules
- Hard filters hide; soft filters rank only.
- Self-Pay bypasses payer-enrollment filter and shows self-pay rate instead of copay/deductible.
- Providers full for new patients are hidden even if their calendar has open (follow-up-reserved) slots.
- Provider attributes sourced from hpcfl.com/providers (web-dev maintained, synced to widget).

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) **Provider filter set locked**: Appointment Mode, Appointment Type, Reason to Visit, Location, Language, Age Group, Insurance, Diagnosis, Therapy Modality. (Supersedes/expands SRS filter list — Reason to Visit + Diagnosis + Appointment Type added as patient-facing filters.)
- Provider list includes **Search button + sort + View Details** per provider.
- After OTP → see provider list → select provider, date, time slot → **Confirm Booking**.

## Dependencies
- Eligibility (file 05) for cost + eligibility-aware filtering.
- Availability/Time Slots (file 08).
- hpcfl.com/providers data source.

## Open Items
- "Pending Credentialing" provider visibility — pending HUPC confirmation.

## Acceptance Criteria
- Only providers passing all hard filters appear.
- Soft filters re-rank without hiding.
- Self-Pay shows self-pay rate; insurance shows copay/deductible.
- View Details opens the detail screen; Book returns provider pre-selected.
- Locked MoM filter set is available in the filter bar.

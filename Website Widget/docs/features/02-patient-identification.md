# Patient Identification — Website Widget Feature Spec

## Purpose
- Define **Step 1**: determine whether the person is a new or existing patient using established match keys, routing existing patients to the Portal and new patients into registration.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Provides identity fields; proceeds to registration |
| Existing Patient | Matched → redirected to Portal |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| PI-1 | Prospect | the system to know if I already have an account | I'm not asked to re-register |
| PI-2 | Existing patient | to be guided to log in | I book through my account |
| PI-3 | New patient | to continue to the form quickly | I can register in one screen |

## Primary Workflow
1. Widget collects identity inputs (Name, DOB, Mobile) as part of initial form entry.
2. System evaluates the three match keys against existing records:
   - **Match Key A**: Name + DOB + Mobile (primary)
   - **Match Key B**: Mobile + DOB (handles name changes)
   - **Match Key C**: First + Last + Mobile + DOB
3. **Existing patient detected** → redirect to Patient Portal login.
4. **New patient detected** → proceed to Step 2 (Registration Form, file 03).
5. Definitive duplicate detection re-runs at submission (file 09) before any record creation.

## Screens
- No screen capture provided; per SRS/MoM, expected UI:
  - Lightweight identity capture (Name, DOB, Mobile) at flow start.
  - If matched as existing: message + redirect ("We found an existing account for you. Please log in to book your appointment.").
  - If new: seamless transition into the full registration form.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Identity input | First name, last name, DOB, mobile number |
| Match result | Matched? (bool), match key triggered (A/B/C), target Patient/Lead ID |

## Business Rules
- All three match keys evaluated; any single match → treated as existing.
- Match Key B exists to catch name changes (e.g., post-marriage surname change).
- Match Key C catches differently-entered last names (typo, hyphenation) with aligned first name + contact.
- Identification routing is advisory; the authoritative duplicate gate is at submission (file 09).

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) DOB-driven **Minor/Adult auto-detection** is live: a minor DOB surfaces the Guardian section in registration.
- Existing vs New gate confirmed; existing patients always routed to Portal.
- No standalone deltas to the three match keys in MoM V2 — SRS match-key set remains authoritative.

## Dependencies
- Entry & Routing (file 01).
- Patient/Lead records in EHR + Lead Management (match targets).
- Duplicate Detection (file 09) for the authoritative submit-time check.

## Open Items
- None specific beyond shared widget open items (landline OTP, guardian-email constraint).

## Acceptance Criteria
- Entering identity of an existing patient triggers a Portal redirect, not registration.
- Entering a brand-new identity proceeds to the registration form.
- Name-change and last-name-typo scenarios are caught by Keys B and C respectively.
- Minor DOB triggers the Guardian section downstream.

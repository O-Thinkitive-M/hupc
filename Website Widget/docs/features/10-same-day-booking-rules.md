# Same-Day Booking Rules — Website Widget Feature Spec

## Purpose
- Define the same-day booking restriction: at most one **insurance-covered** appointment per calendar day across all providers, with a self-pay exception, plus enforcement at submission.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Books; may hit the restriction |
| HUPC staff | Apply override (self-pay conversion / insurance exception) |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| SD-1 | Patient | a clear message if I'm double-booking insurance | I understand payer rules |
| SD-2 | Patient | to convert the 2nd to self-pay | I keep both same-day visits |
| SD-3 | HUPC | the rule enforced everywhere | claims aren't denied |
| SD-4 | Staff | a logged override | exceptions are auditable |

## Primary Workflow
1. At booking submission, widget checks the patient's appointment history for the target date.
2. If no insurance-covered appointment exists that day → allow.
3. If an insurance-covered appointment already exists and this one is insurance-covered → **block** with explanatory message.
4. Patient chooses: **switch this booking to self-pay** OR **pick a different date**.
5. Self-pay second (and additional per-request) appointments are permitted same-day.
6. Staff-side override (self-pay conversion / explicit insurance exception) is logged in the audit trail.

## Rules
| Rule | Detail |
|------|--------|
| One insurance/day | At most one insurance-covered appointment per calendar day across ALL HUPC providers (payer industry rule) |
| Cross-provider | Applies even if appointments are with different providers |
| All surfaces | Applies via widget AND Patient Portal |
| Self-pay exception | 1st same-day = insurance-covered (any provider); 2nd same-day = self-pay (same/different provider); additional self-pay allowed per patient request |

## Conflict Message
- "You already have an insurance-covered appointment on [date]. Insurance typically does not cover two same-day appointments. You may book this as a self-pay appointment, or choose a different date."

## Screens
- No screen capture provided; per SRS/MoM, expected UI: blocking dialog at submit with two actions (Switch to Self-Pay / Choose Different Date).

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Same-day check | Target date, existing same-day insurance appt?, payment type of new booking |
| Override | Type (self-pay conversion / insurance exception), staff user, timestamp (audit) |

## Business Rules
- The restriction targets insurance-covered appointments; self-pay is exempt as the second+ booking.
- Enforcement happens at submission against appointment history for the target date.
- Example: insurance Med-Management visit + self-pay therapy visit same day is allowed.
- Override is staff-only and audited.

## MoM Decisions / Deltas (authoritative)
- No MoM V2 delta overrides the SRS same-day rule; SRS behavior remains authoritative.
- Consistent with MoM scheduling locks: active-patient/eligibility-aware enforcement applies across widget + portal.

## Dependencies
- Patient appointment history (Scheduling).
- Payment & Confirmation (file 12) for self-pay conversion path.
- Eligibility (file 05) / payment-type classification.

## Open Items
- Exact policy for "additional self-pay per request" beyond the second appointment — staff-discretion config.

## Acceptance Criteria
- A second same-day insurance-covered booking is blocked with the messaged options.
- Converting the second booking to self-pay allows it.
- The rule applies across different providers and across widget + portal.
- Staff overrides are logged.

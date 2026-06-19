# Booking Type Differentiation — Website Widget Feature Spec

## Purpose
- Define how booking behavior differs by patient type: follow-up (existing) patients book directly, new patients submit a request requiring staff confirmation; and support booking without provider selection.

## Actors / Roles
| Actor | Role |
|-------|------|
| Existing/Follow-up Patient | Direct booking via Portal |
| New Patient (prospect) | Submits request (pending confirmation) |
| Office Management staff | Reviews/confirms/assigns new-patient requests |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| BT-1 | Follow-up patient | to book directly | I don't wait on staff |
| BT-2 | New patient | to submit a request | staff verifies before confirming |
| BT-3 | New patient | to submit without picking a provider | staff can assign one |
| BT-4 | Staff | to validate before confirming | insurance/consent/assignment are correct |

## Primary Workflow
1. **Follow-up (existing)**: routed via Portal → books directly, subject to provider eligibility (file 07) + same-day rules (file 10) + appointment-type rules → immediately confirmed, visible on provider calendar.
2. **New patient**: completes widget flow → submits → appointment enters **"Requested — Pending Staff Confirmation"** → no auto-confirmation.
3. Office Management reviews: confirm, reassign, or reach out; validates insurance, consent forms, provider assignment.
4. **Without provider selection**: request can still be submitted; staff assigns a provider later (distinct from direct booking which requires provider + slot).

## Booking Type Matrix
| Type | Path | Confirmation | Provider required? |
|------|------|--------------|--------------------|
| Follow-up (existing) | Portal direct booking | Immediate | Yes (+ slot) |
| New patient (with provider) | Widget flow | Requested — pending staff | Yes (+ slot) |
| New patient (no provider) | Widget flow | Requested — staff assigns provider | No |

## Constraints on Follow-up Direct Booking
- Provider eligibility rules (file 07).
- Same-day booking restrictions (file 10).
- Appointment-type rules: TMS and NCT may still require staff confirmation per service-type config.

## Screens
- No screen capture provided; per SRS/MoM, expected UI: new-patient flow ends with a "request submitted / pending confirmation" success state; optional "submit without provider" path.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Appointment request | Status (Requested — Pending Staff Confirmation), patient, provider (optional), slot (optional) |
| Confirmation action | Staff user, action (confirm/reassign/contact), timestamp |

## Business Rules
- New-patient bookings are **requests**, never auto-confirmed.
- Staff validates insurance, consent forms, and provider assignment before finalizing.
- Provider selection is optional for new-patient requests; staff assigns if omitted.
- Follow-up direct bookings confirm immediately and appear on the provider calendar.

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) New-patient flow ends at **Confirm Booking** but enters pending-confirmation for staff review (request semantics retained from SRS; no MoM override).
- Inactive patients cannot be scheduled (staff lock) — reinforces that follow-up direct booking applies to Active patients only.
- Insurance reps cannot directly schedule — Lead Management submission path only (not a booking type here).

## Dependencies
- Provider Selection (file 07), Same-Day Rules (file 10), Eligibility (file 05).
- Office Management / Scheduling (staff confirmation, provider assignment).
- Patient Portal (follow-up direct booking).

## Open Items
- Which appointment types always require staff confirmation for follow-ups (TMS/NCT) — service-type config.

## Acceptance Criteria
- New-patient submissions land in "Requested — Pending Staff Confirmation".
- A new-patient request can be submitted without a provider.
- Follow-up bookings confirm immediately, subject to eligibility/same-day rules.
- Staff review precedes any new-patient confirmation.

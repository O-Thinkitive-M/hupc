# Appointments & Scheduler — Patient Portal Feature Spec

## Purpose
- Let patients book, reschedule, and cancel appointments from the portal.
- Support both **In-Person** and **Telehealth** appointment modes.
- Surface one-week provider availability with copay/deductible context.

## Actors / Roles
| Actor | Capability |
|-------|-----------|
| Patient (Adult) | Book/reschedule/cancel own appointments. |
| Guardian | Same actions for the active child account. |
| System | Enforces availability, 24h fee window, confirmations. |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| APT-1 | Patient | to book by picking provider + slot | I get care when I need it |
| APT-2 | Patient | to choose In-Person or Telehealth | I attend the way that suits me |
| APT-3 | Patient | to reschedule myself | I adjust without calling the office |
| APT-4 | Patient | to cancel myself | I free the slot |
| APT-5 | Patient | to see copay/deductible before booking | I know the cost upfront |

## Primary Workflow
1. Patient opens Appointments → selects **provider** (optionally a preferred provider) and **appointment mode** (In-Person / Telehealth).
2. System shows **one-week availability**; copay/deductible surfaced inline per provider card (eligibility-derived for insured, else self-pay).
3. Patient selects a slot → confirms → appointment created.
4. Confirmation sent via **email (EnGard) and SMS (AWS SNS)**; Telehealth bookings also get a Google Meet link (see 05).
5. **Reschedule / Cancel:** patient uses buttons on the appointment; provides **Reason** and may toggle **Notify Provider**.
6. **24-hour window alert:** if cancelled/rescheduled within 24 hours of the appointment, a **cancellation fee** is charged and deducted from patient balance.
7. Past appointments: encounter notes are NOT viewable (ROI rule).

## Screens
No screen capture provided; per SRS/MoM the expected UI:
- Provider selection with mode toggle (In-Person / Telehealth) and copay/deductible on each card.
- One-week availability grid with selectable slots.
- Appointment list with **Reschedule** + **Cancel** buttons.
- Cancel/reschedule dialog: 24h fee warning, Reason field, Notify Provider checkbox.
- Past-appointments view with notes hidden.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Appointment | apptId, patientId, providerId, mode (In-Person/Telehealth), slot, status |
| Availability | providerId, day, start, end, location, timeZone |
| CancellationEvent | apptId, reason, notifyProvider, within24h, feeApplied |
| CopayContext | apptType, copay, deductible, source (eligibility/self-pay) |

## Business Rules
- Appointment mode is In-Person or Telehealth (telehealth uses Google Meet).
- Availability shown for one week; respects provider time zone and block days.
- Patient-initiated reschedule/cancel allowed; within 24h triggers a fee deducted from balance.
- Reason captured; Notify Provider optional.
- Past-encounter notes blocked from patient view (ROI rule).

## MoM Decisions / Deltas (authoritative)
- **Self-service reschedule + cancel with 24-hour fee window** (MoM item 24).
- **Past appointment notes blocked from patient view (ROI rule)** (MoM item 24).
- **Appointment Mode column: In-Person / Telehealth** (MoM appointment-types delta).
- Appointment-request type queries route via Office Management, not the provider, for scheduling (MoM item 30).

## Dependencies
- Telehealth (05) for Google Meet links.
- Billing (08) for fee deduction and copay context.
- Notifications (10) for confirmations.
- Provider availability/block-days settings (staff EMR).

## Open Items
- Cancellation fee **amount** and exact policy — 24h window confirmed, amount TBD.
- Whether self-service rescheduling re-checks copay/eligibility — TBD.

## Acceptance Criteria
- Patient can book In-Person or Telehealth using one-week availability.
- Reschedule/Cancel buttons present; reason captured; Notify Provider optional.
- Within-24h cancel/reschedule applies a fee deducted from balance.
- Copay/deductible shown before confirmation; past notes never shown to patient.

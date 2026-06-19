# Telehealth — Provider Portal Feature Spec

## Purpose
- Provider-side telehealth (video) visits in Harmony EMR.
- Appointment-mode handling (In-Person ↔ Telehealth), video link delivery, and the telehealth join-link as the **virtual collection barrier** (replaces the front-desk that collects copays/balances in person).

## Actors / Roles
| Role | Capability |
|------|------------|
| Provider | Conducts the video visit; documents note (face-to-face start/end time) |
| Agent / Office Mgmt | Books/converts appointment mode; triggers link delivery |
| Biller | Configures Payment Policy (copay/balance threshold); processes link payments |
| Patient | Receives link via Email/SMS/Portal; pays copay/balance to join |
| Admin | Telehealth vendor config; appointment-type modes |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| TH-1 | Agent | to convert an appointment In-Person↔Telehealth | the visit modality can change without rebooking |
| TH-2 | Patient | to receive the video link via Email/SMS/Portal | I can join the visit |
| TH-3 | Biller | the join link to enforce copay/balance payment | telehealth patients can't dodge payment |
| TH-4 | Provider | the same provider→patient telehealth booking regardless of location | centralized scheduling isn't blocked by location mismatch |
| TH-5 | Agent | multi-stage reminder links (e.g., 3 days + 1–3 hrs before) | the patient is reminded and pays in time |

## Primary Workflow
1. Appointment created with **Mode = Telehealth** (3-mode form: In-Person / Telehealth / Block Time; Telehealth = In-Person form minus Location field).
2. Mode is convertible In-Person↔Telehealth on the appointment record (telehealth is an **appointment type/mode**, not a location attribute).
3. System sends the video join link per configured cadence (multi-stage; e.g., first link 3 days before, second 1–3 hrs before) via Email / SMS / Patient Portal.
4. Patient clicks link → **payment prompt** enforced before join:
   - "You have a copay of $X. Please make that payment to join the visit." (copay always upfront, separate from balance rule).
   - If accumulated balance > configured threshold (e.g., $250) → hard-block join until cleared.
   - Options at prompt: **Pay now** or **Enroll in a payment plan**.
5. On payment satisfied → patient joins video room; provider joins from portal.
6. Provider documents the visit; **actual face-to-face start/end time** captured on the note (module 10).
7. Group telehealth: same balance-threshold hard-stop applies — patient cannot join the group session over threshold.

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-16-07.png` / `…15-16-14.png` — scheduling 3-mode appointment form (Mode column In-Person / Telehealth).
- Telehealth join/payment-prompt screen (verify exact filename in the scheduling/portal capture set).

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Appointment | id, patientId, providerId, mode (In-Person/Telehealth/Block), appointmentType, serviceType, supervisorId, timeZone, location (in-person only) |
| TelehealthSession | appointmentId, vendor, meetingLink, joinTokens, status |
| LinkDelivery | appointmentId, channel (Email/SMS/Portal), stage, scheduledAt, sentAt, clickedAt |
| PaymentGate | appointmentId, copayDue, balanceDue, thresholdLimit, outcome (paid/plan/blocked) |
| PaymentPolicy (master + per-patient override) | advancePayment, balanceLimitThreshold, hardStop|followUp, manualLinkChannels |

## Business Rules
- Telehealth is an **appointment type/mode**, not a location attribute.
- **No location-based scheduling validation** — any patient can be booked with any provider via either modality; provider's home location is for reporting/billing aggregation only.
- Copay always upfront; must be paid before join, separate from accumulated-balance rule.
- Accumulated balance tolerated up to a configurable threshold; above it → hard-block join.
- Payment Policy is a master account-level rule with per-patient exception override.
- Insurance patients: no advance payment for most services (exception: neuropsych testing = upfront). Self-pay: payment before scheduling.
- Group telehealth: balance-threshold hard-stop applies to joining the group session.
- Link delivery cadence configurable; every link click re-runs the payment prompt.
- PHI: meeting links and sessions audit-logged; BAA/HIPAA vendor required.

## MoM Decisions / Deltas (authoritative)
- **Telehealth vendor = Google Meet** (NOT Zoom): locked at $22/user/month, 30-license minimum, 1-month trial post-contract, **BAA required**. (Technical-team call with Google to schedule.) *This overrides any "Zoom" reference; Zoom appears only as a deposition virtual-meeting example, not the clinical telehealth vendor.*
- **SMS via AWS SNS** ($0.01/SMS, HIPAA-compliant) — used for telehealth link/reminder delivery. (See module 17 for full SMS/email channel detail.)
- **Telehealth join link = virtual collection barrier** (replaces front-desk): copay prompt + balance hard-stop enforced on click. Options: pay now / enroll in payment plan.
- Copay always upfront; balance threshold configurable (example $250); master setting + per-patient override.
- Group appointments: same balance hard-stop on join.
- **No location gate** — provider may be booked with any patient via any modality; location = anchor for reporting/billing only (Jun 16).
- 3-mode appointment form (In-Person / Telehealth / Block Time); Telehealth = In-Person minus Location; Appointment Mode column (In-Person/Telehealth) added.
- Manual Payment Link sendable via Email / SMS / Patient Portal (all three confirmed).
- Multi-stage reminder cadence supported (e.g., 3 days + 1–3 hrs before).

## Dependencies
- Google Meet integration (video room, BAA).
- AWS SNS (SMS) + SendGrid (email) for link delivery (module 17).
- Scheduling / appointment types (module 02/16) for mode + cadence config.
- Billing Payment Policy + payment-link processing (module 12).
- Clinical documentation start/end face-to-face time (module 10).

## Open Items
- Google technical-team integration call (credential provisioning per provider) — to schedule.
- Confirm CPT-by-time billing source = note start/end vs scheduled time.
- Exact join-link payment-prompt screen filename.

## Acceptance Criteria
- [ ] Appointment mode can be set/converted In-Person↔Telehealth without rebooking.
- [ ] Video link is delivered via Email/SMS/Portal on the configured multi-stage cadence.
- [ ] Clicking the link enforces copay payment before join; balance over threshold hard-blocks.
- [ ] Prompt offers pay-now and enroll-in-plan options.
- [ ] No location mismatch blocks a telehealth booking.
- [ ] Group telehealth join is blocked when balance exceeds threshold.
- [ ] Provider joins from portal; note captures actual face-to-face start/end time.
- [ ] Sessions/links are audit-logged; vendor is BAA-covered (Google Meet).

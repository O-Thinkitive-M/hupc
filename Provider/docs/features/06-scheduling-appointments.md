# Scheduling & Appointments — Provider Portal Feature Spec

## Purpose
Manage provider availability, block days, appointment types, and bookings (in-person / telehealth / block-time). Provides filterable list + calendar views with utilization metrics, provider/biller views, encounter tab, and batch eligibility checks.

## Actors / Roles
| Role | Use |
|------|-----|
| Agent (Office) | Book/reschedule/cancel, run eligibility, manage block days |
| Provider | Provider view of own schedule, start encounters |
| Biller | Biller view (eligibility + balances), batch eligibility |
| Admin | Configure availability, appointment types |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| SCH-1 | Admin | per-provider availability with time zone | slots are accurate |
| SCH-2 | Agent | block days with conflict handling | provider time off is honored |
| SCH-3 | Agent | book in-person/telehealth/block-time | all visit modes are supported |
| SCH-4 | User | calendar views with utilization metrics | I see capacity at a glance |
| SCH-5 | Biller | batch eligibility for a date range | claims are clean upfront |
| SCH-6 | Provider | provider view + start button | I begin encounters quickly |

## Primary Workflow
1. Admin sets **Availability** per provider: Day / Start / End / Location / Time Zone → save shows "Apply these changes until [date]".
2. **Block Days**: Title / Start Date / End Date / Time Zone / Start / End → on conflict with scheduled patients: **cancel scheduled appts** OR **block slot + create task/triage**.
3. Agent opens **Scheduling** → left filters (Date, Provider, Locations, Appointment Type, Statuses).
4. Pick a calendar view (List / Day / Week / Month) → view utilization (Available/Booked/%).
5. **Schedule Appointment** form — choose mode (In-Person / Telehealth / Block Time) → fill → confirm.
6. Hover a slot → popup (demographics, mode, type, duration, provider, balance) + CTAs (Join, Reschedule, Cancel, Mark No-Show).
7. **Encounters** tab lists completed appointments; **List view** supports batch eligibility for a date range.

## Appointment Form — 3 Modes
| Mode | Key fields |
|------|-----------|
| In-Person | Patient (Active only), Time Zone, Appointment Type, Service Type, Provider, Supervisor, Location (auto), Slots, Duration (overridable), Repeat, Payment Method, **Discount (self-pay only)**, Set Payment Plan toggle, Reason, Notes, Assign Forms |
| Telehealth | Same as In-Person minus Location (Google Meet link) |
| Block Time | Title, Reason, Date, Start Time, End Time |

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-18-17.png` — Scheduling calendar with filters + utilization metrics.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-18-24.png` — Calendar view (Day/Week) + hover popup CTAs.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-18-47.png` — Schedule Appointment form (3-mode).
- `HUPC Screens Images/Screenshot from 2026-06-18 15-19-53.png` — Encounters / appointment detail.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-19-56.png` — Provider/Biller schedule view.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Availability | Provider, Day, Start, End, Location, TimeZone, ApplyUntil |
| Block Day | Title, StartDate, EndDate, TimeZone, Start, End |
| Appointment | Patient, Provider, Supervisor, Type, ServiceType, Mode, Location, DateTime, Duration, Status, Reason, Forms, Eligibility, Balance |
| Repeat | Count, Recurrence (daily/weekly/monthly), DaysFrom, Ends (Never/OnDate/AfterN) |

## Business Rules
- **Only Active patients** appear in the patient selector (inactive blocked).
- **Discount applies to Self-Pay patients only** (Dr. Mohammed).
- Time Zone required on availability + block days.
- Block-day conflicts must resolve via cancel-appts OR block+task.
- Statuses: Seeing, Waiting, Canceled, No Show (color-coded).
- PHI masked per role; reveal audit-logged.

## MoM Decisions / Deltas (authoritative)
- **Time Zone field** added to availability + block days; conflict resolution = cancel OR block+task (Jun 4 §3).
- **Appointment Mode** (In-Person/Telehealth) + **Discount** columns per appointment type; multiple CPT codes attachable (Jun 4 §4).
- **Inactive patients cannot be scheduled** — selector filters Active only (Jun 15 §13).
- **3-mode appointment form**; **Discount = self-pay only**; **inline Payment Plan** with auto-calc (Jun 15 §43).
- **4 calendar views** (List/Day/Week/Month) with utilization metrics; hover popup + CTAs (Jun 15 §41).
- **Provider View + Biller View** (Jun 15 §42); 2 tabs **Appointments / Encounters** (§40).
- Telehealth uses **Google Meet** (vendor locked); reminders via SMS (AWS SNS) / email (EnGard).

## Dependencies
- Patients (file 05) — Active filter, encounter tab, chart appointments.
- Settings (file 13) — availability, appointment types, locations, providers.
- Billing (file 10) — eligibility, payment method, discount, payment plan.
- Integrations (file 17) — Google Meet (telehealth), reminders.

## Open Items
- Landline-only OTP for patient self-booking (older patients) — unresolved.
- Pending-credentialing provider visibility in selector — HUPC confirmation pending.

## Acceptance Criteria
- Availability + block days require Time Zone; block conflict offers cancel vs block+task.
- Patient selector lists only Active patients.
- Appointment form supports 3 modes; Discount visible only for self-pay; payment plan auto-calculates.
- Calendar offers List/Day/Week/Month with utilization metrics and hover CTAs.
- List view runs batch eligibility for a date range.
- Provider and Biller views render their respective column sets.

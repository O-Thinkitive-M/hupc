# Appointments Configuration — Admin Portal Feature Spec

## Purpose
Configure scheduling foundations: per-provider **Availability**, **Holidays / Block Days**, and **Appointment Types** (mode, discount, CPT, attached forms) that drive the scheduling grid.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin / Office Manager | Configure availability, block days, appointment types |
| Provider | Has per-provider availability configured |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| AC-1 | admin | set per-provider availability (day/time/location/time zone) | schedules open correctly |
| AC-2 | admin | create Block Days with conflict handling | closures are respected |
| AC-3 | admin | define appointment types with mode, discount, CPT, forms | scheduling & billing align |
| AC-4 | admin | attach multiple CPT codes per appointment type | claims auto-populate |

## Primary Workflow
1. Admin opens **Appointment** settings (screen 15-38-49); selects a provider.
2. **Availability:** add rows of Day / Start Time / End Time / Location / **Time Zone**; save shows "Apply these changes until [date]".
3. **Block Days:** add Title / Start Date / End Date / Time Zone / Start Time / End Time.
4. On block conflict (patients already scheduled), choose **cancel appointments** OR **block slot + create task/triage** for provider/office management.
5. **Appointment Types:** add Service Type / Appointment Type / Duration / Appointment Name / **Mode (In-Person/Telehealth)** / **CPT codes** / **Discount** / **Attached Forms**.
6. Save; scheduling grid + fee schedule consume the config; changes audit-logged.

## Screens
- Appointment: `Screenshot from 2026-06-18 15-38-49.png`

## Data Entities
| Entity | Key Fields |
|--------|------------|
| Availability | providerId, day, startTime, endTime, locationId, **timeZone**, applyUntilDate |
| BlockDay | title, startDate, endDate, **timeZone**, startTime, endTime |
| AppointmentType | id, serviceType, name, duration, **mode**, cptCodes[], **discount**, attachedForms[] |

## Business Rules
- **Time Zone** field on both availability and block days [MoM §3].
- Block-day **conflict resolution: cancel OR block + task/triage** [MoM §3].
- Appointment Mode: **In-Person / Telehealth** [MoM §4].
- **Discount per appointment type** [MoM §4].
- **Multiple CPT codes attachable** per appointment type; add-on/variant CPTs auto-attach to the note [MoM §4; SRS §3.x].
- Holiday/out-of-office templates **hide affected dates from scheduling** and exclude from occupancy denominator (SRS §3455/§4341).
- Grace/cut-off period configurable per appointment type (group therapy = 15 min) (SRS §15).

## MoM Decisions / Deltas (authoritative)
- [x] Time Zone added to availability + block days [MoM §3].
- [x] Block Days conflict resolution: cancel or block + task [MoM §3].
- [x] Appointment Types: **Mode** (In-Person/Telehealth) + **Discount** columns [MoM §4].
- [x] Multiple CPT codes attachable per appointment type [MoM §4].
- [x] Add Appointment form fields: Service Type / Appointment Type / Duration / Name / Mode / CPT / Discount / Forms [MoM §4].

## Dependencies
- Module 07 (CPT codes) · Module 03 (locations/holidays, time zones) · Module 08 (attached forms/templates) · Module 10 (self-pay fee by appointment/service type) · Module 12 (conflict tasks → alerts/triage).

## Open Items
- "Apply these changes until [date]" recurrence semantics need detail.
- Whether discount is % or flat amount.

## Acceptance Criteria
- [ ] Per-provider availability captured with time zone + apply-until.
- [ ] Block Days with conflict handling (cancel vs block+task).
- [ ] Appointment Types carry mode, discount, multiple CPTs, attached forms.
- [ ] Holidays/block days hidden from scheduling and excluded from occupancy.
- [ ] Changes audit-logged.

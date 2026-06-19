# Clinic Management — Admin Portal Feature Spec

## Purpose
Maintain the clinic's identity, its practice locations, and the location hierarchy that scopes scheduling, billing, and reporting across the EMR.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin | Create/edit clinic profile and locations |
| Office Manager | Maintain location details, hours, holidays |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| CM-1 | admin | edit the clinic profile (name, NPI, Tax ID, address, logo) | identity is correct on claims/docs |
| CM-2 | admin | add/edit/disable practice locations | multi-site operations are supported |
| CM-3 | admin | configure per-location hours and holiday/closure dates | scheduling reflects reality |
| CM-4 | admin | organize locations into a hierarchy | reporting and visibility roll up |

## Primary Workflow
1. Admin opens **Clinic Profile** (screen 15-39-07).
2. Edits clinic-level fields: Clinic Name, Group NPI, Tax ID, primary address, contact, logo.
3. Opens Locations → adds a location (name, address, contact, time zone, hours).
4. Sets per-location holiday/closure dates (feeds module 09).
5. Assigns location to hierarchy parent; saves.
6. Changes audit-logged; downstream config (availability, fee schedule by location) picks up new location.

## Screens
- Clinic Profile: `Screenshot from 2026-06-18 15-39-07.png`

## Data Entities
| Entity | Key Fields |
|--------|------------|
| Clinic | clinicId, name, groupNpi, taxId, address, contact, logo |
| Location | locationId, clinicId, name, address, timeZone, hours, status, parentLocationId |
| LocationHoliday | locationId, date, reason |

## Business Rules
- Clinic Tax ID / Group NPI flow into claims (billing config and Provider Master Group NPI).
- **Holiday/closure dates configurable per location** and excluded from scheduling/occupancy (SRS §4.x).
- Disabling a location must not orphan active schedules — warn/migrate.
- All directory changes are audit-logged (SRS §directory changes).

## MoM Decisions / Deltas (authoritative)
- No explicit clinic-profile delta in MoM V2; **Practice Location** is referenced as a Provider Master field [MoM §5] and **Time Zone** is added to availability/block days [MoM §3] — locations must carry a time zone.
- Clinic Tax ID + Group NPI reaffirmed as captured at provider enrollment parity [MoM §7].

## Dependencies
- Module 04 (Provider Master "Practice Location") · Module 09 (per-location availability/holidays) · Module 10 (fee schedule by location/provider type) · Module 06 (departments per location).

## Open Items
- Depth/shape of location hierarchy (region → site → suite?) not specified.
- Whether one tenant can hold multiple legal clinics (multi-NPI) or one.

## Acceptance Criteria
- [ ] Admin can edit clinic profile incl. Group NPI / Tax ID / logo.
- [ ] Admin can add/edit/disable locations with time zone + hours.
- [ ] Per-location holidays exclude dates from scheduling.
- [ ] Location hierarchy assignment saved and used in reporting/visibility.
- [ ] All changes audit-logged.

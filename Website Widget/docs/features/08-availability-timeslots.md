# Availability & Time Slots — Website Widget Feature Spec

## Purpose
- Define **Step 7**: render one-week provider availability (sourced from the Scheduling availability engine), apply location-based display, closures and working hours, real-time updates, and time-slot selection.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Browses availability, selects a slot |
| Scheduling availability engine | Source of truth for slots |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| AV-1 | Prospect | a week of open slots | I can pick a convenient time |
| AV-2 | Prospect | location labels on slots | I know where I'm going |
| AV-3 | Prospect | closed dates marked | I don't pick an unavailable day |
| AV-4 | Prospect | accurate (not stale) availability | my pick is actually open |

## Primary Workflow
1. After provider selection, widget displays **one full week** of available slots by default for the chosen mode + service.
2. Patient can **scroll forward** to additional weeks.
3. Slots shown as date + time, MM/DD/YYYY + 12-hour (EST).
4. Only slots matching service type, mode, and provider appear; booked/blocked/admin-time excluded.
5. Location-based display: each day labels the working location (multi-location providers).
6. Closed dates greyed/"Closed"; in-person blocked, telehealth still bookable if offered that day.
7. Real-time: availability queried on view; if a slot is taken between view and submit, error + re-fetch.
8. Patient selects a slot → payment & confirmation (file 12).

## Display Rules
| Aspect | Rule |
|--------|------|
| Default range | One full week; scroll forward for more |
| Slot format | MM/DD/YYYY, 12-hour, EST |
| Excluded | Booked, blocked, admin-time slots |
| Location label | Per day, e.g., "Tuesday — Tampa" vs "Wednesday — Ocala" |
| Closed date | Greyed/"Closed"; hover/tap shows "This location is not available to accept appointments on [date] due to [reason]" |
| Closure reason | From availability-engine metadata (holiday, weather, maintenance) |
| Telehealth on closed day | Remains visible/bookable if provider offers telehealth that day |
| Working hours | Location page shows standard hours; closed days shown (not hidden) |

## Real-Time Behavior
- Optimistic UI: slots render from a snapshot on load.
- Server validation at submission: if slot taken between view and submit, widget shows error and re-fetches.
- No stale data held beyond the active browsing session.

## Screens
- No screen capture provided; per SRS/MoM, expected UI: weekly calendar/grid with per-day location labels, greyed closed dates with reason tooltips, slot chips, forward-scroll control.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Availability slot | Date, time, provider, mode, location, status |
| Closure | Date, location, reason (metadata) |
| Location | Working hours, telehealth flag |

## Business Rules
- Widget does not compute availability; it renders what the Scheduling availability engine returns.
- Only slots matching the patient's service/mode/provider/location are shown.
- Closed in-person dates are non-selectable; telehealth may remain bookable.
- Slot must still be open at submit; otherwise re-fetch.

## MoM Decisions / Deltas (authoritative)
- (Settings demo, Jun) Provider availability config adds **Time Zone** field alongside Day / Start / End / Location; **Block Days** (Title / Start / End / Time Zone / Start / End) drive closures.
- Block-day conflict when patients already scheduled → staff option to cancel scheduled appts OR block + create task (affects which slots the engine exposes to the widget).
- (Master Finalization) Slot selection flow: provider → date → time slot → **Confirm Booking**.
- Office Locator (ZIP/city, distance-sorted) feeds the location filter that scopes availability.

## Dependencies
- Scheduling availability engine (Scheduling SRS §7).
- Provider Selection (file 07); Office Locator (location context).
- Payment & Confirmation (file 12).

## Open Items
- Telehealth-flag-by-location behavior on closed days — confirm per-location config.

## Acceptance Criteria
- One-week availability renders by default with forward scroll.
- Booked/blocked/admin slots are excluded.
- Multi-location days are labeled; closed dates are marked with reason and non-selectable for in-person.
- A slot taken concurrently produces an error + re-fetch at submit.

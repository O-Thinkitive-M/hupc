# Alerts — Admin Portal Feature Spec

## Purpose
New Settings section to define operational/clinical **Alerts** — what triggers them, when, their priority, who they go to, and their status — so the practice can automate notifications and task routing.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin / Office Manager | Define and maintain alert rules |
| Assigned user/role | Receives and acts on alerts |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| AL-1 | admin | create an alert with title, type, trigger event & timing | events notify the right people |
| AL-2 | admin | set alert priority and assignee | urgency and ownership are clear |
| AL-3 | admin | enable/disable (status) an alert | rules are managed over time |
| AL-4 | admin | add alerts inline from the Alerts table | configuration is fast |

## Primary Workflow
1. Admin opens **Alerts** (screen 15-39-25) under Settings.
2. Clicks **Add Alert** (inline).
3. Fills: **Alert Title / Alert Type / Trigger Event / Trigger Timing / Priority / Assigned To / Status**.
4. Saves; the alert engine fires when the trigger event/timing condition is met.
5. Alerts route to the assigned user/role/department and surface on the dashboard.

## Screens
- Alerts: `Screenshot from 2026-06-18 15-39-25.png`

## Data Entities
| Entity | Key Fields |
|--------|------------|
| Alert | alertId, **title**, **type**, **triggerEvent**, **triggerTiming**, **priority**, **assignedTo**, **status** |
| AlertInstance | alertId, firedAt, context, assignee, state (open/ack/done) |

## Business Rules
- Alerts table fields (authoritative): **Alert Title / Alert Type / Trigger Event / Trigger Timing / Priority / Assigned To / Status** [MoM §15].
- **Add Alerts inline** [MoM §15].
- Block-day/scheduling conflicts can generate a task/triage routed to provider/office management (MoM §3) — overlaps with alert/task routing.
- Reschedule/Cancel alert example: 24-hour window cancellation rule (SRS §573) is a candidate trigger event.

## MoM Decisions / Deltas (authoritative)
- [x] **Alerts module added to Settings** with the listed fields [MoM §15].
- [x] Add Alerts capability inline [MoM §15].
- This is a net-new Settings section vs SRS [MoM §15].

## Dependencies
- Module 14 (Settings hub) · Module 02 (dashboard surfaces alerts) · Module 06 (assigned-to department/role) · Module 05 (assignee roles) · Module 13 (alert actions audit-logged).

## Open Items
- Enumerate Alert Type and Trigger Event value sets.
- Trigger Timing model (before/after event, offset, recurring) needs detail.
- Delivery channels (in-app, email, SMS) not specified.

## Acceptance Criteria
- [ ] Admin can add/edit an alert with all seven fields.
- [ ] Inline add works from the Alerts table.
- [ ] Alerts fire on the configured trigger event/timing.
- [ ] Alerts route to the assigned user/role and appear on the dashboard.
- [ ] Alert configuration changes are audit-logged.

# Audit Log — Admin Portal Feature Spec

## Purpose
Provide a filterable, read-only audit-log viewer capturing every security-relevant action — config changes, PHI reveals, deletions, overrides — for HIPAA accountability and investigation.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin / Compliance / Security | View, filter, and export audit entries |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| AU-1 | compliance | filter the log by user, role, event, category, patient MRN, outcome, date | I investigate incidents |
| AU-2 | compliance | view PHI-reveal events | minimum-necessary is enforced |
| AU-3 | security | see 403/unauthorized-access entries | misuse is detected |
| AU-4 | compliance | export filtered audit results | I produce evidence |

## Primary Workflow
1. Admin opens **Audit Log** (screen 15-39-02).
2. Applies filters: **Timestamp / User / Role / Event / Category / Patient MRN / Outcome**.
3. Reviews entries (read-only; entries are immutable).
4. Drills into an entry for full context.
5. Optionally exports the filtered result (export itself is audit-logged).

## Screens
- Audit Log: `Screenshot from 2026-06-18 15-39-02.png`

## Data Entities
| Entity | Key Fields |
|--------|------------|
| AuditEntry | timestamp, userId, role, event, category, patientMrn?, outcome, context, sessionId |

## Business Rules
- Logged actions include (non-exhaustive): config changes, **PHI reveal** (module 11), appointment/record **deletion** (user, timestamp, details, reason — SRS §14.3), **out-of-network override** (SRS §2526), duplicate-detection at lead entry (SRS §2.9), directory/role changes, login events, matrix/data exports (SRS §4496).
- Entries are **immutable / append-only**.
- Patient MRN populated where the event concerns a patient.
- Outcome captures success/failure (e.g., 403 unauthorized → audit entry, SRS §1888).
- Access to the audit log is itself permission-gated and logged.

## MoM Decisions / Deltas (authoritative)
- No standalone MoM audit section, but audit logging is reinforced across MoM decisions:
  - [x] PHI **reveal** action audit-logged [MoM §11].
  - [x] **Allowed Amount** changes audit-logged [MoM §13].
  - [x] Fee/config changes generally audit-logged [MoM §12/§13].
- Filter set (Timestamp/User/Role/Event/Category/MRN/Outcome) per screen 15-39-02.

## Dependencies
- Module 11 (reveal events) · Module 10 (billing-config changes) · Module 05 (overrides, role changes) · Module 04 (user/directory changes) · all modules emit events.

## Open Items
- Canonical Event and Category taxonomies need enumeration.
- Retention period and export format (CSV/PDF) TBD.
- Whether audit log is searchable across all portals from this Admin viewer.

## Acceptance Criteria
- [ ] Viewer filters by timestamp, user, role, event, category, patient MRN, outcome.
- [ ] Entries are read-only/immutable.
- [ ] PHI reveals, deletions, overrides, and 403s appear in the log.
- [ ] Filtered results can be exported, and the export is itself logged.

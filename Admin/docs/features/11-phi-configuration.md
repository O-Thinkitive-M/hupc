# PHI Configuration — Admin Portal Feature Spec

## Purpose
Configure per-role masking of sensitive patient demographics (Phone, Email, SSN) with reveal-by-click and full audit logging — added per Dr. Mohammed's masking requirement.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin / Compliance Admin | Define per-role masking rules |
| All EMR users | Subject to masking; reveal-by-click (logged) |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| PH-1 | compliance admin | configure which demographics are masked per role | HIPAA minimum-necessary is enforced |
| PH-2 | user | reveal a masked field by click when needed | I can do my job |
| PH-3 | compliance | have every reveal audit-logged | access is accountable |

## Primary Workflow
1. Admin opens **PHI Configuration** table (under Settings).
2. For each role type, sets masking on **Phone Number / Email / SSN** (masked / visible).
3. Saves; rules apply across all PHI-displaying screens.
4. At runtime, a user with a masked field sees it obscured; clicking **reveals** the value.
5. Each reveal writes an audit entry (user, timestamp, patient context) — see module 13.

## Screens
- No dedicated screenshot; PHI Configuration table lives under the Settings hub (`15-21-21`).

## Data Entities
| Entity | Key Fields |
|--------|------------|
| PhiMaskingRule | roleId, demographic (Phone\|Email\|SSN), masked (bool) |
| PhiRevealEvent | userId, patientId/MRN, field, timestamp, context |

## Business Rules
- Masking is **scoped per role type** [MoM §11].
- In-scope demographics: **Phone / Email / SSN** [MoM §11].
- **Reveal-by-click**; each reveal is **audit-logged** (user, timestamp, patient context) [MoM §11; SRS §23307].
- Phone and address masking by default with reveal-and-log (SRS §26530) — at minimum Phone/Email/SSN per MoM scope.
- Masking applies everywhere PHI is displayed (lists, detail, exports).

## MoM Decisions / Deltas (authoritative)
- [x] **PHI Configuration table with per-role masking** [MoM §11].
- [x] Demographics in scope: **Phone / Email / SSN** [MoM §11].
- [x] **Reveal action is audit-logged** [MoM §11].
- This module is net-new vs SRS, added per Dr. Mohammed's ask [MoM §11].

## Dependencies
- Module 05 (roles) · Module 13 (audit log of reveals) · Every PHI-displaying screen across portals (consumers).

## Open Items
- Whether address is also in scope (SRS suggests address masking; MoM lists Phone/Email/SSN).
- Reveal duration (single view vs session) and re-mask behavior.
- Export/print masking behavior.

## Acceptance Criteria
- [ ] Admin configures masking of Phone/Email/SSN per role.
- [ ] Masked fields display obscured for affected roles.
- [ ] Reveal-by-click shows the value and writes an audit entry.
- [ ] Masking applies consistently across lists, detail, and exports.

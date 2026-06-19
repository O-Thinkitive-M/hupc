# Departments — Admin Portal Feature Spec

## Purpose
Maintain the clinic's departments so users/providers can be assigned to a department for routing, visibility scoping, task assignment, and reporting.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin / Office Manager | Create/edit/disable departments |
| Admin | Assign users to departments (module 04) |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| DP-1 | admin | add/edit/disable a department | org structure is current |
| DP-2 | admin | assign a department to a user/provider | routing & visibility scope |
| DP-3 | admin | scope cross-department visibility | data access is controlled |

## Primary Workflow
1. Admin opens Departments (under Settings/Master area).
2. Adds a department (name, description, optional location).
3. Department becomes selectable on the Users / Provider Master form.
4. Tasks/triage and visibility scope can reference department.
5. Disable to retire; changes audit-logged.

## Screens
- No dedicated screenshot; managed within Settings hub (`15-21-21`) / Master (`15-39-21`).

## Data Entities
| Entity | Key Fields |
|--------|------------|
| Department | departmentId, name, description, locationId?, status |
| UserDepartment | userId, departmentId |

## Business Rules
- **Department field is mandatory on the Users table** (added per MoM — previously missing) [MoM §5].
- Cross-department visibility is limited unless a cross-functional team grants access (SRS §1877).
- Block-day/scheduling conflict tasks may route to "provider or office management" — department aids routing (SRS §, MoM §3).

## MoM Decisions / Deltas (authoritative)
- [x] **Department added to Users table** [MoM §5] — Departments must exist as managed master data to populate that dropdown.
- Departments are a distinct Admin module (**32 FE hours**) [FE estimation].

## Dependencies
- Module 04 (user/provider assignment) · Module 03 (optional per-location departments) · Module 12 (alert/task routing by department).

## Open Items
- Whether departments are global or per-location.
- Whether department drives any default permission scoping vs purely organizational.

## Acceptance Criteria
- [ ] Admin can add/edit/disable departments.
- [ ] Departments populate the Users/Provider Master department dropdown.
- [ ] Disabling a department does not break existing user assignments.
- [ ] Changes audit-logged.

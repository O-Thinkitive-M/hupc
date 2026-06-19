# Settings Hub — Admin Portal Feature Spec

## Purpose
The central configuration hub for the Admin portal: a navigation surface to all config sections plus global settings including **Security Questions** for account recovery.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin | Access all settings; configure global options |
| Office Manager | Access permitted settings sections |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| ST-1 | admin | navigate to all config sections from one hub | configuration is discoverable |
| ST-2 | admin | manage the Security Questions catalog | account recovery works |
| ST-3 | user | set my security-question answers | I can recover my account |
| ST-4 | admin | configure global/system options | clinic-wide defaults are set |

## Primary Workflow
1. Admin opens the **Settings** hub (screen 15-21-21).
2. Hub lists sections: Appointment, Clinic, Billing, Template, Master, Alerts, Audit Log, Users, Roles, PHI Configuration, Departments, Security Questions.
3. Admin selects a section → routes to the owning module (files 03–13).
4. **Security Questions:** admin maintains the question catalog; users select questions and store hashed answers (used at first login and recovery).
5. Global options (locale/i18n defaults, system preferences) configured here.

## Screens
- Settings hub: `Screenshot from 2026-06-18 15-21-21.png`
- Sub-screens routed from hub: Appointment `15-38-49`, Audit Log `15-39-02`, Clinic `15-39-07`, Billing `15-39-12`, Template `15-39-16`, Master `15-39-21`, Alerts `15-39-25`.

## Data Entities
| Entity | Key Fields |
|--------|------------|
| SecurityQuestion | questionId, text, active |
| SecurityQuestionAnswer | userId, questionId, answerHash |
| GlobalSetting | key, value, scope |

## Business Rules
- Settings hub is the single entry point to every Admin config module (RBAC filters which sections appear).
- **Security Questions** are mandatory at first login (module 01) and used for recovery; answers stored hashed.
- Global i18n/locale defaults set here support the multi-language requirement (provider Preferred Language at module 04).
- Section visibility honors role permissions.

## MoM Decisions / Deltas (authoritative)
- MoM V2 expands Settings with net-new sections that must appear in this hub:
  - [x] **Alerts** section [MoM §15].
  - [x] **PHI Configuration** [MoM §11].
  - [x] **Availability + Block Days** with time zone [MoM §3].
  - [x] **Appointment Types** with mode/discount/CPT/forms [MoM §4].
  - [x] **Provider Master** under Users [MoM §5/§7].
  - [x] **Allowed Amount** tab under Billing [MoM §13].
- Settings is a distinct Admin module (**64 FE hours**) [FE estimation].

## Dependencies
- All modules 03–13 (hub routes to them) · Module 01 (security questions at onboarding) · RBAC.

## Open Items
- Final ordering/grouping of hub sections.
- Global-settings catalog (locales, system defaults) not fully enumerated.
- Number of security questions a user must answer.

## Acceptance Criteria
- [ ] Settings hub links to all config sections, filtered by role.
- [ ] Security Questions catalog is manageable; user answers stored hashed.
- [ ] New MoM sections (Alerts, PHI Config) appear in the hub.
- [ ] Global/locale defaults configurable.

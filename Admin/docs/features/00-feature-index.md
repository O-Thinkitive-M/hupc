# Admin Portal — Feature Index

HUPC EMR / Harmony EMR (Medent → custom-built migration by Thinkitive). This index maps the Admin-portal feature specs, defines shared glossary/roles, and documents cross-cutting concerns referenced by every module.

> **Authority order:** **QA MoM V2 OVERRIDES the SRS.** Where the two conflict, the MoM decision wins. Each spec carries a "MoM Decisions / Deltas (authoritative)" section.

## Module Map

| # | File | Module | Primary Screen(s) |
|---|------|--------|-------------------|
| 00 | `00-feature-index.md` | Index / glossary / cross-cutting | 15-21-21 |
| 01 | `01-admin-onboarding.md` | Admin onboarding / first-login | — |
| 02 | `02-dashboard.md` | Admin KPIs / analytics | — |
| 03 | `03-clinic-management.md` | Clinic profile, locations, hierarchy | 15-39-07 |
| 04 | `04-users.md` | Provider / office-staff / agent users, Provider Master | 15-39-21 (Master), Users |
| 05 | `05-roles-credentialing.md` | Role setup, copy-permissions, intern role, network status, credentialing | — |
| 06 | `06-departments.md` | Departments | — |
| 07 | `07-master-data.md` | Data Import, ICD-10, CPT, Payer Master | 15-39-21 |
| 08 | `08-templates.md` | Visit Note templates, Macros | 15-39-16 |
| 09 | `09-appointments-config.md` | Availability, Holidays/Block Days, Appointment Types | 15-38-49 |
| 10 | `10-billing-config.md` | Fee Schedule, Allowed Amount, Legal/Other Fee, Payment Policy | 15-39-12 |
| 11 | `11-phi-configuration.md` | Demographics masking per role | — |
| 12 | `12-alerts.md` | Alert type / trigger / priority / assigned-to / status | 15-39-25 |
| 13 | `13-audit-log.md` | Audit-log filters & viewer | 15-39-02 |
| 14 | `14-settings.md` | Security Questions + global config hub | 15-21-21 |

## Glossary

| Term | Meaning |
|------|---------|
| **Medent** | Legacy EMR being migrated from; Provider Master mirrors Medent's `new_provider` enrollment form |
| **Harmony EMR** | Target custom-built EMR (HUPC) |
| **Provider Master** | Settings record per provider (photo, credentials, type, licenses, taxonomy, NPI) |
| **Provider Type** | Medication / Therapy / Counseling — drives specialty bucket |
| **Specialty Bucket** | Two only: **Psychiatric** (MD/APRN med-management) and **Therapy/Counseling** (therapists/PhD-Psy) |
| **Network Status** | Color-coded, future-dated payer-network state of a provider; drives Rendering vs Supervising claim submission |
| **Taxonomy Number** | NUCC provider taxonomy code required at EHR enrollment |
| **Allowed Amount** | Contracted rate per CPT + Payer + Provider Type, with effective dates |
| **Self-Pay Amount** | Charge based on Service/Appointment Type (new/follow-up/testing), distinct from CPT-based Insurance Charge |
| **PHI Config** | Per-role masking rules for Phone/Email/SSN demographics |
| **MRN** | Medical Record Number — patient identifier surfaced in audit log |
| **TFL** | Timely Filing Limit (billing/AR) |
| **EPCS** | Electronic Prescribing of Controlled Substances |

## Roles (RBAC)

| Role | Scope |
|------|-------|
| **Super Admin** | Full configuration: clinic, users, roles, master data, billing config, PHI config, alerts |
| **Admin / Office Manager** | Day-to-day config, user/role management, scheduling config |
| **Provider** | Clinical user (MD / APRN / PA / Therapist / PhD-Psy) |
| **Office Staff** | Scheduling, front-desk, intake |
| **Billing Agent** | Claims, AR, payments |
| **Intern** | Restricted role; supervised; permissions copied from a base role then narrowed |

> **Users = anyone using the EMR** (providers, office assistants, billing agents, employees). [MoM §6]

## Cross-Cutting Concerns

| Concern | Rule |
|---------|------|
| **RBAC** | Permissions are role-based; new roles may **copy permissions from an existing role** then be edited. Direct-URL access to disallowed resources → `403` + audit-log entry. |
| **PHI Masking** | Phone/Email/SSN masked by default per role; **reveal-by-click** logs an audit entry (user, timestamp, patient context). Configured in module 11. |
| **Audit Log** | Every config change, masking reveal, deletion, and out-of-network override is logged: timestamp, user, role, event, category, patient MRN (where applicable), outcome. Viewer in module 13. |
| **Master Data** | ICD-10, CPT, Payer, Departments are shared catalogs consumed by appointments, billing, templates, and clinical modules. Codes entered via dropdown only (no free text). |
| **i18n** | Provider profile carries Preferred Language; multi-language UI support is a global requirement (see Settings hub). |
| **Effective Dating** | Network Status, Allowed Amount, and Fee Schedule entries are date-ranged (From/End). |

## Spec Template (used by all files)
`## Purpose` · `## Actors / Roles` · `## Key User Stories` · `## Primary Workflow` · `## Screens` · `## Data Entities` · `## Business Rules` · `## MoM Decisions / Deltas (authoritative)` · `## Dependencies` · `## Open Items` · `## Acceptance Criteria`

## Source Documents
- SRS: `HUPC__SRS_Document.pdf` (712p)
- **QA MoM V2 (authoritative):** `HUPC QA MoM V2.pdf` (121p)
- FE estimation: `FE_Estimation_Summary.md` (Admin = 968 FE hours)
- Screens: `HUPC Screens Images/` (settings hub 15-21-21; sub-screens 15-38-49 … 15-39-25)

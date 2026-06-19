# Users & Provider Master — Admin Portal Feature Spec

## Purpose
Define and manage every person who uses the EMR (providers, office staff, billing agents, employees) and capture rich provider profiles via the **Provider Master**, mirroring Medent's provider-enrollment form.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin / Office Manager | Add/edit/disable users, assign roles & departments |
| Admin | Maintain Provider Master profiles, licenses, preferences |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| US-1 | admin | add any EMR user (provider/staff/agent) | everyone has access |
| US-2 | admin | assign role + department + practice location to a user | RBAC & scoping work |
| US-3 | admin | create a full Provider Master profile (photo, credentials, NPI, taxonomy) | claims & scheduling are accurate |
| US-4 | admin | capture per-state license IDs with expiry | multi-state compliance |
| US-5 | admin | record provider preferences (insurance/language/age group/diagnosis specialties) | matching & routing work |
| US-6 | admin | enable/disable a user | offboarding is clean |

## Primary Workflow
1. Admin opens Users (Master area, screen 15-39-21) → Add User.
2. Enters identity, work/personal email, **Department**, role.
3. If provider, opens **Provider Master** form and fills all enrollment-parity fields.
4. Adds State Licenses (State + License ID + Expiry) — one per state.
5. Adds Provider Preference data (age criteria, diagnosis/testing specialties).
6. Saves; user receives activation; change audit-logged.

## Screens
- Master / Users: `Screenshot from 2026-06-18 15-39-21.png`

## Data Entities — Provider Master
| Field group | Fields |
|-------------|--------|
| Identity | First / Last / Middle Initial, Profile Photo, Credentials (MD/NP/PhD) |
| Type | Provider Type (Medication / Therapy / Counseling) → Specialty Bucket |
| Contact | Personal Email, Work Email, Mobile, Contact #, Fax |
| Org | Department, Practice Location |
| IDs | Taxonomy Number, Provider NPI, Group NPI (HUPC), Tax ID (HUPC), Provider License Number |
| Address | Business Address / City / State / ZIP |
| Capabilities | Eligible Appointment Types, Direct Messaging, EPCS (med providers) |
| Preferences | Preferred Insurance, Preferred Language, Preferred Age Group, Diagnosis Specialties, Therapy/Counseling diagnosis, Testing diagnosis |
| Licenses | StateLicense[]: State, License ID, Expiry Date |

## Business Rules
- **Users = everyone using the EMR** [MoM §6].
- Provider Master **mirrors Medent `new_provider` form** minus Medent-specific account number [MoM §7].
- **Taxonomy number required** at enrollment [MoM §7].
- Provider Type drives the two specialty buckets (see module 05).
- Department is mandatory on the Users table (was previously missing) [MoM §5].
- EPCS/Direct Messaging apply to medication providers.

## MoM Decisions / Deltas (authoritative)
- [x] Photo / NPI / Tax ID / Group NPI fields confirmed [MoM §5/§7].
- [x] **Department** field added to Users table [MoM §5].
- [x] Provider Type dropdown: medication / therapy / counseling [MoM §5].
- [x] **Different License ID per State** supported [MoM §5].
- [x] Provider Master mirrors Medent enrollment form (minus account number) [MoM §7].
- [x] Provider Preference Form (age criteria + diagnosis specialty + testing categories) drives the data model [MoM §7].
- [x] Photo / Taxonomy / NPI / Tax ID / Group NPI all captured [MoM consolidated].

## Dependencies
- Module 05 (roles, specialty buckets, network status, credentialing) · Module 06 (departments) · Module 03 (practice location) · Module 09 (eligible appointment types) · Module 07 (payer for preferred insurance).

## Open Items
- Exact required-field set vs Medent form (Lucas to forward both Medent enrollment + Provider Preference PDFs).
- Document-type taxonomy for uploads (photo, license, etc.) [MoM Open Items].

## Acceptance Criteria
- [ ] Any user type can be created with role + department.
- [ ] Provider Master captures all enrollment-parity fields incl. taxonomy, photo, NPIs.
- [ ] Multiple state licenses (State + ID + Expiry) supported.
- [ ] Provider Type sets specialty bucket.
- [ ] Provider preferences captured and queryable.
- [ ] All user/profile changes audit-logged.

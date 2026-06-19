# Admin Onboarding — Admin Portal Feature Spec

## Purpose
First-time setup and authentication of an administrator/clinic, enabling access to the configuration portal so the clinic, users, master data, and billing can be configured before go-live.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin | Receives initial credentials; completes first-login setup |
| Implementation team (Thinkitive) | Provisions tenant/clinic, seeds initial Super Admin |
| Office Manager / Admin | Continues configuration after handoff |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| AO-1 | new admin | activate my account via invite link | I can log in securely |
| AO-2 | admin | set a password and security questions on first login | my account is recoverable |
| AO-3 | admin | be guided through a setup checklist (clinic → locations → users → master data) | go-live config is complete |
| AO-4 | admin | enable MFA | the portal meets HIPAA access controls |

## Primary Workflow
1. Implementation team creates the clinic tenant and seeds a Super Admin with a temporary credential.
2. Admin receives an invite/activation email with a one-time link.
3. On first login the admin sets a password and configures **Security Questions** (see module 14).
4. Optional MFA enrollment.
5. Setup checklist routes the admin through: Clinic Profile → Locations/Hierarchy → Departments → Users/Provider Master → Roles → Master Data → Appointment Config → Billing Config → PHI Config → Alerts.
6. Admin marks setup complete; portal unlocks full navigation.

## Screens
- Login / activation (settings hub entry: `Screenshot from 2026-06-18 15-21-21.png`)
- First-login security-question setup (see module 14 screen 15-21-21)

## Data Entities
| Entity | Key Fields |
|--------|------------|
| AdminAccount | userId, email, status (invited/active/locked), mfaEnabled |
| ActivationToken | token, expiry, consumedAt |
| SetupChecklist | clinicId, step, completedAt |
| SecurityQuestionAnswer | userId, questionId, answerHash |

## Business Rules
- Activation links are single-use and expire.
- Password complexity and lockout-on-failed-login enforced (auto-block on repeated failures — security checklist).
- Security questions are mandatory at first login.
- All onboarding/auth events are audit-logged.

## MoM Decisions / Deltas (authoritative)
- No dedicated onboarding deltas in MoM V2; onboarding is the lightest Admin module (**12 FE hours**, FE estimation).
- Login identity model carried to **patient** portal is contested (email-as-unique-id vs Medent-style username) — **admin/staff** login uses standard email-based accounts; the username debate is patient-only [MoM Open Items].

## Dependencies
- Module 14 (Security Questions) · Module 03 (Clinic Profile) · Module 04 (Users) · Auth/security infrastructure.

## Open Items
- MFA method (TOTP vs SMS) not finalized.
- Whether a self-service clinic sign-up exists, or all tenants are provisioned by implementation team.

## Acceptance Criteria
- [ ] Admin can activate via invite link and is forced to set password + security questions.
- [ ] Expired/used activation links are rejected.
- [ ] Setup checklist tracks completion across all config modules.
- [ ] All login/onboarding events appear in the audit log.

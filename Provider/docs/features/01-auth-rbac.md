# Authentication & RBAC — Provider Portal Feature Spec

## Purpose
Secure provider-portal sign-in, session control, role-based access, and PHI masking configuration for the Harmony EMR. Replaces Medent username login with email/phone identity and enforces per-role data masking.

## Actors / Roles
| Role | Auth scope |
|------|-----------|
| Admin | Full access incl. Settings → Users/Roles/PHI Config |
| Provider | Clinical modules per assigned permissions |
| Agent (Office) | Scheduling, leads, registration, fax/request triage |
| Biller | Billing, claims, AR, payments |
| Guardian | Patient-portal proxy (file 14) — not a provider-portal login |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| AUTH-1 | User | log in with email or phone + password | I don't memorize a custom username |
| AUTH-2 | User | receive a 2FA OTP on my user-ID credential | my access is secured |
| AUTH-3 | User | be logged out after inactivity | PHI is protected on unattended sessions |
| AUTH-4 | Admin | define roles and copy permissions from an existing role | role setup is fast |
| AUTH-5 | Admin | configure which demographics are masked per role | PHI exposure is minimized |
| AUTH-6 | User | reveal a masked field by click | I can read PHI when needed, with audit |

## Primary Workflow
1. User enters **email OR phone** + password (no username field anywhere).
2. System validates against unique-constrained identity (email unique, phone unique).
3. **2FA OTP** sent to the user-ID credential (or preferred channel); user enters OTP.
4. On success → role-appropriate landing (providers → Chart Central dashboard).
5. Session timer starts; PHI fields render masked per the user's role config.
6. Reveal-by-click on any masked field → value shown + reveal written to audit log.
7. After **30 minutes** of inactivity → auto-logout; re-auth required.

## Screens
- (No dedicated auth screenshot in provided set.) Login mirrors patient-portal email/phone pattern; OTP step reuses the `Mobile Verification → Enter OTP → Verify & Continue` flow seen on the website-widget screens.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| User | UserId, Email (unique), Phone (unique), PasswordHash, Role, Department, Status, ProviderType (if provider), Credentials |
| Role | RoleId, Name, Permissions[], CopiedFromRoleId (nullable) |
| Session | SessionId, UserId, StartedAt, LastActivityAt, ExpiresAt (30-min sliding) |
| PHIConfig | RoleId, Field (Phone/Email/SSN), Masked (bool) |
| AuditEvent | EventId, UserId, Action (login/reveal/edit), Target, Timestamp |

## Business Rules
- Login credential = **email OR phone**; both unique-constrained per account; **at least one required**, neither individually mandatory.
- Same email/phone may **not** map to two accounts (blocks guardian-reuse).
- **2FA OTP** routes to the user-ID credential or the preferred channel.
- **No username-based login** anywhere in the system.
- Session timeout = **30 minutes** inactivity (PHI-protection driven).
- Masking applies to **Phone / Email / SSN** by default; reveal logs to audit.
- New role creation supports **"Copy permissions from previous role"** checkbox + role selector.
- "Users" = anyone using the EMR (providers, office staff, billing agents, employees).

## MoM Decisions / Deltas (authoritative)
- **Login = Email OR Phone, NO username** (Dr. Mohammed locked, Jun 15 §44). Rationale: patients/users forget custom usernames → staff-reset overhead.
- **Email unique per account; Phone unique per account; at least one required** (resolves the Jun 4 guardian-multi-account open question).
- **2FA OTP** via user-ID credential or preferred channel.
- **PHI Configuration table** added (Dr. Mohammed's masking ask, Jun 4 §11): masked demographics = **Phone / Email / SSN**, reveal-by-click, **scoped per role type**, reveal **audit-logged**.
- **Copy-permissions-from-role** on new role creation (Jun 4 §10).
- **Users definition** = everyone using the EMR (Jun 4 §6).
- *(SRS note: Medent-style username login is superseded by email-as-unique-identifier.)*

## Dependencies
- Settings → Users / Roles / PHI Configuration (file 13) supplies role + masking config.
- Audit log service (cross-cutting) consumes all reveal/login events.
- OTP delivery: AWS SNS (SMS) / EnGard (email) per integrations (file 17).

## Open Items
- Landline-only OTP delivery for users/patients without a mobile — unresolved.
- Account-recovery / forgot-password flow detail (self-service reset) — to confirm.

## Acceptance Criteria
- Login accepts email or phone; rejects any attempt to register a duplicate email/phone.
- OTP required and delivered to the correct credential on every login.
- Session auto-expires at 30 min inactivity and forces re-auth.
- Phone/Email/SSN render masked per the logged-in user's role; reveal shows value and writes an audit entry with actor + timestamp.
- New role created via copy-permissions inherits the source role's permission set.

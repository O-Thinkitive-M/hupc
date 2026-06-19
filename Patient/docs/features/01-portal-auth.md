# Portal Authentication — Patient Portal Feature Spec

## Purpose
- Let patients and guardians securely sign in and access their (or linked children's) records.
- Replace the legacy Medent username login with **email + phone** identity, OTP-verified.
- Support a guardian holding **multiple child accounts** with an in-portal account switcher.

## Actors / Roles
| Actor | Capability |
|-------|-----------|
| Patient (Adult) | Sign in to own account, manage session. |
| Guardian | Sign in once, switch between linked child accounts. |
| System (Office Mgmt / EMR) | Issues activation, OTP delivery, audit logging. |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| AUTH-1 | Patient | to log in with email + phone (no username) | login is simple and memorable |
| AUTH-2 | Patient | an OTP to verify my identity | my account is protected |
| AUTH-3 | Guardian | to see/switch between my children's accounts | I manage each child without separate logins |
| AUTH-4 | Patient | the portal to time out when idle | my PHI is safe on shared devices |
| AUTH-5 | New patient | a portal activation link after onboarding | I can start using the portal |

## Primary Workflow
1. Patient opens portal → enters **email + mobile phone** (username field removed everywhere).
2. System validates credentials; sends **OTP via SMS (AWS SNS)** to the registered mobile.
3. Patient enters OTP → on success lands on dashboard.
4. If account is a **guardian** with linked children, an **account switcher** is shown; guardian picks which child's record to view.
5. Idle/session timeout forces re-authentication.
6. New users: activation link/code delivered during onboarding (email via EnGard / SMS via AWS SNS) → set credentials → first OTP login.

## Screens
No screen capture provided; per SRS/MoM the expected UI:
- **Login screen:** email field + phone field (no username), "Send OTP" primary button, accessible labels.
- **OTP screen:** single OTP input, resend link with cooldown, masked phone hint.
- **Dashboard:** appointments, messages, billing, records tiles.
- **Account switcher:** dropdown/list of linked child accounts with name + DOB; active account clearly indicated.
- Timeout modal with re-login prompt.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| PortalAccount | accountId, email, phone, status (active/pending), patientId |
| GuardianLink | guardianAccountId, childPatientId, relationship |
| OtpChallenge | challengeId, accountId, code, expiry, attempts |
| Session | sessionId, accountId, activeChildId, lastActivity, expiry |

## Business Rules
- Username login removed; **email + phone** is the identity pair, OTP-verified.
- Each portal user (or guardian's active child) sees only their own PHI (HIPAA self-access-only); access audited.
- Idle session times out; re-authentication required (interval TBD with HUPC).
- Guardian may have ≥1 linked child; switching active child re-scopes all portal data.
- OTP has expiry + limited attempts; resend rate-limited.

## MoM Decisions / Deltas (authoritative)
- **Patient portal login: email + phone (no username)** — username field replaced everywhere (MoM item 24).
- **Guardian with multiple child accounts shows multiple accounts** (Tushar) — switcher in portal (MoM item 25).
- **SMS/OTP vendor = AWS SNS; email vendor = EnGard** (MoM-confirmed).

## Dependencies
- AWS SNS (OTP/SMS), EnGard (activation email).
- Onboarding (02) for activation.
- Identity platform enforcing email uniqueness.

## Open Items
- **Unique-constraint conflict:** same email cannot map to two accounts on the identity platform — email-as-unique-identifier vs Medent-style username (initials+DOB) for the guardian-multiple-children case. **Dr. Mohammed to re-confirm** (MoM item 25, Open).
- Session timeout duration, OTP expiry/attempt limits — TBD.

## Acceptance Criteria
- No username field anywhere; login accepts email + phone and verifies via OTP.
- Guardian with two children can log in once and switch between both accounts.
- Switching active child re-scopes appointments/records/billing to that child only.
- Idle timeout enforces re-login; OTP enforces expiry + attempt limits.
- All access events written to the audit log.

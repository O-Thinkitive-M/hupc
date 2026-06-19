# Profile & Preferences — Patient Portal Feature Spec

## Purpose
- Let patients view/update demographics and contact details.
- Manage **communication preferences and consents** (email / phone / SMS / marketing opt-in).
- Set preferred pharmacy, lab, radiology, and provider.

## Actors / Roles
| Actor | Capability |
|-------|-----------|
| Patient (Adult) | Edit demographics, consents, preferences. |
| Guardian | Same for the active child account. |
| System | Honors stored consents across all channels. |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| PRF-1 | Patient | to update my demographics/contact info | my record stays accurate |
| PRF-2 | Patient | to set channel consents (email/phone/SMS/marketing) | I control how I'm contacted |
| PRF-3 | Patient | to set a preferred pharmacy | prescriptions go to the right place |
| PRF-4 | Patient | to set preferred lab/radiology | orders route correctly |
| PRF-5 | Patient | to pick a preferred provider | I see who I prefer |

## Primary Workflow
1. Patient opens Profile → reviews/updates demographics and contact info (email, phone — note these are also identity for login, see 01).
2. Patient sets **communication preferences/consents**: email, phone, **SMS consent**, **marketing opt-in**.
3. Patient sets **preferred pharmacy**, **preferred lab**, **preferred radiology**, and **preferred provider** (all optional).
4. Changes saved; consents are enforced wherever the system sends communications (10) — e.g., confirmation SMS only if SMS consent granted.
5. Sensitive fields (phone/email/SSN) masked in shared views; updates audited.

## Screens
No screen capture provided; per SRS/MoM the expected UI:
- Profile/demographics form with contact fields.
- Communication-preferences panel: toggles for Email, Phone, SMS, Marketing opt-in.
- Preferred-providers panel: pharmacy, lab, radiology, provider selectors (searchable, optional).
- Save with confirmation; masked sensitive fields.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Demographics | patientId, name, dob, address, phone, email |
| CommunicationConsent | patientId, emailConsent, phoneConsent, smsConsent, marketingOptIn |
| Preferences | patientId, preferredPharmacy, preferredLab, preferredRadiology, preferredProvider |

## Business Rules
- SMS/email/marketing communications honor stored consents (e.g., confirmation SMS only if SMS consent granted).
- Preferred provider/lab/pharmacy/radiology are optional (not mandatory).
- Editing email/phone affects login identity (coordinate with Auth, 01) — re-verification may be required.
- Sensitive demographics masked with reveal-by-click + audit, scoped per role (PHI configuration).
- Self-access-only; changes audited.

## MoM Decisions / Deltas (authoritative)
- Communication consents (email/phone/**SMS**/marketing) govern outbound messages (SRS, retained; channels = EnGard email + AWS SNS SMS per MoM).
- PHI masking with reveal-by-click + audit applies to phone/email/SSN (MoM PHI Configuration).
- No MoM change reverses optional preferred-provider behavior; remains optional.

## Dependencies
- Auth (01) — email/phone identity coupling.
- Notifications (10) — consent enforcement.
- Pharmacy/lab/radiology/provider masters (staff EMR).

## Open Items
- Whether changing login email/phone requires fresh OTP re-verification — confirm with Auth.
- Granularity of marketing opt-in (categories vs single toggle) — TBD.

## Acceptance Criteria
- Patient can update demographics and save successfully.
- Channel consents (email/phone/SMS/marketing) are stored and enforced on outbound messages.
- Preferred pharmacy/lab/radiology/provider can be set and remain optional.
- Sensitive fields masked; all edits audited.

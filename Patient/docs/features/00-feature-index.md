# Patient Portal — Feature Index & Glossary

## Purpose
- Entry point and map for all patient-facing portal feature specs of the HUPC / Harmony EMR.
- Audience: product, QA, engineering building the **patient/guardian-facing** portal (distinct from the staff EMR).
- Authority order: **QA MoM V2 OVERRIDES the SRS.** Each spec carries a "MoM Decisions / Deltas (authoritative)" section.

## Actors / Roles
| Actor | Description |
|-------|-------------|
| Patient (Adult) | 18+, self-consents, default financial-responsible party; full self-access portal user. |
| Guardian | Parent/legal guardian acting for one or more Minor or Non-Consenting-Adult patients; sees and switches between linked child accounts. |
| (System counterpart) Office Management | Not a portal actor, but ALL portal requests/responses route through it first (see 04). |

## Module Map
| File | Module | Core MoM delta |
|------|--------|----------------|
| 01-portal-auth.md | Login, OTP, guardian switcher, sessions | Email + phone login replaces Medent username |
| 02-onboarding-intake.md | Registration, consent, PFSH/intake, pre-check-in | Portal activated during onboarding |
| 03-appointments-scheduler.md | Book / reschedule / cancel, telehealth vs in-person | Self-service reschedule+cancel, 24h fee window |
| 04-messaging.md | Provider↔patient requests | All requests default-route to Office Management |
| 05-telehealth.md | Patient-side video join | Google Meet (not Zoom); link via SMS/email |
| 06-documents.md | Upload + view shared docs | Attachment document-type selector |
| 07-health-records.md | Psychiatric history, clinical data visibility | Past psychiatric history patient-view added |
| 08-billing-payments.md | Invoices, payments, plans | Check payment method; "Paid Amount" column |
| 09-profile-preferences.md | Demographics, consents, preferred providers | Communication preferences/opt-ins |
| 10-notifications.md | Email/SMS portal notifications | EnGard email + AWS SNS SMS |

## Glossary
| Term | Meaning |
|------|---------|
| OTP | One-time password sent via SMS to verify login/identity. |
| ROI | Release of Information rule — past-encounter notes blocked from patient view. |
| Office Management | Triage department that reads and routes every portal request. |
| On-call / coverage provider | Provider covering responses when the assigned provider is out of office. |
| PFSH | Past, Family & Social History intake. |
| Document type selector | Dropdown the patient uses when attaching a file; controls chart placement. |
| Self-Pay / Insurance | Two fee tiers; copay derived from eligibility. |

## Confirmed Vendors (MoM-authoritative)
| Capability | Vendor | Note |
|------------|--------|------|
| Email | **EnGard** | NOT SendGrid. |
| SMS / OTP / notifications | **AWS SNS** | NOT Twilio. |
| Telehealth video | **Google Meet** | NOT Zoom. |

## Cross-Cutting Requirements (apply to every module)
- **Accessibility:** WCAG 2.1 AA — keyboard nav, focus order, color contrast, labels/ARIA, screen-reader support.
- **Mobile-first:** responsive layouts, touch targets, low-bandwidth friendly; primary device is the patient's phone.
- **i18n:** all patient-facing strings externalized; locale-aware dates/times; expandable to additional languages.
- **HIPAA self-access-only:** a portal user sees only their own (or their linked child's) PHI; reveal/access is audited.
- **Masking + audit:** sensitive demographics (phone, email, SSN) masked in shared views; access events logged.
- **Consent-gated communication:** email/SMS/marketing channels honored per stored consents (see 09).

## Open Items (portfolio-level)
- Guardian-with-multiple-children identity model (email unique-constraint) — Dr. Mohammed to re-confirm.
- Cancellation fee amount/policy — 24h window confirmed, amount TBD.
- Document-type taxonomy not finalized (Photo ID, insurance card, etc.).
- Additional languages for i18n — TBD.

## Acceptance Criteria
- All 11 spec files present and internally consistent on actors, vendors, and routing rules.
- No spec contradicts a MoM V2 decision; SRS-only items flagged as such.
- Cross-cutting requirements referenced (not re-defined) by each module spec.

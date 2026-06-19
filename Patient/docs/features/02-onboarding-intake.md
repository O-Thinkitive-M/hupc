# Onboarding & Intake — Patient Portal Feature Spec

## Purpose
- Convert a new or existing patient into an activated portal user.
- Collect registration data, consents, PFSH/intake history, and pre-check-in info before the visit.
- Activate the patient portal so the patient can self-serve afterward.

## Actors / Roles
| Actor | Capability |
|-------|-----------|
| Patient (Adult) | Completes own registration, consents, intake. |
| Guardian | Completes registration/consents on behalf of a Minor or Non-Consenting Adult. |
| System | Classifies patient type, activates portal, auto-populates consent forms. |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| ONB-1 | New patient | to register online | I avoid paperwork at the office |
| ONB-2 | Guardian | to consent for my child | the minor can be treated |
| ONB-3 | Patient | to complete intake/PFSH forms before my visit | the provider has my history |
| ONB-4 | Patient | to pre-check-in | check-in at the office is faster |
| ONB-5 | Existing patient | to be redirected to portal login | I reuse my account |

## Primary Workflow
1. Patient self-identifies as **New** or **Existing**; existing patients are redirected to portal login (01).
2. New patient registers; system classifies patient type: **Adult / Minor / Non-Consenting Adult** (drives consent + financial logic).
3. Guardian section required for Minor / Non-Consenting Adult; legal-documentation upload required for Non-Consenting Adult (POA / next-of-kin).
4. Consent forms presented and signed (auto-populated by patient type); privacy notice shown at point of data entry.
5. Patient completes **intake + PFSH** forms (medical history, medications, allergies) — these later auto-pull into the provider note.
6. Pre-check-in collects/confirms demographics, insurance, marketing attribution ("How did you hear about us?").
7. **Portal is activated** so the patient can access details, book, message, pay, and view records.

## Screens
No screen capture provided; per SRS/MoM the expected UI:
- Registration form with patient-type-driven sections (guardian block appears for Minor/Non-Consenting).
- Consent screen with signature capture and inline privacy notice.
- Intake/PFSH multi-step forms (history, medications, allergies) with progress indicator.
- Pre-check-in summary + marketing-attribution mandatory field.
- Portal-activation confirmation.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Registration | patientId, patientType, demographics |
| GuardianSection | guardianName, relationship, legalDocUpload (Non-Consenting) |
| ConsentForm | consentType, signedBy, signedDate, version |
| IntakeForm / PFSH | medicalHistory, medications, allergies, social/family history |
| MarketingAttribution | source (Google, Social, Referral, …) |

## Business Rules
- Patient type (Adult/Minor/Non-Consenting Adult) drives consent authority and default financial-responsible party.
- Non-Consenting Adult requires legal-documentation upload; guardian signs consents.
- Consent + privacy language presented at point of data entry; no PHI stored before consent where applicable.
- Intake clinical data (allergies/medications/etc.) auto-pulls into the provider note — patient is the source of truth (MoM clinical auto-pull).
- "How did you hear about us?" is mandatory (marketing attribution).

## MoM Decisions / Deltas (authoritative)
- **Clinical data (vitals, allergies, current medications) auto-pulls from patient portal into the note** — no manual pull (MoM item 1, Clinical section).
- **PHQ-9 / intake answers entered in portal auto-populate the note** (MoM clinical section).
- Portal activated as part of onboarding (SRS, retained).

## Dependencies
- Auth (01) for activation + login.
- EnGard (activation email) / AWS SNS (SMS).
- Clinical documentation module (consumes auto-pulled intake data).

## Open Items
- Intake/PFSH required-field set and form versions — TBD with HUPC.
- ID-expiration notification intervals/channels/templates — TBD.

## Acceptance Criteria
- New patient can register, be classified by type, consent, complete intake, and get portal activated.
- Guardian completes flow for Minor/Non-Consenting with required legal upload.
- Existing patient is redirected to portal login, not re-registered.
- Intake clinical data is available to auto-pull into the provider note.

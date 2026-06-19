# Entry & Routing — Website Widget Feature Spec

## Purpose
- Define the public entry point ("Schedule an Appointment" CTA) and the initial routing fork that sends prospects into the new-patient flow and existing patients to the Patient Portal.
- Establish HIPAA posture for the patient-facing, pre-auth surface.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Selects "New Patient", enters booking flow |
| Existing Patient | Selects "Existing Patient", redirected to Portal login |
| Guardian / Legal Representative | Enters flow on behalf of a minor |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| ER-1 | Prospect | a clear booking CTA on hpcfl.com | I can start scheduling |
| ER-2 | Prospect | to pick New vs Existing | I'm routed to the right flow |
| ER-3 | Existing patient | to be sent to the Portal login | I book follow-ups via my account |
| ER-4 | Patient | confidence my data is private | I trust the public form |

## Primary Workflow
1. Patient clicks **Schedule an Appointment** on hpcfl.com.
2. Widget launches as a **modal or dedicated page**.
3. Widget presents two paths: **Existing Patient** / **New Patient**.
4. **Existing Patient** → redirect to Patient Portal login (no identity check here).
5. **New Patient** → enter new-patient booking flow (→ Step 1 Identification).
6. Consent / privacy notice presented at point of data entry.
7. If an existing patient mistakenly picks "New Patient", duplicate detection (file 09) catches it before any record is created.

## Screens
- No screen capture provided; per SRS/MoM, expected UI:
  - **CTA button** on public site, prominent, mobile-friendly.
  - **Gate screen**: two large selectable cards/buttons — "Existing Patient" and "New Patient" — with brief helper text.
  - **Privacy/consent banner** at first data-entry step.
  - **Redirect interstitial** for existing patients ("Redirecting to your patient portal…").

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Session context | Entry channel, selected path (new/existing), selected location (if from Office Locator) |
| Consent record | Privacy notice acknowledgement timestamp |

## Business Rules
- Existing patients do **NOT** proceed through new-patient registration; always routed to Portal.
- No identity verification at the gate — routing is by patient self-selection, validated later by match keys + duplicate detection.
- Selected location (from Office Locator) carries forward into the downstream booking flow.
- No PHI stored in third-party form handlers or analytics platforms.

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) **"Schedule an Appointment" gate added** with 2 options: Existing Patient (→ Patient Portal) / New Patient (→ form). [decision locked]
- New-Patient form sections begin at this gate: Personal Information / Appointment Details / Address Information / Insurance Information / Financial Responsibility / Additional Information.
- Inactive patients cannot be scheduled (staff-side lock); on the widget this is enforced via the new/existing gate + downstream duplicate detection.

## Dependencies
- Patient Portal (redirect target for existing patients).
- Office Locator (optional pre-step supplying selected location).
- Patient Identification (file 02) and Duplicate Detection (file 09).

## Open Items
- Exact CTA placement / count across hpcfl.com pages — web-dev maintained.
- Modal vs dedicated-page final decision for launch surface.

## Acceptance Criteria
- CTA launches widget gate with exactly two paths.
- Existing Patient selection redirects to Portal login without collecting PHI.
- New Patient selection starts identification step.
- Consent/privacy language shown before any PHI capture.

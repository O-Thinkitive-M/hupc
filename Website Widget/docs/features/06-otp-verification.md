# OTP Phone Verification — Website Widget Feature Spec

## Purpose
- Define **Step 5**: mobile OTP verification of the patient's phone number to prevent spam submissions and confirm a valid follow-up contact.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Receives + enters OTP |
| SMS provider | Delivers OTP via SMS |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| OTP-1 | Prospect | a code to my phone | I verify the number is mine |
| OTP-2 | Prospect | to resend if it didn't arrive | I'm not blocked |
| OTP-3 | HUPC | spam submissions reduced | only real contacts proceed |

## Primary Workflow
1. After eligibility pass (insurance) or directly after registration (self-pay), system sends OTP via SMS to the entered mobile number.
2. Patient enters OTP on the verification screen.
3. **Success** → proceed to Provider Selection (Step 6, file 07).
4. **Failure** → patient can request a **resend**.
5. Communication log tracks resend count.

## Screens
- No screen capture provided; per SRS/MoM, expected UI:
  - **Mobile Verification** screen: masked phone display, OTP input, **Verify & Continue**.
  - **Resend** link with cooldown.
  - Error state for invalid/expired code.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| OTP challenge | Mobile number, code, expiry, attempts, resend count |
| Verification result | Verified? (bool), timestamp |

## Business Rules
- OTP is sent to the mobile number captured in registration (mobile is required).
- Verification must succeed before provider selection.
- Resend allowed on failure; resend count tracked in the communication log.
- Mobile-only delivery in current scope (SMS).

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) **OTP flow confirmed**: after form submit + eligibility check → Continue → Mobile Verification screen → Enter OTP → **Verify & Continue** → provider/date/slot selection. [decision locked]
- Communication log tracks **resend count**.
- Patient Portal login also moved to **email + phone (OTP-style)**; no username.
- **OPEN:** older patients with landline-only (no mobile) OTP delivery — deferred (Jim raised; not resolved).

## Dependencies
- Registration Form (file 03) mobile number.
- Eligibility Check (file 05) precedes OTP for insurance patients.
- SMS provider.
- Provider Selection (file 07) follows success.

## Open Items
- Landline-only OTP delivery for patients without a mobile — open/deferred.
- OTP expiry window and max-attempts — config.

## Acceptance Criteria
- A valid OTP entry advances the patient to provider selection.
- An invalid OTP shows an error and allows resend.
- Resend count is recorded in the communication log.
- Mobile number is required upstream so OTP can always be sent (within current mobile scope).

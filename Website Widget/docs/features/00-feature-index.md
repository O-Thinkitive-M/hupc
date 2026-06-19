# Website Widget — Feature Index

## Purpose
- Single-source index for the **Website Widget**: the public, pre-auth, patient-facing online-booking surface embedded on the HUPC public website (hpcfl.com).
- Primary actor: **New Patient / prospective patient** initiating an appointment via the "Schedule an Appointment" call-to-action.
- Built **inside** the HUPC EHR (not a third-party form e.g. Jotform); data flows directly into the EHR under the same HIPAA controls.

## Actors / Roles
| Actor | Role in widget |
|-------|----------------|
| New Patient (prospect) | Completes new-patient booking flow; primary actor |
| Existing Patient | Redirected to Patient Portal (does NOT use new-patient flow) |
| Guardian / Legal Representative | Books on behalf of a minor / non-consenting adult |
| HUPC Scheduling / Office Mgmt staff | Confirm new-patient requests, resolve eligibility, assign providers (downstream, not in-widget) |
| Insurance Rep | Submits via Lead Management widget only — no direct scheduling |

## Module Map (files in this set)
| File | Module | Flow step |
|------|--------|-----------|
| `01-entry-routing.md` | Entry & CTA routing, existing-patient redirect, HIPAA | Entry |
| `02-patient-identification.md` | New vs existing identification (match keys) | Step 1 |
| `03-registration-form.md` | Single comprehensive registration form | Step 2 |
| `04-insurance-card-ocr.md` | Card upload + OCR auto-fill + manual fallback | Step 3 |
| `05-eligibility-check.md` | Real-time eligibility pass/fail | Step 4 |
| `06-otp-verification.md` | Mobile OTP phone verification | Step 5 |
| `07-provider-selection.md` | Provider cards, filtering, detail screen | Step 6 |
| `08-availability-timeslots.md` | One-week availability, location display, slots | Step 7 |
| `09-duplicate-detection.md` | Pre-creation duplicate detection | At submit |
| `10-same-day-booking-rules.md` | One insurance appt/day, self-pay exception | At submit |
| `11-booking-type.md` | Follow-up direct vs new-patient request-only | At submit |
| `12-payment-confirmation.md` | Payment capture, auto-charge, confirmation | Step 8 |

## End-to-End Booking Flow Overview
1. Patient clicks **Schedule an Appointment** → chooses Existing (→ Portal) or New Patient.
2. **Step 1** Identification — match keys; existing → Portal, new → form.
3. **Step 2** Registration form (Personal, Appointment Details, Address, Insurance, Financial Responsibility, Additional Info).
4. **Step 3** Insurance card upload + OCR auto-fill (Insurance payment type only; Self-Pay skips).
5. **Step 4** Eligibility check (insurance only; Self-Pay skips). Fail → Callback or Live Agent.
6. **Step 5** Mobile OTP verification.
7. **Step 6** Provider selection (eligibility-aware, filtered).
8. **Step 7** Availability one-week view + time-slot selection.
9. **At submit** Duplicate detection + same-day rules enforced.
10. **Step 8** Payment capture → submit as **request** (new patient = pending staff confirmation).
11. Confirmation email/SMS; pre-check-in + portal activation sent downstream.

## Glossary
| Term | Meaning |
|------|---------|
| CTA | "Schedule an Appointment" call-to-action button on hpcfl.com |
| OCR | Optical character recognition for insurance-card field extraction |
| Eligibility check | Real-time payer verification (active plan, in-network, copay/deductible) |
| OTP | One-time password via SMS for mobile verification |
| Lead | Pre-patient record in Lead Management (created on no-match new submission) |
| Match Key | Field combination used to detect existing patients/leads |
| Self-Pay | Patient pays directly; skips insurance/eligibility |
| Pre-check-in | Downstream consent + PFSH completion flow (not in widget) |

## Cross-Cutting Concerns
| Concern | Rule |
|---------|------|
| HIPAA | Widget lives inside EHR; no PHI in 3rd-party handlers/analytics; consent shown at data entry |
| Existing-patient redirect | Existing patients always routed to Portal login (never new-patient flow) |
| Eligibility-aware UX | Provider list + cost surfaced based on verified eligibility |
| Mobile-first | Phone/webcam card capture, mobile OTP, touch-friendly UI |
| Accessibility | Accessible, mobile-first patient-facing surface |
| i18n | Spoken-language preference (soft provider filter); patient-facing language support |
| New patient = request | New-patient bookings enter "Requested — Pending Staff Confirmation" |

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) New Patient form sections **locked**: Personal Information / Appointment Details / Address Information / Insurance Information / Financial Responsibility / Additional Information.
- "Suffix" field **renamed to "Prefix"** and made **optional** (Dr. Mohammed correction).
- Financial Responsibility: "Guardian" → **"Guardian / Legal Representative"**; "Organization" → **"Other"**.
- DOB-driven **Minor/Adult auto-detection** live (minor → Guardian section).
- Provider filter set locked: Appointment Mode, Appointment Type, Reason to Visit, Location, Language, Age Group, Insurance, Diagnosis, Therapy Modality.
- Office Locator: search by **ZIP or city**, distance-sorted.
- Insurance reps → **Lead Management widget only** (no direct scheduling).
- Inactive patients cannot be scheduled (staff-side lock); widget already gates via new/existing.
- Portal login = **email + phone** only (no username); both unique-constrained.

## Dependencies
- Scheduling module availability engine (slots).
- Lead Management (lead creation, duplicate logic, convert-to-patient).
- Patient Registration / EHR (account creation).
- Eligibility verification vendor; payment processor (Stripe).
- hpcfl.com/providers listing (provider-attribute source of truth).

## Open Items
- Landline-only OTP delivery for older patients without mobile — open.
- Guardian using one email for two child accounts vs unique-constraint — pending Dr. Mohammed.
- "Pending Credentialing" provider visibility — pending HUPC confirmation.

## Acceptance Criteria
- Each module file below conforms to the shared template and reflects MoM-over-SRS precedence.
- Flow is traceable end-to-end from CTA to confirmation.
- New-patient submissions never create duplicate records.

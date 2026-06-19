# Payment & Confirmation — Website Widget Feature Spec

## Purpose
- Define **Step 8**: payment capture at booking, supported methods, encryption/tokenization (Stripe), auto-charge at confirmation, refunds/adjustments, and the booking confirmation + post-submission experience.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Enters payment, consents, receives confirmation |
| HUPC staff (AR / Office Mgmt) | Handle failed charges, manual collection, refunds |
| Payment processor (Stripe) | Tokenizes + charges |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| PC-1 | Prospect | to see my expected charge | I know what I'll pay |
| PC-2 | Prospect | to save a payment method securely | future visits are easy |
| PC-3 | Prospect | to opt out of auto-charge | staff collects manually |
| PC-4 | Prospect | confirmation + next steps | I know my booking is in |
| PC-5 | Patient | to manage saved methods in the portal | I stay in control |

## Primary Workflow
1. After slot selection, patient enters payment (card or bank), encrypted + tokenized via processor.
2. Widget calculates + displays expected charge: eligibility-derived copay/deductible (insurance) OR self-pay rate (self-pay).
3. Consent captured to store payment method on file (editable in portal).
4. Patient may **opt out of auto-charge** at booking.
5. On submission, system creates the account, books the appointment (as request for new patients — file 11), and sends communications.
6. **Auto-charge fires at staff confirmation**, not at submission.
7. Patient receives confirmation + receipts; failed charges create an AR/Office Mgmt task.

## Payment Methods
| Method | Notes |
|--------|-------|
| Credit / debit card | Visa, Mastercard, Amex, Discover |
| ACH bank transfer | Account holder, bank, account number, type |
| HSA / FSA cards | Treated as credit cards by processor |

## Encryption & Storage
| Aspect | Rule |
|--------|------|
| In transit / at rest | Encrypted |
| Tokenization | Card numbers + bank routing tokenized via processor (e.g., Stripe) |
| Stored by EHR | Token + last four digits only |
| Staff visibility | Staff cannot view full card numbers |

## Auto-Charge & Refunds
| Aspect | Rule |
|--------|------|
| Trigger | Calculated amount auto-charged when appointment confirmed by staff |
| Receipt | Email/SMS receipt sent |
| Failure | Failed charges → task in AR / Office Mgmt queue |
| Opt-out | Patient may opt out at booking → staff collects manually |
| Cancellation refund | Cancel before auto-charge cutoff (configurable, e.g., 24h pre-appt) → charge auto-reversed |
| Completed-appt refunds | Follow standard refunds workflow |

## Confirmation & Post-Submission
- On confirmation the system: creates the EHR patient account; books the appointment; sends portal activation email, pre-check-in email, and (telehealth) video-visit link; sends confirmation via email + SMS (SMS if consent granted).
- Patient sees a success message + "Return to Home" button (back to hpcfl.com).
- Subsequent communications (pre-check-in, telehealth link, reminders) via standard patient communication pipeline.
- New-patient prerequisites messaged: consent forms (Financial Responsibility, HIPAA Notice, Practice Policies) + PFSH are mandatory, completed via pre-check-in (not in widget).

## Screens
- No screen capture provided; per SRS/MoM, expected UI: payment form (card/bank) with expected-charge summary, save-method + auto-charge consent toggles, success/confirmation screen with Return to Home.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Payment method | Token, last four, type, holder, auto-charge consent |
| Charge | Amount (copay/deductible or self-pay rate), status, receipt, cutoff window |
| Confirmation | Appointment details, comms sent (email/SMS), next-step links |

## Business Rules
- Payment info collected at booking; **charge occurs at confirmation**, not submission.
- Patient consent required to store method on file; editable from portal anytime.
- Only token + last four retained; full numbers never stored/visible to staff.
- Opt-out shifts to manual staff collection.

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) Financial Responsibility payment-method capture (card or bank) is part of the form: **Self / Guardian-Legal Representative / Other** each carry a payment method.
- Slot selection ends with **Confirm Booking**; new-patient bookings remain request-status until staff confirms (drives auto-charge timing).
- No MoM change to Stripe tokenization / token + last-four storage model.

## Dependencies
- Eligibility (file 05) for copay/deductible; Provider/Slots (files 07–08) for amount basis.
- Payment processor (Stripe).
- AR / Office Management (failed charges, manual collection, refunds).
- Patient Portal (manage saved methods).

## Open Items
- Auto-charge cutoff window value (e.g., 24h) — configurable.

## Acceptance Criteria
- Expected charge displays correctly for insurance vs self-pay.
- Payment is tokenized; only token + last four stored; staff cannot see full numbers.
- Auto-charge fires at staff confirmation; opt-out routes to manual collection.
- Cancellation before cutoff auto-reverses the charge.
- Confirmation triggers account creation, booking, and the email/SMS communication set.

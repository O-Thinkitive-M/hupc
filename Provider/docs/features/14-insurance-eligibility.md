# Insurance & Eligibility — Provider Portal Feature Spec

## Purpose
- Insurance capture, real-time eligibility verification, payer enrollment/credentialing filtering, and prior authorization in Harmony EMR.
- Drives provider visibility (credentialing/payer match), copay/deductible surfacing, and downstream billing accuracy.

## Actors / Roles
| Role | Capability |
|------|------------|
| Agent / Scheduling | Run eligibility, resolve fails (callback/live agent), capture insurance |
| Biller | Verify coverage, manage prior auth, payer enrollment status |
| Provider | Credentialing/network status drives bookability and submission (Rendering vs Supervising) |
| Admin | Payer Master, Allowed Amount, credentialing config |
| Patient/Guardian | Enter insurance/subscriber details at intake |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| IE-1 | Agent | a real-time eligibility check on submitted insurance | I confirm active/in-network + copay/deductible |
| IE-2 | Agent | pass/fail handling with callback or live-agent fallback | failed eligibility still gets booked |
| IE-3 | Biller | payer-enrollment/credentialing filtering | only credentialed providers are shown for that payer |
| IE-4 | Biller | prior authorization workflow + copy-all patient data | I complete prior-auth forms quickly |
| IE-5 | Biller | provider network status (color-coded, future-dated) | I submit under Rendering vs Supervising correctly |

## Primary Workflow
1. Insurance captured at intake (payer, member ID, group, subscriber details if patient ≠ subscriber).
2. **Eligibility Check** runs in real time against the entered insurance (self-pay skips this step).
   - Determines: plan active? HUPC in-network? copay/deductible for the service? plan service coverage (Med Mgmt, Psychotherapy, etc.).
3. **Pass** → proceed (copay/deductible surfaced for booking, eligibility-aware provider display).
4. **Fail** (inactive, out-of-network, vendor outage, unsupported payer) → two options:
   - **Request a Callback** (captures registration data; scheduling team follows up within SLA).
   - **Connect to Live Agent** (phone bridge; gated by operating hours).
5. **Payer Enrollment / Credentialing filter** — providers Not Credentialed for the patient's payer are hidden; Pending Credentialing visibility = TBD; self-pay bypasses.
6. **Prior Authorization** — biller compiles patient/insurance data (Copy-All for prior-auth forms), submits/ tracks auth, attaches to encounter/claim.
7. **Provider Network Status** (color-coded, future-dated) informs billing whether to submit under Rendering or Supervising provider.

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-17-15.png` / `…15-17-35.png` — insurance entry + eligibility result (pass/fail). *(verify in intake/patient capture set)*
- Settings → Billing → Payer + Allowed Amount (module 16) for enrollment/credentialing config.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Insurance | patientId, payerId, memberId, group, subscriber (if not patient), planType |
| EligibilityCheck | insuranceId, runAt, result (Pass/Fail), reason, planActive, inNetwork, copay, deductible, coveredServices[] |
| PayerEnrollment | providerId, payerId, credentialingStatus (Credentialed/Pending/Not Credentialed), effectiveDates |
| PriorAuth | patientId, payerId, serviceCpt, authNumber, status, validFrom, validTo, attachments |
| ProviderNetworkStatus | providerId, payerId, status (color-coded), futureEffectiveDate |

## Business Rules
- Self-pay patients skip eligibility entirely.
- Eligibility runs real-time on insurance submission; copay/deductible drive booking display.
- Fail path always offers callback; live-agent gated by operating hours (hidden/disabled outside hours).
- Provider filtering: Not Credentialed for payer → hidden; self-pay bypasses credentialing filter.
- "Accepting New Patients" + age-range + credentialing all gate provider visibility at booking.
- Provider Network Status is color-coded and future-dated; billing uses it for Rendering vs Supervising submission.
- Copy-All available for prior-auth/invoice patient data (clipboard).
- Allowed Amount (contracted rate) is per CPT + Payer + Provider Type with effective dates (lives in Settings/Billing).
- PHI masking + audit logging on eligibility runs and reveals.

## MoM Decisions / Deltas (authoritative)
- **Provider Network Status** (color-coded, future-dated) confirmed — billing decides Rendering vs Supervising submission (carried from May 21).
- **Payer Master** adds Mailing Address + Contact Number ("contact number is must"); Payer ID already present; Client Type + Status retained.
- **Allowed Amount** tab: Procedure Code / Payer / Provider Type / Amount / Effective Dates; inline add for new payer + CPT; changes audit-logged.
- Provider-type-aware rate logic lives at the **Allowed Amount / Fee Schedule** level — **not** on the ICD-10/CPT master (Provider Type column removed from masters, Jun 16).
- Insurance details used downstream for verification + eligibility (SRS) drive eligibility-aware provider display and copay/deductible.
- ID expiration: automated notification to patient (portal) + internal task on approaching expiry (intervals/channels TBD).

## Dependencies
- Eligibility/clearinghouse vendor integration (real-time 270/271).
- Scheduling/provider-selection widget (credentialing/age/new-patient filters).
- Settings → Payer Master, Allowed Amount, credentialing (module 16).
- Billing claims (Rendering/Supervising submission, module 12).
- Notifications (callback SLA, ID-expiry alerts, module 17).

## Open Items
- "Pending Credentialing" provider visibility rule — pending HUPC confirmation (SRS Open Item 9.A).
- ID-expiration notification intervals/channels/templates — TBD with HUPC.
- Prior-auth UI/field set not yet detailed in MoM — confirm in a future session.

## Acceptance Criteria
- [ ] Eligibility runs in real time on insurance submit; self-pay skips it.
- [ ] Pass surfaces copay/deductible and coverage; Fail offers callback + live-agent (hours-gated).
- [ ] Providers not credentialed for the payer are hidden; self-pay bypasses the filter.
- [ ] Provider Network Status (color-coded, future-dated) drives Rendering vs Supervising submission.
- [ ] Prior-auth workflow supports Copy-All of patient/insurance data.
- [ ] Payer Master shows Mailing Address + Contact Number; Allowed Amount per CPT/Payer/Provider Type with effective dates.
- [ ] Eligibility runs and reveals are audit-logged; PHI masked per role.

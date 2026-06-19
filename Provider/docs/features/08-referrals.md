# Referrals — Provider Portal Feature Spec

## Purpose
Manage inbound and outbound referrals across fax and email channels. Provides a referral worklist + detail, a Create-Referral drawer (Referral In / Out), side-by-side fax review with OCR extraction, ROI gating on outbound, and ICD-10 / procedure coding.

## Actors / Roles
| Role | Use |
|------|-----|
| Agent (Office) | Process inbound faxes/emails, create referrals, convert to patient |
| Provider | Review reason-for-referral, initiate referral out |
| Admin | Configure document templates, referral sources |
| Biller | Eligibility context on referred patients |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| REF-1 | Agent | inbound referrals via fax + email sub-tabs | all channels are tracked |
| REF-2 | Agent | side-by-side fax view + OCR extraction | I create/attach patients accurately |
| REF-3 | Agent | a Create-Referral drawer (In/Out) | I record referrals consistently |
| REF-4 | Provider | initiate referral out with ROI gate | consent is enforced |
| REF-5 | Agent | flag STAT/Primary/Secondary | urgency is prioritized |
| REF-6 | Admin | track referrals by referring provider | I analyze referral outcomes |

## Primary Workflow (Referral In)
1. Inbound **fax or email** arrives → appears under the referral channel sub-tab (today's faxes bold, older grayed).
2. Agent opens → **side-by-side view**: original document | extracted data (Reason for Referral, referring provider, diagnosis/procedure codes, demographics).
3. Edit/correct extracted fields → **Create New Patient** OR **Attach to Existing** (DOB shown for verification).
4. Saved fax stored in patient **Documents**; new patient linked to **Lead Management** as source.
5. Referral tracked: Pending / Converted / Incomplete / Rejected.

## Primary Workflow (Referral Out)
1. Staff initiates **Referral Out** via Create-Referral drawer ("Referred To").
2. System checks **ROI**: if not on file → generate ROI consent → send to patient via portal for e-signature → status **Pending ROI**.
3. On signature → referral can be **Sent**; status updates. Drafts saveable anytime (statuses: **Sent / Draft**).
4. Records: recipient facility/provider, date sent, documents included, status.

## Create Referral Drawer
Two popups (Referral In / Referral Out) share most fields; difference: **Referred From** (In) vs **Referred To** (Out). Fields: patient, referring/recipient provider, reason, diagnosis (**ICD-10**), procedure codes, urgency (**STAT / Primary / Secondary**), attached documents.

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-41.png` — Referrals list (In/Out, fax/email sub-tabs).
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-45.png` — Referral detail / side-by-side fax review + Create Referral drawer.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Referral In | ReferralId, ReferredFrom, Patient, Reason, ICD-10[], ProcedureCodes[], Urgency, Documents, Status (Pending/Converted/Incomplete/Rejected) |
| Referral Out | ReferralId, ReferredTo, Patient, Documents, DateSent, ROIStatus, Status (Sent/Draft/Pending ROI) |
| Fax entry | Sender/Number, PageCount, ReceivedDate/Time, Status, Annotations |
| ROI | ConsentDoc, SignedDate, PortalLink |

## Business Rules
- **Outbound referral requires signed ROI** before sending; else Pending ROI.
- Side-by-side view applies to **every** fax action (create/attach/assign), not just OCR create.
- Extracted data always editable before save; original document stays linked.
- Fax inbox under **Chart Central / Medical Records**; dedicated agent monitors.
- Outbound fax requires document uploaded to chart Documents first; choose template (Insurance Letter, Medical Records, Lab Results, FMLA, Prior Auth, ROI, General, Other).
- Unique referral links auto-tag the referral source.
- PHI masked per role; reveal audit-logged.

## MoM Decisions / Deltas (authoritative)
- **Referrals surfaced in patient chart**: all patient-related referrals from **Fax/Email** with filter + search; Referral Out section; Print Recent Referral + Create Follow-up Tasks (Jun 15 §30).
- **SMS via AWS SNS** + email via EnGard used for referral/ROI notifications (vendor session) — *not* RingCentral (RingCentral = call widget only).
- ROI gate + **Sent/Draft** statuses (SRS, carried forward; not contradicted by MoM).
- STAT-keyword OCR detection to elevate urgent faxes — **pending OCR-reliability confirmation** (Open Item).

## Dependencies
- Communication (file 09) — fax (SR Fax) + email channels, Message Central.
- Patients chart (file 05) — Documents, referral history, attach-to-existing.
- Leads (file 03) — new patient from referral linked as lead source.
- Patient Portal (file 14) — ROI e-signature.
- Integrations (file 17) — SR Fax, SignNow (e-sig), OCR.

## Open Items
- STAT / urgent keyword OCR detection reliability — Thinkitive to confirm.
- E-signature vendor for ROI (SignNow) contract timing.

## Acceptance Criteria
- Inbound referrals appear under fax + email sub-tabs with bold today / grayed older.
- Side-by-side view shows source + extracted data for every fax action.
- Create-Referral drawer supports In (Referred From) and Out (Referred To) with ICD-10 + STAT/Primary/Secondary.
- Referral Out is blocked until ROI is signed; supports Sent/Draft/Pending ROI.
- New patient from a referral is linked to Lead Management as a source.
- All referral PHI masked by default with audit-logged reveal.

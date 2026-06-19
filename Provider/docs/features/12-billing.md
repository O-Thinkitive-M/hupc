# Billing — Provider Portal Feature Spec

## Purpose
- End-to-end revenue cycle in Harmony EMR: Charge Capture & Coding → Claims → Remits → AR Management → Patient Invoicing → Patient Payments → Patient Collection.
- Replaces Medent billing flows; "everything related to money is handled by the Billing team only" (refunds, invoices, POs included).

## Actors / Roles
| Role | Capability |
|------|------------|
| Biller / Billing Lead | Code, scrub, submit/correct claims, post remits, work AR, invoice, take payments, refunds |
| AR Agent | Work assigned claims; log attempts; TFL follow-up |
| Coder | Charge capture review (Pending Coder Review → Approved to Post) |
| Provider | Sign-and-lock note that feeds CPT/ICD into charge capture |
| Admin | Fee Schedule, Allowed Amount, Payer Master, Alerts config |
| Patient/Guardian | Receive invoices/statements; pay via portal link |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| BL-1 | Coder | charges in Pending Coder Review / Approved to Post / Internal Scrubbing | I review before posting |
| BL-2 | Biller | bulk-assign + bulk-submit claims | I process volume efficiently |
| BL-3 | Biller | a distinct "Corrected Claim" action (CMS frequency 7) | corrected resubmissions are filed correctly vs new claims |
| BL-4 | Biller | EDI auto-sync + PDF/manual remit upload | paper remits are captured too |
| BL-5 | AR Agent | TFL alerts + an Attempts column | I see touches per claim and beat the filing deadline |
| BL-6 | Biller | "Paid Amount" (cumulative) on invoices + re-send invoice | the same Invoice ID tracks all sends |
| BL-7 | Biller | payment Time + Payment Method on patient payments | each payment is fully auditable |

## Primary Workflow
1. **Charge Capture & Coding** — signed note feeds Encounter#, DOS, CPT, ICD-10. Statuses: **Pending Coder Review → Approved to Post → Internal Scrubbing**. Coder reviews/edits codes; favorited codes bubble to top of picker.
2. **Claims List** — Record ID, Assigned To, statuses incl. **No Response**; select rows → **Bulk Assign / Bulk Submit**. Per-claim actions: View Claim / Update / **Submit** / **CMS 1500 form** / Assign / **Appeal** / **Corrected Claim**.
3. **Submit vs Corrected Claim** — "Submit/Resubmit" = fresh submission; **Corrected Claim** = distinct flow filed with **CMS frequency-7** resubmission code.
4. **Remits** — EDI auto-sync from clearinghouse (default); EDI upload + **PDF upload** + **Add Remit Manually**. Manual form: **Billing Provider auto-populated and locked** (HUPC = static).
5. **AR Management** — two sub-tabs: **TFL Alerts** + **Work Assignment**. Columns = Claims List + TFL Deadline / Next Action Date / Original (Claim) Date / **Attempts** (touches per claim). Work Assignment lists unassigned claims for bulk or individual assignment.
6. **Patient Invoicing** — Invoice list: Invoice ID / Invoice Date / Patient Name / Date of Service / Statement Date / Total Amount / **Paid Amount** / Amount Due / Status. **Resend Invoice** action (same Invoice ID; communication log records each send).
7. **Patient Payments** — record Date + **Time**, **Payment Method** (Check / Cash / Credit Card), Card Type + last-4 for cards.
8. **Patient Collection** — accumulated balances escalate; cumulative invoice for next encounter gets a NEW ID containing prior balances; collection thresholds tie into Payment Policy.

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-21-09.png` — Billing **Charge Capture** (Encounter#, DOS, CPT, ICD-10; Pending Coder Review / Approved to Post / Internal Scrubbing).
- `HUPC Screens Images/Screenshot from 2026-06-18 15-21-01.png` — Claims List (bulk assign/submit, Record ID, per-claim actions). *(verify filename)*
- Reports → Billing & Collections (Insurance Aging, Denial Analytics, TFL Alerts, Appeal/Refund Tracking) — see module 15.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Charge | encounterId, DOS, CPT[], ICD10[], status (PendingCoderReview/ApprovedToPost/InternalScrubbing), billingProvider |
| Claim | recordId, claimId, providerId, DOS, payerId, location, allowedAmount, status (incl. No Response), submissionType (New/Resubmit/Corrected-freq7), assignedTo |
| Remit | id, source (EDI auto/EDI upload/PDF/Manual), billingProvider (locked), payerId, postedAmount |
| ARItem | claimId, tflDeadline, nextActionDate, originalClaimDate, attempts, assignedTo |
| Invoice | invoiceId (per encounter), invoiceDate, patientId, DOS, statementDate, totalAmount, paidAmount (cumulative), amountDue, status, sendLog[] |
| Payment | invoiceId, date, time, method (Check/Cash/Credit), cardType, last4, amount |

## Business Rules
- Charge statuses gate posting: only **Approved to Post** advances; Internal Scrubbing validates before claim creation.
- **Corrected Claim is a distinct submission flow** (CMS frequency 7) — never reuse "Submit" for corrections.
- Claims "On Hold" section removed; "No Response" status added.
- Remit **Billing Provider auto-populated and static** (HUPC); EDI auto-sync is default, manual/PDF for edge cases; EDI raw content not user-visible (metadata only).
- AR **Attempts** = number of touches/work actions per claim by an agent.
- Invoice **Paid Amount = cumulative paid** against the invoice (not last payment).
- One invoice per encounter; **Resend Invoice keeps the same Invoice ID**; communication log records every send (count + timestamps). Cumulative next-encounter invoice = NEW ID including prior balances.
- Invoices visible per-patient in the patient chart Billing tab.
- Payment methods: Check / Cash / Credit Card (cash future-proofed); record Date **and Time**.
- All refunds/POs/invoices executed by Billing only (Legal/other depts auto-triage to Billing).
- Fee Schedule has two amounts: Insurance Charge (CPT-based) + Self-Pay Amount (Service/Appointment-Type-based). Allowed Amount tab: Procedure Code / Payer / Provider Type / Amount / Effective Dates.
- All edits audit-logged; PHI masking per role.

## MoM Decisions / Deltas (authoritative)
- **Claims List**: bulk-assign + bulk-submit; **Record ID** column; **No Response** status; per-claim actions locked = View / Update / Submit / CMS 1500 / Assign / **Appeal**. "On Hold" section removed.
- **Submit vs Corrected**: "Corrected Claim" added as a distinct status/flow → **CMS frequency 7**; Submit/Resubmit tracked separately (Darren).
- **Remit**: EDI upload + **PDF upload** + **Add Remit Manually**; **Billing Provider auto-populated + static**; EDI auto-sync default.
- **AR Management**: TFL Alerts + Work Assignment sub-tabs; **Attempts column** (touches per claim, Robert); columns add TFL Deadline / Next Action Date / Original Claim Date; bulk + individual assignment.
- **Patient Invoicing**: "Payment" column renamed to **"Paid Amount"** (cumulative). Columns include **Statement Date**.
- **Re-Send Invoice**: same Invoice ID; communication log records each send (Darren's "you only sent one" concern). One invoice/encounter; cumulative next invoice = new ID with prior balances.
- Invoices visible inside patient chart Billing tab.
- **Patient Payments**: **Time** column added; "Payment Card" header → **"Payment Method"**; methods = Check / Cash / Credit Card (cash for future); last-4 + Card Type retained.
- Fee Schedule: Insurance Charge + Self-Pay Amount; Allowed Amount tab labeled "Allowed Amount" (CPT/Payer/Provider Type/Amount/Effective Dates), inline add for new payer+CPT, change audit-logged.
- Payer Master adds Mailing Address + Contact Number ("contact number is must").

## Dependencies
- Clinical documentation (signed note → CPT/ICD for charge capture, module 10).
- Settings → Fee Schedule / Allowed Amount / Payer Master / Payment Policy (module 16).
- Clearinghouse EDI integration (claims out, remits/ERA in).
- Legal refund/PO triages route to Billing (module 13).
- Notifications: invoice/statement/payment-link delivery (module 17).

## Open Items
- Confirm Medent "Default Billing Type" provider-form field mapping (Darren).
- Deposition/legal invoice field set pending Billing finalization (Dr. Mohammed OOO).
- Whether CPT-by-time billing pulls start/end from the note vs scheduled time.

## Acceptance Criteria
- [ ] Charge capture shows Encounter#, DOS, CPT, ICD-10 with the three review statuses.
- [ ] Claims List supports row-select bulk assign + bulk submit; Record ID + No Response present.
- [ ] Corrected Claim files with CMS frequency-7, distinct from Submit/Resubmit.
- [ ] Remit screen accepts EDI auto-sync, EDI upload, PDF upload, and manual entry with locked Billing Provider.
- [ ] AR shows TFL Alerts + Work Assignment with an Attempts column.
- [ ] Invoice shows cumulative "Paid Amount" and Statement Date; Resend keeps same Invoice ID + logs each send.
- [ ] Payments record Date, Time, and Payment Method (Check/Cash/Credit).
- [ ] All billing actions audit-logged; PHI masked per role.

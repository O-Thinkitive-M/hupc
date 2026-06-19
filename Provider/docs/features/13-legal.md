# Legal — Provider Portal Feature Spec

## Purpose
- Legal module in Harmony EMR covering **Delinquents** (collections) and **Depositions** workflows.
- Links legal records to patient accounts (Legal sub-section in Patient Charting); routes all money actions (refunds/POs/invoices) to Billing via auto-triage — **Legal handles no payments**.

## Actors / Roles
| Role | Capability |
|------|------------|
| Legal Dept (Astor/Krystal) | Create deposition requests, set status, reschedule, decide refunds, manage delinquents |
| Office Management | Receives auto-triage to block provider calendar |
| Provider | CC'd on Office Mgmt triage (gets location/link/time; no separate follow-up) |
| Biller | Executes refunds/POs/invoices from Legal triage |
| Admin | Demand/complaint letter templates; status config |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| LG-1 | Legal | a Deposition Request list with locked columns | I track requests through to actual depositions |
| LG-2 | Legal | initial request → status change → actual deposition | only confirmed requests become depositions |
| LG-3 | Legal | a refund decision pop-up on cancel/reschedule | refunds route to Billing without Legal touching money |
| LG-4 | Legal | reschedule with audit log + color-coded marker | I spot reschedules without opening each record |
| LG-5 | Legal | demand letter + complaint letter templates | delinquent collections follow standard letters |
| LG-6 | Legal | bulk assign + export on delinquents | I work the collection queue efficiently |

## Primary Workflow
### Depositions tab
1. New email/response from attorney → **New Deposition Request** → form (Case Number, Provider, Source, Urgency, Law Firm, Attorney Name, Deposition Date, Time Slot, Deposition Type, Testimony Hours, Amount, Payment Status; **Patient Name + Account Number** searchable selector by **DOB**).
2. Record starts as **Initial Request**; status change converts it to an **actual deposition**; auto-links to patient account (Legal sub-section in Patient Charting).
3. Per-row actions: Attach PO / View PO / View Invoice / Generate Invoice (Generate active for Unpaid).
4. **Payment Status** = Paid / Unpaid / PO Received; **Payment Date** auto-captured on status change. Upload Payment Receipt (Paid) parallels PO upload (PO Received).
5. **Deposition Status** = Completed / In Progress / On Hold / Cancelled / Rescheduled.
6. **Reschedule**: edit date + time slot; audit log keeps all prior dates; **color-coded dashboard marker** for rescheduled.
7. On Paid OR PO Received → **auto-triage to Office Management** (provider / date / time / type / link or address) to block calendar; **Provider CC'd**; OM reply auto-flows to CC'd provider (email-based).
8. On future cancel/reschedule with payment received → **Refund Decision Pop-Up** (refund / take charge again / adjust as prior; option to create a **triage to Billing**). Billing executes; Legal never processes payments.

### Delinquents tab
1. Delinquent (collections) queue; statuses: **Pre-Demand / Demand-Sent / Lawsuit-Filed / MSA-Pending / MSA-Partial / SSA-Complete / SSA-Pending / Closed**.
2. **Bulk assign** + **Export**; per-row actions incl. Send Reminder, Send to Legal.
3. Generate **Demand Letter** and **Complaint Letter** from templates.
4. **Mutual Agreement** (pre-lawsuit installment plan) vs **Stipulated Agreement** (post-lawsuit settlement / mediations) — payment plan amounts decided by Legal w/ Dr. Mohammed/attorney; collection executed by Billing.

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-21-09.png` — Legal **Delinquents** list (statuses, bulk assign, export). *(shared billing/legal capture — verify tab)*
- Deposition Request list + new-request form + per-row action menu (verify exact filename in legal capture set).

## Data Entities
| Entity | Key fields |
|--------|-----------|
| DepositionRequest | caseNumber, provider, source, urgency, lawFirm, attorneyName, depositionDate, timeSlot, depositionType (Virtual: meetingLink / In-Person: address), testimonyHours, amount, paymentStatus (Paid/Unpaid/PO Received), paymentDate, depositionStatus, patientId (DOB selector), accountNumber, priorDates[] |
| DepositionInvoice | lawFirm, attorney, contact, dueDate, address, description, qty, rate, total (provider/patient names NOT on outbound) |
| Delinquent | patientId, balance, status (Pre-Demand…Closed), assignedTo, lettersSent[], agreementType (Mutual/Stipulated) |
| LetterTemplate | type (Demand/Complaint), body, mergeFields |
| Triage | subject, reason/category, assignedTo (dept), CC, description (editable), priority, dueDate |

## Business Rules
- Deposition records start as Initial Request; status change converts to actual deposition.
- All depositions are for already-enrolled patients; linked to account via Legal sub-section in Patient Charting; identifier preference = **DOB**.
- Case Reference Number = deposition case number (manual, from attorney docs).
- Payment Date auto-captured on status change.
- **7-working-day refund window**: no refund if reschedule <7 working days before deposition (Paid/online). PO Received: refund rule does NOT apply (PO paid post-hearing).
- Refund Decision Pop-Up triggers when payment received + status Cancelled/Rescheduled + future date.
- **Legal processes NO payments** — refunds/POs/invoices auto-triage to Billing.
- Reschedule: keep full date history (no overwrite) + color-coded marker.
- Auto-triage to Office Mgmt on Paid/PO Received; Provider CC'd; same info (location/link/time) shared with both; email-based notification (Astor preference).
- Outbound deposition invoice must NOT show Provider Name / Patient Name (company/Harmony side only).
- Mutual = pre-lawsuit; Stipulated = post-lawsuit; plan amounts by Legal (w/ Dr. Mohammed/attorney); money execution by Billing.
- All status/date changes audit-logged; PHI masked per role.

## MoM Decisions / Deltas (authoritative)
- **(Jun 8)** Demand letter + complaint letter templates requested; under HUPC attorney (Jack) review (~1 week); Shirley to deliver once approved.
- **(Jun 9, Deposition Workflow)** Deposition Request list = 12 columns (locked). Initial Request → status change → actual deposition.
- Depositions linked to patient account; **new Legal sub-section in Patient Charting**; identifier = **DOB**.
- Patient Name + Account Number = searchable selector on deposition form.
- Payment Status = Paid/Unpaid/PO Received; **Payment Date** auto-captured on status change.
- Per-row actions: Attach PO / View PO / View Invoice / Generate Invoice.
- Upload Payment Receipt (Paid) parallels PO upload (PO Received).
- **Reschedule** with audit log of prior dates + **color-coded marker**.
- **Deposition Status** = Completed / In Progress / On Hold / Cancelled / Rescheduled ("On Hold" not "Pause").
- **7-working-day refund window**; PO Received exempt; Refund Decision Pop-Up (refund / take charge again / adjust as prior).
- **Refunds executed by Billing**; auto-triage to Billing; Legal handles no payments.
- Deposition Type Virtual (meeting link) / In-Person (address).
- Auto-triage to Office Mgmt on Paid/PO Received (provider/date/time/type/link-or-address); triage auto-populated (Subject/Reason/Category/Assigned-to/CC/Description editable; Priority+Due Date manual); **Provider CC'd**; OM reply flows to CC'd provider; email-based.
- Outbound deposition invoice = company side only (no provider/patient name); final invoice fields pending Billing finalization.
- **Mutual = pre-lawsuit; Stipulated = post-lawsuit** (terminology locked); plans by Legal; collection by Billing.
- Delinquent flow scheduled for design (Jun 10) with placeholder templates until final copies arrive.

## Dependencies
- Patient Charting Legal sub-section (module 08/09).
- Billing (refunds/POs/invoices/collections execution, module 12).
- Triage / task system → Office Management + Provider CC (module 17 notifications).
- Settings → letter templates, deposition statuses (module 16).
- Scheduling (block provider calendar on triage).

## Open Items
- Demand + complaint letter final templates pending attorney (Jack) approval (~1 week).
- Deposition invoice field set pending Billing finalization (Dr. Mohammed OOO).
- Mutual/Stipulated payment plan: not currently traceable in EHR (internal) — confirm whether to track.

## Acceptance Criteria
- [ ] Deposition Request list shows the 12 locked columns; New Request form has DOB-searchable patient selector.
- [ ] Status change converts Initial Request → actual deposition and links to the patient's Legal sub-section.
- [ ] Payment Date auto-captures on status change; receipt/PO uploads available per status.
- [ ] Reschedule keeps prior dates and shows a color-coded marker.
- [ ] Refund pop-up fires on future cancel/reschedule with payment; routes to Billing.
- [ ] Auto-triage to Office Mgmt on Paid/PO Received with Provider CC'd.
- [ ] Delinquents tab shows the 8 statuses, bulk assign, export, and demand/complaint letter generation.
- [ ] Outbound deposition invoice omits provider/patient names.
- [ ] All status/date changes audit-logged.

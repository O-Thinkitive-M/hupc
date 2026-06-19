# Billing & Payments — Patient Portal Feature Spec

## Purpose
- Let patients view invoices and balances and pay from the portal.
- Support payment methods including **Check**, Cash, and Credit Card.
- Auto-charge copay, surface cumulative **Paid Amount**, and support payment plans.

## Actors / Roles
| Actor | Capability |
|-------|-----------|
| Patient (Adult) | View invoices, pay, see balance/plan. |
| Guardian | Same for the active child account (financial-responsible party rules apply). |
| Billing / Legal | (Backend) Create/manage payment plans; execute payments/refunds. |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| BIL-1 | Patient | to see my invoices in a billing tab | I know what I owe |
| BIL-2 | Patient | to pay by check/cash/card | I use my preferred method |
| BIL-3 | Patient | copay auto-charged | check-in is faster |
| BIL-4 | Patient | to see cumulative Paid Amount | I track what I've paid |
| BIL-5 | Patient | to follow a payment plan | I pay over time |

## Primary Workflow
1. Patient opens the **Billing tab** → invoice list with columns incl. Invoice ID / Invoice Date / Date of Service / **Paid Amount** / Balance.
2. Patient selects an invoice → chooses a **payment method**: **Check / Cash / Credit Card**.
3. **Check** payment captures: **Bank Name / Branch Name / Check Number / Account Holder Name** (no additional fields).
4. **Copay** is **auto-charged** for the appointment based on eligibility/self-pay (surfaced at booking, see 03).
5. **Paid Amount** shows cumulative paid against the invoice (not the last payment).
6. Re-sent invoices keep the **same Invoice ID** within the same encounter (resend action; communication log counts sends).
7. **Payment plans** (created/managed by Legal/Billing) let the patient pay installments; the patient views plan status/balance.

## Screens
No screen capture provided; per SRS/MoM the expected UI:
- Billing tab: invoice list with Invoice ID, Date, Date of Service, **Paid Amount**, Balance, Pay action.
- Payment dialog with method selector; **Check** shows Bank/Branch/Check Number/Account Holder fields.
- Copay/receipt confirmation.
- Payment-plan view: total balance, installments, next due, status.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Invoice | invoiceId, invoiceDate, dateOfService, paidAmount (cumulative), balance, encounterId |
| Payment | paymentId, invoiceId, method (Check/Cash/Credit Card), amount, date |
| CheckDetails | paymentId, bankName, branchName, checkNumber, accountHolderName |
| PaymentPlan | planId, patientId, totalBalance, installmentCount, startDate, firstAmount, status |
| Copay | apptId, amount, source (eligibility/self-pay), autoCharged |

## Business Rules
- Payment methods supported: **Check / Cash / Credit Card** (cash for future-proofing).
- Check captures Bank Name / Branch Name / Check Number / Account Holder Name only.
- "Payment Card" label is "Payment Method"; **"Payment" column renamed to "Paid Amount"** (cumulative).
- Copay auto-charged per eligibility/self-pay.
- Same Invoice ID retained across re-sends within an encounter; communication log tracks resend count.
- Payment plans created/managed by **Legal** (in consultation with Dr. Mohammed); **Billing executes** payments/refunds.

## MoM Decisions / Deltas (authoritative)
- **Patient portal supports Check payment method** with Bank/Branch/Check Number/Account Holder fields (MoM item 27).
- **"Payment Card" → "Payment Method"; methods = Check / Cash / Credit Card** (MoM item 23).
- **Invoice "Payment" column renamed to "Paid Amount"** (cumulative) (MoM item 20).
- **Re-send invoice keeps the same Invoice ID; communication log counts sends** (MoM item 21).
- Payment plans decided/managed by **Legal**, executed by **Billing** (MoM legal section).

## Dependencies
- Appointments (03) for copay context and 24h cancellation fee.
- Notifications (10) for invoice/receipt emails (EnGard) + SMS (AWS SNS).
- Billing/Legal modules (staff EMR) for plans and refunds.

## Open Items
- Cancellation-fee amount applied to balance — TBD.
- Online card-payment processor/gateway — confirm.
- Patient-facing payment-plan self-enroll vs view-only — TBD (Legal owns creation).

## Acceptance Criteria
- Billing tab lists invoices with a **Paid Amount** (cumulative) column.
- Check payment captures the four specified fields and records the payment.
- Copay is auto-charged with the correct source amount.
- Re-sent invoices keep the same Invoice ID; sends are logged.
- Patient can view active payment-plan status/balance.

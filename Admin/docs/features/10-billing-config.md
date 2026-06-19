# Billing Configuration — Admin Portal Feature Spec

## Purpose
Configure the financial rules of the practice: the **Fee Schedule** (Insurance Charge + Self-Pay Amount), the **Allowed Amount** tab (contracted rate per CPT + Payer), Legal/Other Fees, and Payment Policies. Second-heaviest Admin module (**144 FE hours**).

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin / Billing Admin | Configure fee schedule, allowed amounts, policies |
| Billing Agent | Consumes rates during claims/AR |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| BC-1 | billing admin | set Insurance Charge per CPT per provider type with date ranges | claims charge correctly |
| BC-2 | billing admin | set a Self-Pay Amount by service/appointment type | self-pay patients are billed right |
| BC-3 | billing admin | maintain Allowed Amounts (CPT+Payer+Provider Type) with effective dates | contracted rates are tracked |
| BC-4 | billing admin | add new insurance/CPT inline from Allowed Amount | maintenance is fast |
| BC-5 | admin | configure Legal/Other Fees and Payment Policies | edge-case billing is covered |

## Primary Workflow
1. Admin opens **Billing** settings (screen 15-39-12).
2. **Fee Schedule:** per CPT per provider type, set **Insurance Charge** with From/To dates; set **Self-Pay Amount** keyed by **service/appointment type** (new/follow-up/testing).
3. **Allowed Amount tab:** add rows — Procedure Code / Payer / Provider Type / Amount / From Date / End Date; add new payer/CPT inline.
4. Configure **Legal / Other Fees** (e.g., no-show, records, legal).
5. Configure **Payment Policy** by service type (self-pay before appt; co-pay timing; deductibles; testing).
6. Save; all changes audit-logged; rates flow to scheduling/claims.

## Screens
- Billing: `Screenshot from 2026-06-18 15-39-12.png`

## Data Entities
| Entity | Key Fields |
|--------|------------|
| FeeScheduleEntry | cptCode, providerType, **insuranceCharge**, fromDate, toDate |
| SelfPayRate | serviceType/appointmentType, **selfPayAmount** |
| AllowedAmount | procedureCode, payerId, providerType, amount, fromDate, endDate |
| FeeOther | type (legal/no-show/records), amount |
| PaymentPolicy | serviceType, timing rule, blockingThreshold |

## Business Rules
- **Fee Schedule holds TWO amounts:** Insurance Charge (CPT-based) AND Self-Pay Amount (service/appointment-type-based) [MoM §12].
- Self-pay charges are by **service type (new/follow-up/testing), not CPT time** [MoM §12].
- **Allowed Amount** = contracted rate per CPT + Payer + Provider Type with effective dates; label stays "Allowed Amount" (not "Contracted Rate") [MoM §13].
- Inline add of **new insurance + new CPT** from the Allowed Amount tab [MoM §13].
- Rate varies by provider type (MD vs APRN/PA; APRN=PA, MD differs) (SRS §16.3).
- Payment policy by service type: self-pay before scheduling; co-pay before/day-of; deductibles billed post-EOB (SRS §16.1).
- Pending-balance blocking threshold $100 ($200+ pending confirmation) (SRS §16.2).
- **All fee/allowed-amount changes are audit-logged** [MoM §13].

## MoM Decisions / Deltas (authoritative)
- [x] Fee Schedule: Insurance Charge AND Self-Pay Amount (service-type-based) [MoM §12].
- [x] Self-pay categories = appointment type / service type [MoM §12].
- [x] Allowed Amount tab labeled "Allowed Amount" [MoM §13].
- [x] Allowed Amount fields: Procedure Code / Payer / Provider Type / Amount / Effective Dates [MoM §13].
- [x] Inline add for new insurance + CPT [MoM §13].
- [x] Allowed-Amount changes audit-logged [MoM §13].

## Dependencies
- Module 07 (CPT, Payer Master) · Module 04/05 (provider type) · Module 09 (self-pay keyed to appointment/service type) · Module 13 (audit) · Claims/AR (consumers).

## Open Items
- Final pending-balance blocking threshold ($100 vs $200+).
- Legal/Other Fee type list not enumerated.
- Whether discount (module 09) interacts with self-pay amount.

## Acceptance Criteria
- [ ] Fee Schedule supports Insurance Charge (CPT/provider-type, date-ranged) + Self-Pay (service-type).
- [ ] Allowed Amount tab with CPT/Payer/Provider Type/Amount/effective dates + inline payer/CPT add.
- [ ] Payment policies configurable per service type.
- [ ] Rate varies by provider type.
- [ ] All billing-config changes audit-logged.

# HUPC / Harmony EMR — Provider Portal Feature Index

> Migration project: Medent → custom-built **Harmony EMR** (Thinkitive build).
> **Authority rule:** the **QA MoM V2** (latest) OVERRIDES the SRS on any conflict. Every feature file carries an explicit "MoM Decisions / Deltas (authoritative)" section.

## Module Map (Provider Portal — 11 nav modules)

| # | Module (nav) | File | One-line purpose |
|---|--------------|------|------------------|
| — | Index / glossary | `00-feature-index.md` | This file: module map, glossary, roles, cross-cutting concerns |
| 1 | Auth / RBAC | `01-auth-rbac.md` | Login (email/phone), 2FA OTP, 30-min session timeout, roles & permissions, PHI config |
| 2 | Dashboard | `02-dashboard.md` | **Chart Central** — KPI stat cards, appointments, pending/urgent items, results, messages |
| 3 | Leads | `03-leads.md` | Lead sources, leads worklist, follow-up scheduling, convert-to-patient with dup-check |
| 4 | Patients (Registration) | `04-patient-registration-onboarding.md` | Registration sources, patient-type, guardian/financial responsibility, Add-Patient form |
| 5 | Patients (Chart) | `05-patients-chart.md` | Patient list (4 status tabs), patient chart tabs, face sheet, clinical overview, billing tab |
| 6 | Scheduling | `06-scheduling-appointments.md` | Availability, block days, appointment types, 3-mode booking, calendar views, eligibility |
| 7 | Groups | `07-groups.md` | Group scheduling + group therapy notes (group note + per-participant individual notes) |
| 8 | Referrals | `08-referrals.md` | Referral In/Out, fax + email sub-tabs, create-referral drawer, ROI gate, ICD-10 |
| 9 | Communication | `09-communication.md` | **Message Central** — SMS (AWS SNS) / email / fax / patient requests, dept routing *(authored separately, listed for completeness)* |
| 10 | Billing | `10-billing.md` | Claims, remits, AR, invoicing, payments, fee schedule, allowed amount *(other agent)* |
| 11 | Legal | `11-legal.md` | Depositions, delinquent/collections, demand letters, payment plans *(other agent)* |
| 12 | Reports | `12-reports.md` | Operational, financial, lead-attribution, AR reports *(other agent)* |
| 13 | Settings | `13-settings.md` | Users/roles, providers, locations, PHI config, appointment types, payers, fee schedule, alerts *(other agent)* |
| 14 | Patient Portal | `14-patient-portal.md` | Patient-facing login, rescheduling, requests, history, payments *(other agent)* |
| 15 | Call Widget | `15-call-widget.md` | RingCentral call: recording, forward, incoming-call create-patient *(other agent)* |
| 16 | Prior Auth / Orders | `16-prior-auth-orders.md` | Prescriptions (DrFirst), lab/imaging, prior authorization *(other agent)* |
| 17 | Integrations | `17-integrations.md` | Vendor integration map (DrFirst, Google Meet, SR Fax, AWS SNS, EnGard, Quest, SignNow) *(other agent)* |

> Files **01–09** authored by this agent. Files **10–17** authored by another agent; listed here so the index is complete.

## Glossary

| Term | Meaning |
|------|---------|
| **Chart Central** | Provider schedule + medical-records hub (Dashboard landing). Fax inbox sits under Chart Central / Medical Records. |
| **Message Central** | Patient communication hub (SMS / email / fax / requests) with department-level message segregation. |
| **MRN** | Medical Record Number (patient identifier). |
| **EOB / Remit** | Explanation of Benefits / Remittance (auto via EDI, or manual PDF upload). |
| **TFL** | Timely Filing Limit (claims deadline tracked in AR). |
| **ROI** | Release of Information — signed consent required before outbound referral. |
| **PHI** | Protected Health Information — phone / email / SSN masked per role, reveal-by-click + audit. |
| **10DLC** | 10-Digit Long Code SMS campaign registration (AWS SNS). |
| **EPCS** | Electronic Prescribing of Controlled Substances (DrFirst, medication providers). |
| **Self-Pay** | Patient pays out of pocket; discounts and self-pay fee amounts apply only to these patients. |
| **Macro** | Chart-populated text snippet (e.g., diagnosis on group notes pulls via macro, not free text). |

## Roles (RBAC)

| Role | Scope |
|------|-------|
| **Admin** | Full system + Settings (users, roles, PHI config, masters). |
| **Provider** | Clinical: charting, notes, orders, scheduling, sign-and-lock. (Psychiatric / medication MD-APRN **or** Therapy/Counseling buckets.) |
| **Agent** (Office / Front Desk) | Scheduling, leads, registration, request triage, fax processing. |
| **Biller** | Claims, remits, AR, invoicing, payments, eligibility; **all money handled by billing only**. |
| **Guardian** | Patient-portal proxy for a minor / non-consenting adult (separate file 14). |

> **Users = everyone using the EMR** (providers, office assistants, billing agents, employees) — MoM V2 §6.

## Cross-Cutting Concerns (apply throughout all modules)

| Concern | Rule |
|---------|------|
| **PHI masking** | Phone / Email / SSN masked by default; **reveal-by-click**, scoped **per role type** via Settings → PHI Configuration; every reveal **audit-logged**. |
| **Audit log** | Every reveal, edit, status change, invoice resend, allowed-amount change, and alert-note inactivation is recorded with actor + timestamp. |
| **RBAC** | Permissions per role; new roles can **copy permissions from an existing role**. |
| **Status registry** | Color-coded statuses across leads, claims, appointments, referrals, payment plans; consistent palette for fast triage. |
| **i18n / Preferences** | Preferred Language captured per provider + patient; drives provider-filter and communication channel. |
| **Money segregation** | Refunds and all payment processing executed by **Billing department only** (never Legal/Provider). |
| **Department routing** | Patient-portal requests default-route to **Office Management** for triage, then forwarded to the correct department. |

## Open Items (cross-cutting)

- Landline-only OTP delivery for older patients without mobile — unresolved.
- Guardian using one email/phone for multiple child accounts — **locked**: email & phone are unique per account; at least one required.
- Social History full template — deferred to end of project (HUPC to send reference template).

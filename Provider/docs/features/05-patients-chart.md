# Patients & Patient Chart — Provider Portal Feature Spec

## Purpose
Central patient registry (status-segmented list) and the comprehensive patient chart: face sheet, clinical overview, vitals, orders/prescriptions, history, documents, triages, notes, billing, account info, insurance/pharmacy, plus a collapsible clinical side panel.

## Actors / Roles
| Role | Use |
|------|-----|
| Provider | Clinical charting, notes, orders, vitals, history |
| Agent | Demographics, requests, documents, portal activation |
| Biller | Billing tab, invoices, claims, ledger, payment plans, prior auth |
| Admin | Full access; status transitions |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| PAT-1 | User | a patient list split by status | I find the right cohort |
| PAT-2 | Provider | a face sheet + clinical overview | I get patient context fast |
| PAT-3 | Provider | vitals, problem list, allergies, history | I document and review clinically |
| PAT-4 | Biller | invoices, claims, ledger, prior auth in chart | I handle money per patient |
| PAT-5 | Agent | inactivate/archive/mark deceased + restore | lifecycle is managed |
| PAT-6 | User | a collapsible side panel of key clinicals | context follows me across tabs |

## Patient List — 4 Status Tabs
| Tab | Columns (key) |
|-----|---------------|
| Active | Name, MRN, Source, Phone, Email, DOB, Last Appointment, Primary Provider, Actions (Inactivate/Archive/Mark Deceased) |
| Inactive | + Inactivated By, Inactivation Date, Reason, **Send Activation Link** |
| Archived | + Archived By, Archive Date, Reason, **Restore** |
| Deceased | + Date Updated, Reason |

## Primary Workflow
1. User opens **Patients** → selects status tab → searches/filters → opens a patient.
2. Chart opens on **Face Sheet** with utility buttons (Print Chart, Edit Demographics, Copy Demographics, Send Portal Activation Link).
3. Navigate chart tabs (below); **collapsible side panel** stays pinned with key clinicals.
4. Provider documents in Clinical Overview / Vitals / Orders / Progress Notes / History.
5. Biller works Billing tab (invoices, claims, ledger, payment plans, prior auth).
6. Lifecycle: Inactivate / Archive / Mark Deceased (Restore from Archived).

## Chart Tabs / Sections
- **Face Sheet**: Practice Details, Insurance, Guardian Info, Activity Log, Referral History, Balance, Task, Patient ID, Tags/Notes/Alert Notes/Patient Balance.
- **Clinical Overview**: Problem List, Allergies, Medications, Social History, Past Medical, Appointments, Past Surgical, Screening Forms, Forms, Documents, Triage.
- **Vitals** (NEW): Date, Time, Height, Weight, BMI, BP, Pulse + per-vital notes.
- **Orders**: Prescriptions (DrFirst iframe), Lab, Imaging.
- **Progress Notes**: Date+Time, Supervising/Rendering Provider, Type, Signed, Dx/HPI, ICD, CPT, Treatment Plan, Status.
- **History** (5): Psychiatric, Medical, Surgical, Family, Social.
- **Requests / Triages / Documents / Notes & Alert Notes**.
- **Legal panel** + **Deposition panel** (per-patient).
- **Billing**: Invoices, Claims, Ledger, Credit Balance, Payment Plans, Prior Auth.
- **Account Info** + **Insurance** (Active/History) + **Pharmacy** (NEW).

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-17-15.png` — Patient list with status tabs / patient chart entry.

## Data Entities (selected)
| Entity | Key fields |
|--------|-----------|
| Patient status | Status, By, Date, Reason |
| Vital | Date, Time, Height, Weight, BMI, BP, Pulse, Note |
| Problem | Name, Onset, Status, Note |
| Allergy | Allergy, Type, Reaction, Severity, Onset, RecordedBy, Note |
| Alert Note | Note, VisibleAt, CreatedBy, CreatedTo, Status, InactivatedBy |
| Pharmacy | Primary, Secondary, Mail Order (City + Name) |

## Business Rules
- **Inactive patients cannot be scheduled** (scheduling selector filters to Active only).
- PHI masked by default on chart; reveal-by-click + audit.
- Alert Note inactivation records who inactivated (hover reveals).
- Medications live under **Orders → Prescriptions** (DrFirst iframe, auto-fetched recent + past meds).
- Paper EOB upload available via chart (Add Insurance Payment / Add Patient Payment).

## MoM Decisions / Deltas (authoritative)
- **4 status tabs** Active/Inactive/Archived/Deceased with per-status columns + Restore + Send Activation Link (Jun 15 §12, §14).
- **Inactive patients BLOCKED from scheduling** (Jim's lock, Jun 15 §13).
- **NEW Vitals** with per-vital notes (§22); **Medications → Prescriptions under Orders** via DrFirst iframe (§23).
- **NEW** Patient Request (§26), Legal panel (§27), Deposition panel (§28), Documents (§29), Triages w/ communication thread (§32), Pharmacy section (§38).
- **Progress Note + Appointments + Referrals + 5 History sections** (§30); Social History full template **deferred** (§31).
- Billing tab: **NO Add Insurance Payment on Claims action menu**; KEEP Add Insurance/Patient Payment in chart for **paper EOB upload** (Darren, §35).
- **Collapsible side panel** (Vitals/Allergies/Meds/Problems/Past Encounters) pinned (§39).

## Dependencies
- Registration (file 04), Scheduling (file 06), Referrals (file 08), Billing (file 10), Legal (file 11), Orders/Prior-Auth (file 16).

## Open Items
- Social History full field list — deferred to end of project.
- EOB upload UX (fields, size limits, virus scan) — to revisit with payment screens.

## Acceptance Criteria
- Patient list shows 4 status tabs with correct columns and actions; Restore works on Archived.
- Inactive patients never appear in the scheduling patient-selector.
- Face sheet renders all sections + 4 utility buttons.
- Vitals captures per-vital notes; Prescriptions loads DrFirst iframe with auto-fetched meds.
- Collapsible side panel persists across chart tabs.
- All chart PHI masked by default with audit-logged reveal.

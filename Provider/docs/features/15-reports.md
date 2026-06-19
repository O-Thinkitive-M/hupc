# Reports — Provider Portal Feature Spec

## Purpose
- Reporting hub in Harmony EMR grouped into three categories: **Marketing Reports**, **Billing & Collections**, **Operational & Clinical**.
- Category sidebar + report table with date-range filter and export.

## Actors / Roles
| Role | Capability |
|------|------------|
| Admin / Manager | All report categories; export |
| Biller / AR Lead | Billing & Collections reports |
| Marketing | Referral-source attribution reports only (scoped) |
| Provider | Own productivity report (read) |
| Agent | Operational reports per permission |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| RP-1 | Manager | a category sidebar + report table | I navigate report types quickly |
| RP-2 | Any | a date-range filter (e.g., Last 3 Months) with Apply/Reset | I scope the data window |
| RP-3 | Any | an Export button | I pull the data out for analysis |
| RP-4 | Manager | provider-wise productivity (new vs established, in-person vs telehealth) | I measure provider output |
| RP-5 | Marketing | patient-origin geography by zip/proximity | I assess marketing ROI by location |

## Primary Workflow
1. Open **Reports** → left **category sidebar** with collapsible groups: Marketing Reports / Billing & Collections / Operational & Clinical.
2. Select a report → right pane shows report **title + record count**, a **Date Range** dropdown (e.g., "Last 3 Months") with **Apply / Reset**, and a **data table** (paginated, rows-per-page).
3. **Export** (top-right) downloads the current filtered result.

### Report catalog (per screen)
| Category | Reports |
|----------|---------|
| Marketing Reports | Website Form Activity, Conversion Report, New vs Unique Patient, Location Performance, Referral Source, Opt-In Segmentation |
| Billing & Collections | Insurance Aging, Patient Balance, Denial Analytics, Timely Filing Alerts, Under/Overpayment, Appeal Tracking, Agent Productivity, Refund Tracking, Payment Plans, Unapplied Payments, Error Logs |
| Operational & Clinical | No-Show Report, Cancellation Report, No Therapy Appts, All Appointments |

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-21-17.png` — Reports module: left category sidebar (Marketing / Billing & Collections / Operational & Clinical), "Website Form Activity" report with Date Range (Last 3 Months) + Apply/Reset, data table (Month / Form Starts / Form Completes / Completion Rate % / Opt-Ins / Converted to Patient / Conversion Rate), Export button top-right, pagination.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| ReportDefinition | id, category, name, columns[], permissionScope |
| ReportRun | reportId, dateRangeFrom, dateRangeTo, generatedAt, recordCount |
| MarketingMetric | month, formStarts, formCompletes, completionRate, optIns, convertedToPatient, conversionRate |
| ProviderProductivity | providerId, period, newPatients, establishedPatients, pctInPerson, pctTelehealth |
| PatientOriginGeo | locationId, zip, ringMiles (5/10), patientCount |

## Business Rules
- Three fixed categories; sidebar collapsible per group.
- Date-range filter on every report (preset ranges; Apply/Reset).
- Export downloads the currently filtered/paginated dataset.
- Record-count badge reflects filtered results.
- Permission-scoped: Marketing sees referral-source attribution only; provider productivity is per-provider.
- Aggregations and breakdowns by Category / Service Type / Code (per SRS Section 5).
- PHI masking per role on any patient-level drill-down; report runs audit-logged.

## MoM Decisions / Deltas (authoritative)
- **(Jun 16)** Two new reports added to scope:
  - **Provider-wise productivity report**: count of new patients, established patients, % in-person vs telehealth — per provider for a period.
  - **Patient origin geography report**: by patient mailing address / zip code; count of patients originating within a radius (5/10-mile rings) around each clinic location — marketing ROI driven.
- Patient-origin aggregation is **zip-code based** (Saurabh).
- Provider's anchor location drives reporting/billing aggregation (no location scheduling gate) — productivity/geo reports roll up to anchor location.
- Marketing role: view referral-source attribution reports only.

## Dependencies
- Marketing attribution capture (acquisition source, opt-ins) — module 03/04.
- Billing/AR data (claims, remits, payments, refunds) — module 12.
- Scheduling/encounter data (no-show, cancellation, appointment counts) — module 02.
- Settings → locations + zip data for geo report (module 16).
- Export service (CSV/XLSX).

## Open Items
- Final column sets for new productivity + geo reports (beyond the locked metrics).
- Whether report exports are CSV, XLSX, or PDF — confirm.
- Drill-down behavior from report rows (e.g., aging buckets → filtered task list) — confirm per report.

## Acceptance Criteria
- [ ] Reports page shows the three categories in a collapsible sidebar.
- [ ] Each report shows a date-range filter with Apply/Reset and a record-count badge.
- [ ] Export downloads the current filtered dataset.
- [ ] Provider-wise productivity report shows new vs established and in-person vs telehealth per provider.
- [ ] Patient-origin geography report aggregates by zip / 5–10-mile ring per location.
- [ ] Marketing role sees only referral-source attribution reports.
- [ ] Report runs are audit-logged; PHI masked on drill-downs.

# Settings — Provider Portal Feature Spec

## Purpose
- Centralized 3-level Settings hub in Harmony EMR: a hub of grouped cards → sections → inner tabs.
- Configures Appointment, Clinic, Billing, Templates, Master, Alerts, and Audit Log (provider-portal/admin view).

## Actors / Roles
| Role | Capability |
|------|------------|
| Admin | Full Settings access (all groups) |
| Billing Lead | Billing group (Fee Schedule, Allowed Amount, Legal/Other Fee, Payment Policy) |
| Office Mgmt | Clinic (Locations, Departments, Users), Appointment |
| Provider | Read availability/templates relevant to them |
| All staff | Audit Log per permission |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| ST-1 | Admin | a settings hub of grouped cards | I reach any config in ≤3 levels |
| ST-2 | Admin | user-to-location assignment with Select All | I grant access to all 30–40 locations in one click |
| ST-3 | Admin | PHI Configuration per role (Visible/Reveal Only/Masked) | demographics are masked appropriately |
| ST-4 | Admin | ICD-10/CPT masters + Favorites | universal codes are managed and fast to pick |
| ST-5 | Admin | Alerts config (title/type/trigger/timing/priority) | system alerts fire correctly |

## Primary Workflow
1. Open **Settings** → hub of grouped cards (Level 1).
2. Click a card item → section (Level 2).
3. Section opens inner tabs / detail editor (Level 3).

### Settings hub map (per screen)
| Group | Items (Level 2) |
|-------|-----------------|
| **Appointment** | Availability, Holidays, Appointment Types |
| **Clinic** | Profile, Location, Users, Roles & Responsibility, PHI Configurations, Department, Security Questions |
| **Billing** | Fee Schedule, Allowed Amount, Legal/Other Fee, Payment Policy |
| **Templates** | Visit Note, Macros |
| **Master** | Data Import, ICD-10 Code, CPT Code, Payer |
| **Alerts** | Alert |
| **Audit Log** | Audit Log |

### Key inner-tab behaviors
- **Availability**: per-provider grid (Provider/Availability/Slots/Block Days/Last Updated/Updated By); day-slot editor (date range, weekday checkboxes, start/end, location, time zone, Virtual/In-Person checkboxes); save confirmation popup; **Block Days** overlap → conflict alert with reschedule-task option; **Holidays** (title/date/description).
- **Appointment Types**: segregated by service type (Psychotherapy, Med Management, Psychotherapy+Med Mgmt, Group Therapy, TMS, Other); form = Service Type, Type Name, Duration, **Mode** (In-Person/Telehealth), CPT codes, **Discount**, Attached Forms.
- **Location**: Location Name/ID/Contact/Email/Tax ID/Time Zone/Address/Hours + **Departments** sub-table; user-to-location is assigned **from the user record** (Patients tab removed from location).
- **Users**: single tab for staff + providers; user types = **Staff / Prescribing Provider / Non-Prescribing Provider / Patient**; provider names show credential title; multi-select Location + **Select All**; **Employment Type (W2/1099)** internal-only.
- **Roles & Responsibility**: Role Type + Role; create role with "copy permissions from role **or existing user**"; per-feature Create/Update/View/Delete matrix; Select All on location scope.
- **PHI Configurations**: per-role, per-field (Phone/Email/SSN) → Visible / Reveal Only / Masked; reveal click logged in activity log.
- **Fee Schedule**: Insurance Charge (CPT-based) + Self-Pay Amount (Service/Appointment-Type-based).
- **Allowed Amount**: Procedure Code/Payer/Provider Type/Amount/Effective Dates; inline add new payer+CPT.
- **Legal/Other Fee**: legal + non-legal fee table.
- **Payment Policy**: Advanced Payment, Balance Limit Threshold (hard-stop or follow-up), Manual Payment Link (Email/SMS/Portal).
- **Templates**: Visit Note (Form Builder — drag-and-drop sections/text/dropdowns; concurrent notes + centralized chart data) + **Macros** (Name/Type/Description).
- **Master**: Data Import (Entity/File/Verify → Success/Failed w/ reason); ICD-10 + CPT (Provider Type column removed; Favorites bubble to top); Payer (Name/ID/Client Type/Status + Mailing Address + Contact Number).
- **Alerts**: Alert Title/Type/Trigger Event/Trigger Timing/Priority/Assigned To/Status.
- **Audit Log**: change + reveal history.

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-21-21.png` — Settings hub: cards Appointment / Clinic / Billing / Templates / Master / Alerts / Audit Log with their items (matches map above).
- `HUPC Screens Images/Screenshot from 2026-06-18 15-38-49.png` … `…15-39-25.png` — the 7 settings sub-screens (Availability, Locations/Users, PHI Config, Fee Schedule/Allowed Amount, Templates/Macros, Master codes, Alerts/Audit) — map each to the inner tabs above.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| ClinicProfile | name, contact, email, address, fax, billingAddress, description |
| Location | name, id, contact, contactPerson, email, taxId, timeZone, address, hours; Department[] |
| User | name, title, email, contact, type (Staff/Prescribing/Non-Prescribing/Patient), locations[], role, employmentType (W2/1099) |
| Role | roleType, name, permissionMatrix (CRUD per feature), clonedFrom (role|user) |
| PHIConfig | role, field (Phone/Email/SSN), state (Visible/Reveal/Masked) |
| FeeSchedule/AllowedAmount | cpt, payer, providerType, insuranceCharge, selfPayAmount, effectiveDates |
| Macro / VisitNoteTemplate | name, type, description, body (Form Builder) |
| CodeMaster | code, name, description, status, favorite (ICD-10/CPT) |
| Payer | name, id, clientType, status, mailingAddress, contactNumber |
| Alert | title, type, triggerEvent, triggerTiming, priority, assignedTo, status |

## Business Rules
- **User-to-location inversion**: assign locations from the user record (multi-select + Select All); no per-location user list.
- User types = 4 buckets; only PA/APRN/MD are Prescribing Providers; provider names show credential title.
- Employment Type (W2/1099) internal-only; never on website.
- No location-based scheduling gate; location = anchor for reporting/billing.
- ICD-10/CPT masters are universal — Provider Type column removed; rate logic lives in Allowed Amount; Favorites bubble to top.
- PHI fields (Phone/Email/SSN): Visible/Reveal Only/Masked per role; reveal logged.
- Permissions clonable from a role OR an existing user.
- Block-day overlap on booked appointments triggers conflict alert + reschedule task.
- Diagnosis selection AI-driven (no checkbox diagnosis selector in templates).
- Concurrent notes + centralized chart data enforced via Templates/Form Builder.
- All changes audit-logged.

## MoM Decisions / Deltas (authoritative)
- **(Jun 16, Settings Walkthrough)** User→location assignment inverted (multi-select + Select All); no per-location user list.
- User-type taxonomy = **Staff / Prescribing Provider / Non-Prescribing Provider / Patient**; only PA/APRN/MD prescribe; provider name shows credential title.
- **Employment Type (W2/1099)** tag added (renamed from "Provider Type" field); internal-only.
- **No location-based scheduling validation**; anchor location for reporting/billing only; **Patients tab removed** from location detail.
- **PHI Configuration** approved for v1 (Phone/Email/SSN; Visible/Reveal/Masked); reveal logged; more demographics addable later.
- Permissions clonable from **existing user** (not only role).
- **ICD-10/CPT Provider Type column removed** (universal codes); rate logic at Allowed Amount; import full standard sets (~10,000 ICD + CPT); **Favorites** bubble to top.
- **Diagnosis AI-driven** — no checkbox diagnosis selector.
- **Concurrent notes + centralized chart data** locked; **Form Builder** needed for templates (deferred to dedicated sessions).
- **Fee Schedule** = Insurance Charge + Self-Pay Amount; **Allowed Amount** tab (CPT/Payer/Provider Type/Amount/Effective Dates); **Payer Master** adds Mailing Address + Contact Number.
- **Alerts** section added (Title/Type/Trigger Event/Trigger Timing/Priority/Assigned To/Status).
- Block-day overlap → conflict alert + reschedule task.
- **Legal/Other Fee** renamed (same table used for non-legal charges).
- **Payment Policy** = Advanced Payment / Balance Limit Threshold (hard-stop or follow-up) / Manual Payment Link (Email/SMS/Portal).

## Dependencies
- Drives every module: Appointment Types→scheduling/telehealth (02/11), Fee/Allowed/Payer→billing (12), PHI Config→all PHI masking, Templates/Macros→clinical docs (10), Alerts→notifications (17), Master codes→charge capture (12).

## Open Items
- Confirm Medent "Default Billing Type" provider-form field mapping (Darren).
- Templates structure/rebuild + Form Builder details deferred to dedicated sessions.
- Additional PHI demographics beyond Phone/Email/SSN (later via edit).

## Acceptance Criteria
- [ ] Settings hub shows 7 groups with the items in the map; navigation is ≤3 levels.
- [ ] Users assign locations via multi-select with Select All; no per-location user list.
- [ ] User types are the 4 buckets; provider names show credential titles; W2/1099 internal-only.
- [ ] PHI Config supports Visible/Reveal Only/Masked per role; reveals log to activity log.
- [ ] ICD-10/CPT masters omit Provider Type and support Favorites; Allowed Amount holds rate logic.
- [ ] Alerts config captures all listed fields; block-day overlap triggers a conflict alert.
- [ ] Templates use a Form Builder; concurrent notes + centralized chart data enforced.
- [ ] All Settings changes audit-logged.

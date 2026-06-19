# Leads (Lead Management) — Provider Portal Feature Spec

## Purpose
Capture, track, and convert prospective patients from every inbound channel. Provides a leads worklist with advanced filters, contact logging, follow-up scheduling, and convert-to-patient with duplicate detection — preserving marketing attribution throughout.

## Actors / Roles
| Role | Use |
|------|-----|
| Agent (Office) | Work the leads list, log contacts, schedule follow-ups, convert |
| Admin | Configure sources, view attribution reports |
| Biller | (read) self-pay / insurance context during conversion |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| LEAD-1 | Agent | a worklist of leads with source + status | I prioritize outreach |
| LEAD-2 | Agent | advanced filters (source/status/service/agent/follow-up date) | I focus my queue |
| LEAD-3 | Agent | log each contact (call/SMS/email) with outcome | history is auditable |
| LEAD-4 | Agent | schedule a follow-up with frequency + assignee | nothing is dropped |
| LEAD-5 | Agent | convert a lead to a patient with dup-check | no duplicate charts |
| LEAD-6 | Admin | export the leads list | offline analysis |

## Primary Workflow
1. Lead enters from a source channel (see sources below) → created in **New** status.
2. Agent opens **Leads list** → applies filters/search → selects a lead.
3. **Lead detail view**: personal info, appointment details, insurance, additional info; upload documents per lead.
4. **Log Contact**: choose call / SMS / email → record outcome + description (increments Attempts).
5. **Set Schedule Follow-Up**: choose frequency, assign to staff → a task is created.
6. **Convert to Patient** → system runs duplicate-check:
   - Match found → show matching fields → choose **Create New Patient Account** OR **Update Existing Patient Account**.
   - No match → create new patient (lead source retained for attribution).

## Lead Sources
Website widget (new-patient form), inbound call, **referral** (form/link), fax referral, paid ad, CRM import, insurance-directory, walk-in. "How did you hear about us?" captured for attribution.

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-17-04.png` — Leads list/table: columns, filters, search, Export.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-16-58.png` — Lead detail + Log Contact / Convert-to-Patient view.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Lead | LeadId, Name, DOB, Phone, Email, Source, Status, ReceivedOn, Frequency, Attempts, LastContactedAt, AssignedAgent |
| Contact log | Channel (call/SMS/email), Outcome, Description, Timestamp, Agent |
| Follow-up | Frequency, AssignedTo, TaskId, DueDate |
| Lead document | FileName, Type, UploadedBy, Date |

## Lead Statuses
New → Callback Requested → In Progress → Awaiting Response → Follow up at later date → Converted / Rejected (color-coded). "Retry Limit Reached" indicator when attempts exhausted.

## Business Rules
- Lead **retains original source** through conversion (attribution preserved).
- Inactive/converted patients do **not** revert to Lead status; a new inquiry creates a new lead.
- **Convert-to-Patient** must run duplicate-check (Name+DOB+Mobile primary match key) before creating.
- Insurance company reps cannot directly schedule — they submit via the **lead-management widget only** (no login).
- PHI masked per role on list + detail; reveal audit-logged.

## MoM Decisions / Deltas (authoritative)
- **3-option lead flow simplified to 2 + Set Schedule Follow-Up** (frequency + staff assignment → task created) — Jun 15 §11.
- **New columns**: Frequency, **Attempts** (staff call count), Last Contacted Date+Time, Source, Status, Received On.
- **Log Contact** records call / **SMS** / email with outcome + description.
- **Document upload per lead** added.
- **Convert-to-Patient with duplicate-check**: Create New Account OR Update Existing Account.
- **Insurance reps → lead-management submission path only** (no direct scheduling) — Jun 15 §16.
- Search + filter + **Export** button on list.

## Dependencies
- Patient Registration (file 04) — convert target; mirrors widget form structure.
- Communication (file 09) — SMS/email channels for Log Contact + outreach.
- Referrals (file 08) — fax/form/link referrals create leads.
- Reports (file 12) — lead-source attribution + overdue-rate reporting.

## Open Items
- Follow-up frequency option set (presets vs custom) — to confirm.
- Auto vs manual increment of Attempts on each outreach — to confirm.

## Acceptance Criteria
- Leads list shows Frequency, Attempts, Last Contacted, Source, Status, Received On with color-coded statuses, filters, search, and Export.
- Log Contact records channel + outcome + description and increments Attempts.
- Set Schedule Follow-Up creates a staff-assigned task with the chosen frequency.
- Convert-to-Patient runs dup-check and offers Create New vs Update Existing.
- Converted patient retains the original lead source for attribution.

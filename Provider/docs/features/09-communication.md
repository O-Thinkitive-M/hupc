# Communication (Message Central) — Provider Portal Feature Spec

## Purpose
Unified communication hub for the practice. Six sub-tabs — **Messages, Fax, Triage, Contact Directory, Call Log, Request Management** — let staff send/receive patient SMS (via AWS SNS), view/send faxes (SR Fax), route internal triage tasks, look up directory contacts, review recorded/forwarded calls, and process portal-submitted requests. Department-segregated, PHI-masked, audit-logged.

## Actors / Roles
| Role | Use |
|------|-----|
| Agent (Office / Office Mgmt) | Send/receive SMS, manage faxes, triage routing, answer portal requests |
| Provider | Receive triage, message patients, review call notes, CC on triage |
| Office Management | Default landing point for all portal requests; routes to correct department |
| Admin | Configure SMS templates per department, opt-out lists, directory |
| Department Head / Communicator | Auto-CC on triage; owns department queue |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| COM-1 | Agent | send patient SMS from department templates | messaging is consistent + fast |
| COM-2 | Agent | see only my department's messages | dept-level segregation is enforced |
| COM-3 | Agent | be blocked from sending a duplicate SMS | the patient isn't double-texted |
| COM-4 | Agent | track delivery status + incoming replies | I know what was received |
| COM-5 | Patient | reply STOP to opt out | TCPA/consent is honored |
| COM-6 | Agent | view/send faxes side-by-side | referrals/records are processed |
| COM-7 | Agent/Provider | create + receive triage tasks | work routes to the right team |
| COM-8 | Office Mgmt | receive all portal requests then re-route | concerns reach the right department |
| COM-9 | Agent | review call recordings + transcripts | call context is preserved |
| COM-10 | Agent | mark a message read manually | read state is intentional, not auto |

## Primary Workflow (Outbound SMS)
1. Agent opens **Messages**, scoped to their **department** (segregation enforced).
2. Selects patient → picks a **department-based template** (or free text) → reason category (**Billing / Medication / Other**).
3. System runs **duplicate-send prevention** check; blocks if an identical message is pending/recently sent.
4. System checks patient **opt-out** status; if opted out (STOP), send is blocked.
5. Message sent via **AWS SNS** (10DLC, HIPAA, BAA) → **delivery status** tracked (Sent / Delivered / Failed).
6. Patient reply arrives as **incoming message**; agent **manually marks read** (no auto-read).
7. STOP/keyword reply auto-adds patient to **opt-out** list; future sends blocked.

## Sub-Modules

### Messages / SMS
- Inbound + outbound SMS threads via **AWS SNS** (not RingCentral). Secondary tabs/status filters: Incoming / Outgoing / All.
- **Department-level segregation** — agents see only their department's threads.
- **Department-based templates**; reason categories Billing / Medication / Other.
- **Duplicate-send prevention**; **delivery-status** tracking; **manual read status**.
- **Opt-out (TCPA/STOP)** handling; opt-out list blocks future sends. **Chatbot integration hook** for automated replies. Portal patient↔provider messaging surfaces here.

### Fax
- View / send faxes via **SR Fax** ($68.95/mo ~2,500 faxes). Fax viewer with side-by-side document review (shared with Referrals).
- Status filters (received today bold, older grayed); attach fax to patient chart Documents.

### Triage
- Internal task/message routing to a provider, office, or department. Auto-populated form: Subject / Reason / Category / **Assigned-to** / **CC** / Description (editable).
- Auto-triage sources: refund→Billing, payment received→Office Mgmt, amend-request→original provider. **Provider auto-CC'd** to eliminate redundant follow-up.

### Contact Directory
- Searchable practice directory: name, **Department**, role, **Contact Number / Fax Number**, Direct Messaging / **EPCS** eligibility flag.

### Call Log
- Records from the calling widget: **recordings**, transcripts, Basic Details / Transcript / Note sections, forward-to-agent history.
- **Phone numbers masked** across all views (PHI). Resend/contact count tracked in communication log.

### Request Management
- Portal-submitted requests: columns **Case Number / Department / Status / Priority / Submitted Date / Last Updated**.
- **All requests default-route to Office Management** regardless of patient-selected department; OM reads then re-routes; **reassignment logged**.

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-05.png` — Messages (incoming/outgoing SMS threads).
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-41.png` — Fax viewer.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-45.png` — Triage / Contact Directory.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-47.png` — Request Management / Contact Directory.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-50.png` — Call Log (PHI-masked numbers).
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-54.png` — Request Management.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Message (SMS) | MessageId, PatientId, Department, Direction (In/Out), TemplateId, ReasonCategory (Billing/Medication/Other), Body, DeliveryStatus, ReadStatus (manual), SnsMessageId, SentAt |
| SMS Template | TemplateId, Department, Name, Body, Active |
| Opt-Out | PatientId/Phone, OptedOutAt, Source (STOP keyword), Active |
| Fax | FaxId, Sender/Number, PageCount, ReceivedDate, Status, PatientLink, Annotations |
| Triage | TriageId, Subject, Reason, Category, AssignedToDept, CC[], Description, Status, Source, CreatedBy |
| Directory Contact | ContactId, Name, Department, Role, ContactNumber, FaxNumber, EPCSEligible |
| Call Log | CallId, Direction, MaskedNumber, Recording, Transcript, Note, ForwardedTo, ResendCount, Timestamp |
| Request | CaseNumber, PatientId, Department (informational), Priority, Description, Status, SubmittedDate, LastUpdated, ReassignmentLog[] |

## Business Rules
- **SMS sends via AWS SNS** (10DLC registered, $0.01/SMS, HIPAA + BAA); RingCentral is the call widget only.
- **Department segregation**: agents view/act only within their assigned department's messages.
- **Duplicate-send prevention** blocks identical pending/recent sends to the same patient.
- **Opt-out enforced**: STOP keyword auto-opts-out; opted-out patients cannot be messaged.
- **Read status is manual** — messages are not auto-marked read on view.
- **Delivery status** (Sent/Delivered/Failed) tracked per message.
- Portal patient↔provider messaging + all portal requests **default-route to Office Management**, then re-route with **reassignment audit log**.
- **PHI masking**: phone numbers masked in Call Log and across views; reveal is **audit-logged**.
- Reason categories limited to **Billing / Medication / Other**.
- Triage auto-populated + **provider CC'd**; assigned-to defaults to Office Mgmt where applicable.

## MoM Decisions / Deltas (authoritative)
- **[AUTHORITATIVE] SMS via AWS SNS** locked as vendor — $0.01/SMS, $4 + $50 one-time (10DLC), HIPAA-compliant + BAA confirmed; free push notifications bundled (QA MoM §5). **Overrides any SRS RingCentral-SMS assumption.**
- **[AUTHORITATIVE] Department-level segregation** of messages — agents see only their department.
- **[AUTHORITATIVE] Duplicate-send prevention** required on outbound SMS.
- **[AUTHORITATIVE] Manual read status** — no auto-read on view.
- **[AUTHORITATIVE] Opt-out / STOP (TCPA)** handling + opt-out list enforcement.
- **[AUTHORITATIVE] Portal messaging + Request Management default-route to Office Management** regardless of patient-selected department; OM re-routes with reassignment log (QA MoM §28).
- Department-based **templates** + reason categories (Billing / Medication / Other).
- **Chatbot integration hook** planned for automated messaging.
- Call widget: **recording / forward / incoming-call** handling; numbers masked (QA MoM §1).
- SR Fax locked as fax vendor ($68.95/mo, ~2,500 faxes, pay-as-you-go).

## Dependencies
- Referrals (file 08) — shares Fax viewer + SR Fax channel.
- Patients chart (file 05) — attach fax/message to Documents; patient context.
- Settings (file 16) — department-based SMS templates, opt-out config, directory, PHI masking config.
- Billing / Legal (files 12, 13) — auto-triage sources (refund→Billing, payment→Office Mgmt).
- Patient Portal (file 14) — portal messaging + request submission.
- Integrations (file 17) — **AWS SNS** (SMS), **SR Fax** (fax), RingCentral (call widget), EnGard (email), chatbot.

## Open Items
- Production AWS SNS **organization-level account** for HUPC (separate from Thinkitive dev account) — pending production phase.
- Chatbot integration scope + provider for automated message replies — not finalized.
- Guardian one-email-two-accounts unique-constraint conflict (portal login) — confirmation to Dr. Mohammed pending.
- Final duplicate-send-prevention matching rule (exact body vs. same template/time window) — to confirm.

## Acceptance Criteria
- Outbound SMS sends via AWS SNS with delivery status (Sent/Delivered/Failed) tracked.
- Agents see only their department's messages (segregation enforced).
- Identical pending/recent SMS to the same patient is blocked (duplicate-send prevention).
- STOP reply opts the patient out; subsequent sends are blocked.
- Incoming messages require **manual** mark-as-read.
- Department templates + reason categories (Billing/Medication/Other) available on send.
- Fax viewer supports view/send with patient attach; Call Log shows masked numbers with audit-logged reveal.
- All portal requests/messages land in Office Management first; re-routing writes a reassignment log entry.

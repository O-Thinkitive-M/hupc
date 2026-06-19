# Notifications — Provider Portal Feature Spec

## Purpose
- Outbound patient/staff communications in Harmony EMR across **Email** and **SMS** plus in-app tasks/triages.
- Template-driven, event-key triggered: appointment reminders, booking confirmations, post-session follow-up, billing/invoice, telehealth links, ID-expiry, eligibility callbacks.

## Actors / Roles
| Role | Capability |
|------|------------|
| Admin | Configure templates, event keys, channels, cadence (Settings → Alerts/Templates) |
| Agent / Office Mgmt | Trigger manual sends (payment link, reminders); receive triages |
| Biller | Invoice/statement/payment-link sends; communication log review |
| Provider | CC'd on triages (e.g., deposition Office-Mgmt block) |
| Patient/Guardian | Receives email/SMS; portal messaging |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| NT-1 | Admin | event-keyed templates per channel | the right message fires on the right event |
| NT-2 | Patient | appointment reminders (multi-stage) | I don't miss visits |
| NT-3 | Agent | to send a manual payment link via Email/SMS/Portal | the patient can pay |
| NT-4 | Biller | a communication log of every invoice send | I prove how many times an invoice was sent |
| NT-5 | Admin | duplicate-send prevention + department segregation | patients aren't spammed and routing is correct |

## Primary Workflow
1. An event fires (booking, reminder window, sign-off, invoice, telehealth-link cadence, ID-expiry, eligibility-fail callback) → resolves an **event key**.
2. System selects the **template** for that event key + channel (Email / SMS) and merges patient/encounter fields.
3. **Channel routing**: Email via EnGard; SMS via AWS SNS; Portal message for in-app; in-app task/triage for staff.
4. **Duplicate-send prevention** applied; **department segregation** routes to the correct sending department.
5. Send recorded in the **communication log** (count + timestamps); reveals/PHI audit-logged.
6. Multi-stage cadence for reminders/links (e.g., first 3 days before, second 1–3 hrs before).

### Notification event keys (representative)
| Event key | Channel(s) | Notes |
|-----------|-----------|-------|
| appointment.booked | Email/SMS | booking confirmation |
| appointment.reminder | Email/SMS | multi-stage cadence |
| telehealth.link | Email/SMS/Portal | join link + payment prompt (module 11) |
| session.followup | Email | post-session |
| billing.invoice / invoice.resend | Email | same Invoice ID; logs each send |
| billing.payment_link | Email/SMS/Portal | manual payment link |
| id.expiry | Portal + staff task | approaching ID expiry |
| eligibility.callback | Email/SMS | callback SLA confirmation |
| legal.triage (Office Mgmt) | Email | provider CC'd (module 13) |

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-21-21.png` — Settings hub → **Alerts** card (Alert config) + **Templates** card.
- Communication module screens (`…15-19-xx` capture set) — message threads, SMS, portal messaging. *(verify filenames)*

## Data Entities
| Entity | Key fields |
|--------|-----------|
| NotificationTemplate | id, eventKey, channel (Email/SMS/Portal), subject, body, mergeFields, department |
| NotificationEvent | eventKey, triggerSource, payload |
| Delivery | templateId, patientId, channel, scheduledAt, sentAt, status, dedupeKey |
| CommunicationLog | entityId (invoice/appt), channel, sendCount, timestamps[], outcome |
| Alert (Settings) | title, type, triggerEvent, triggerTiming, priority, assignedTo, status |

## Business Rules
- Templates are event-keyed and per-channel; merge patient/encounter fields.
- **Duplicate-send prevention** to avoid repeat messages on the same event.
- **Department segregation** — sends routed/attributed to the correct department.
- Invoice re-send keeps the same Invoice ID and **logs every send** in the communication log.
- Manual payment link sendable via Email / SMS / Patient Portal.
- Multi-stage reminder/link cadence configurable per appointment type.
- ID-expiry: patient (portal) + internal staff task.
- Eligibility-fail callback: SLA confirmation message; live-agent gated by operating hours.
- All vendors BAA-covered (HIPAA); PHI in messages masked/audit-logged.
- Email volume baseline ~300,000/month (10,000+ encounters × 30–40 touchpoints).

## MoM Decisions / Deltas (authoritative)
- **Email vendor = EnGard** ($10/month unlimited) — **NOT SendGrid**; AWS SES rejected (EnGard already in production). *Supersedes any "SendGrid" reference.*
- **SMS vendor = AWS SNS** ($0.01/SMS, HIPAA-compliant, BAA) — **NOT Twilio/RingCentral**. Test on Thinkitive dev AWS during build; separate HUPC org AWS account for production. *Supersedes any "Twilio/RingCentral" reference.*
- Notifications per encounter ~30–40 touchpoints (booking + reminder + post-session follow-up + billing + lab + support).
- **Manual Payment Link** via Email / SMS / Patient Portal — all three confirmed (Dr. Mohammed + Darren).
- **Multi-stage reminder cadence**: first link 3 days before, second 1–3 hrs before; every link click re-runs the payment prompt (telehealth, module 11).
- **Communication log records every invoice send** (Darren's "you only sent one" concern); same Invoice ID across resends.
- **Settings → Alerts** module: Alert Title / Type / Trigger Event / Trigger Timing / Priority / Assigned To / Status.
- **Legal Office-Mgmt triage** uses **email-based** notification (Astor preference); Provider CC'd; OM reply auto-flows to CC'd provider.
- ID-expiry notification to patient portal + internal task (intervals/channels/templates TBD).
- All vendors have BAA; dev credentials during build → production credentials post-go-live.

## Dependencies
- EnGard (email) + AWS SNS (SMS) integrations.
- Settings → Templates / Alerts (module 16).
- Scheduling (reminder cadence) + Telehealth link delivery (modules 02/11).
- Billing invoice/statement/payment-link + communication log (module 12).
- Legal triage routing + Provider CC (module 13).
- Eligibility callback + ID-expiry triggers (module 14).

## Open Items
- ID-expiry notification intervals/channels/templates — TBD with HUPC.
- Full event-key catalog + per-template copy — to finalize.
- Production AWS org account + EnGard production credentials — post-go-live.
- Chatbot hook (portal) — referenced for Communication; integration point TBD.

## Acceptance Criteria
- [ ] Templates are event-keyed and per-channel (Email via EnGard, SMS via AWS SNS, Portal).
- [ ] Appointment reminders fire on a configurable multi-stage cadence.
- [ ] Manual payment links send via Email/SMS/Portal.
- [ ] Invoice resends keep the same Invoice ID and log each send in the communication log.
- [ ] Duplicate-send prevention and department segregation are enforced.
- [ ] Legal Office-Mgmt triage notifies via email with the provider CC'd.
- [ ] ID-expiry notifies patient portal + creates an internal task.
- [ ] All vendors BAA-covered; PHI masked/audit-logged in messages.

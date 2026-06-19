# Notifications — Patient Portal Feature Spec

## Purpose
- Deliver portal notifications to patients via **email (EnGard)** and **SMS (AWS SNS)**.
- Cover messages/requests, appointments, telehealth links, and billing events.
- Honor patient communication consents.

## Actors / Roles
| Actor | Capability |
|-------|-----------|
| Patient (Adult) | Receive notifications; act on links. |
| Guardian | Receive notifications for the active child account. |
| System | Sends consent-gated email/SMS; logs delivery. |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| NTF-1 | Patient | appointment confirmations/reminders | I don't miss visits |
| NTF-2 | Patient | a telehealth join link by SMS/email | I can join the visit |
| NTF-3 | Patient | alerts when I get a response | I follow up on requests |
| NTF-4 | Patient | billing/invoice notices | I pay on time |
| NTF-5 | Patient | notifications only on consented channels | I'm not over-contacted |

## Primary Workflow
1. A portal event occurs (new message/response, appointment booked/changed, telehealth link, invoice/receipt, ID expiring).
2. System composes the notification and selects channel(s) per the patient's **consents** (email/SMS).
3. **Email sent via EnGard; SMS/OTP sent via AWS SNS.**
4. Appointment confirmations sent via email and SMS; telehealth bookings include the Google Meet link.
5. Delivery is logged (communication log tracks sends, e.g., invoice resends).

## Notification Catalog
| Event | Channels | Notes |
|-------|----------|-------|
| Login OTP | SMS (AWS SNS) | Identity verification (01). |
| Appointment confirmation | Email + SMS | On booking (03). |
| Appointment reminder | Email/SMS | Per consent. |
| Telehealth join link | Email and/or SMS | Google Meet link (05). |
| Request response | Email/SMS | From Office Mgmt/provider (04). |
| Invoice / receipt | Email (+SMS) | Resends keep same Invoice ID (08). |
| ID expiration | Portal + email/SMS | Intervals/templates TBD. |

## Screens
No screen capture provided; per SRS/MoM the expected UI:
- In-portal notifications list/badge with read/unread state.
- Email/SMS templates (branded, accessible, i18n-ready).
- Link to the relevant portal screen from each notification.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Notification | notifId, patientId, eventType, channel, status, sentAt |
| CommunicationLog | refId (invoice/request/appt), channel, sendCount, lastSentAt |
| Template | templateId, eventType, locale, channel |

## Business Rules
- **Email vendor = EnGard; SMS/OTP vendor = AWS SNS** (NOT SendGrid/Twilio).
- Channel selection honors patient consents (SMS only if SMS consent granted; marketing only if opted in).
- Appointment confirmation sent via both email and SMS.
- Communication log records sends (e.g., invoice resend count).
- All patient-facing content is i18n-ready and accessible.

## MoM Decisions / Deltas (authoritative)
- **Email = EnGard, SMS = AWS SNS** (MoM-confirmed vendors; overrides any SRS placeholder vendors).
- **Telehealth link auto-sent via email and/or SMS** based on booking selection (MoM/SRS).
- **Communication log tracks resend count; invoice resends keep same Invoice ID** (MoM item 21).
- OTP delivered via SMS for email+phone login (MoM item 24 + Auth 01).

## Dependencies
- EnGard (email), AWS SNS (SMS/OTP).
- Profile/Preferences (09) for consents.
- Appointments (03), Telehealth (05), Messaging (04), Billing (08) as event sources.

## Open Items
- ID-expiration notification intervals, channels, and templates — TBD with HUPC.
- Reminder cadence/timing for appointments — TBD.
- Failure/retry handling for failed sends — TBD.

## Acceptance Criteria
- Notifications send via EnGard (email) and AWS SNS (SMS) only.
- Channel selection respects stored consents.
- Appointment confirmations go out on email + SMS; telehealth link delivered.
- Sends are logged; invoice resends reuse the same Invoice ID.

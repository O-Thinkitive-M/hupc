# Telehealth — Patient Portal Feature Spec

## Purpose
- Let patients join telehealth visits from the portal via **Google Meet**.
- Deliver the video meeting link to the patient through SMS and/or email.

## Actors / Roles
| Actor | Capability |
|-------|-----------|
| Patient (Adult) | Join the telehealth visit from portal or link. |
| Guardian | Join on behalf of / with the active child. |
| System | Generates and delivers the Google Meet link. |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| TEL-1 | Patient | a telehealth visit option when booking | I can be seen remotely |
| TEL-2 | Patient | to receive a join link by SMS/email | I can join easily |
| TEL-3 | Patient | a "Join" button in the portal | I don't hunt for the link |
| TEL-4 | Patient | the link to open Google Meet | I use a familiar tool |

## Primary Workflow
1. Patient books an appointment with mode = **Telehealth** (see 03).
2. System generates a **Google Meet** link for the visit.
3. Link is **auto-sent to the patient via email (EnGard) and/or SMS (AWS SNS)** based on selection at booking time.
4. At visit time, patient taps **Join** in the portal (or the link in the SMS/email) → Google Meet opens.
5. Patient connects with the provider for the visit.

## Screens
No screen capture provided; per SRS/MoM the expected UI:
- Appointment detail showing Telehealth mode with a prominent **Join** button (enabled near visit time).
- Confirmation/reminder messages containing the Google Meet link.
- Pre-join readiness hint (camera/mic, browser) — accessibility-friendly.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| TelehealthSession | apptId, meetLink (Google Meet), deliveryChannels (email/SMS), joinWindow |
| LinkDelivery | apptId, channel, sentAt, status |

## Business Rules
- Telehealth video provider is **Google Meet** (NOT Zoom, NOT Twilio video).
- Meeting link is auto-sent via email and/or SMS per the patient's selection at booking.
- Join availability gated to a window around the scheduled time.
- Same self-access and consent rules as the rest of the portal apply.

## MoM Decisions / Deltas (authoritative)
- **Telehealth video vendor = Google Meet** (MoM-confirmed; overrides any SRS "Zoom" example wording).
- **Email = EnGard, SMS = AWS SNS** for link delivery (MoM-confirmed).
- Appointment Mode column includes Telehealth (MoM appointment-types delta).

## Dependencies
- Appointments (03) for mode + scheduling.
- Notifications (10) / EnGard / AWS SNS for link delivery.
- Google Meet integration.

## Open Items
- Join-window length before/after the scheduled time — TBD.
- Whether a backup link-resend is offered if delivery fails — TBD.

## Acceptance Criteria
- Booking a Telehealth appointment generates a Google Meet link.
- Link is delivered via email and/or SMS per booking selection.
- Patient can join from a portal **Join** button or the delivered link.
- No Zoom/Twilio video dependency appears anywhere.

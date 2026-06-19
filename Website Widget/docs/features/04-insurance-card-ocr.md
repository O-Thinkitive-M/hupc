# Insurance Card OCR — Website Widget Feature Spec

## Purpose
- Define **Step 3**: insurance card capture (front + back), OCR auto-fill of insurance fields, manual-entry fallback, and subscriber capture, for patients who selected Insurance as payment type.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Uploads card, reviews/confirms extracted fields |
| HUPC staff | Verify card images/fields downstream (audit) |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| OCR-1 | Prospect | to snap my insurance card | I don't type every field |
| OCR-2 | Prospect | to review/correct extracted values | data is accurate |
| OCR-3 | Prospect | manual entry if OCR fails | I can still proceed |
| OCR-4 | Prospect | to add subscriber info if not me | coverage maps correctly |

## Primary Workflow
1. Patient (Insurance payment type) is shown card-capture UI.
2. Uploads **front AND back** via phone camera or webcam.
3. If only one side uploaded → block: "Both the front and back of your insurance card are required."
4. Widget runs **OCR** to extract insurance fields (below).
5. Extracted fields auto-populate the insurance section; patient reviews and confirms; **manual override permitted**.
6. If OCR fails or patient prefers, **manual entry fallback** — same fields typed by hand.
7. If patient is not the policy subscriber, capture **subscriber details** separately.
8. Card images retained alongside extracted/entered fields for staff verification + audit.
9. Proceed to eligibility check (file 05).

## OCR-Extracted Fields
| Field | Notes |
|-------|-------|
| Insurance company / payer name | |
| Member ID | |
| Group number | |
| Plan name | |
| RX BIN | |
| RX PCN | |
| RX Group | |
| Effective dates | Stored as structured field |

## Screens
- No screen capture provided; per SRS/MoM, expected UI:
  - Front + back upload tiles with camera/webcam capture (mobile-first).
  - Error state for missing side.
  - Auto-filled, editable insurance field set with "review and confirm".
  - "Fill Details Manually" fallback link.
  - Conditional subscriber sub-form (if not subscriber).

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Insurance | Payer, member ID, group, plan, RX BIN/PCN/Group, effective dates |
| Card images | Front image, back image (retained, archived not deleted on replace) |
| Subscriber | Name, relationship, DOB (when patient ≠ subscriber) |

## Business Rules
- Both front and back images required to proceed.
- Extracted values are editable; manual override always allowed.
- Manual entry is a fallback, not the default (OCR is default capture path).
- Card images retained for staff verification and audit; prior ID archived (not deleted) on replacement.

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) **OCR auto-fill confirmed**: upload front + back → fields auto-fill via OCR → **editable after**. [decision locked]
- OCR auto-fill pairs with the in-flow **Check Eligibility** action and "view report" (file 05).
- No MoM change to required dual-side capture or retained-image audit behavior.

## Dependencies
- Registration Form (file 03) payment-type selection.
- OCR/image service.
- Eligibility Check (file 05) consumes captured insurance.

## Open Items
- OCR accuracy thresholds / confidence handling — implementation detail, not specified.

## Acceptance Criteria
- Uploading only one side blocks progression with the required message.
- OCR populates the listed fields and values remain editable.
- Manual fallback captures the same fields when OCR fails.
- Subscriber section appears when patient is not the subscriber.
- Card images persist for audit.

# Documents — Patient Portal Feature Spec

## Purpose
- Let patients upload documents (e.g., photo ID, insurance card) with a **document-type selector** that controls where the file lands in the patient chart.
- Let patients view documents shared with them by the practice.

## Actors / Roles
| Actor | Capability |
|-------|-----------|
| Patient (Adult) | Upload documents with a type; view shared documents. |
| Guardian | Same for the active child account. |
| Staff (Office Mgmt) | (Backend) In v1, may download + re-upload to the correct chart section. |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| DOC-1 | Patient | to upload a file and pick its document type | it lands in the right chart section |
| DOC-2 | Patient | to attach a photo ID via upload (not via message) | it's classified correctly |
| DOC-3 | Patient | to view documents the practice shared | I access my paperwork |
| DOC-4 | Staff | type-driven routing of uploads | I avoid manual re-filing |

## Primary Workflow
1. Patient opens Documents (or attaches within a request — see 04) → selects a file.
2. Patient selects a **document type** from a required selector (e.g., Photo ID, Insurance Card, Other).
3. File uploads; document type controls intended chart placement (e.g., Photo ID → Photo ID section).
4. **v1 fallback:** if type-based routing isn't wired, the file lands as a generic attachment and staff downloads + re-uploads to the correct chart section.
5. Patient can view documents shared by the practice in a list.

## Screens
No screen capture provided; per SRS/MoM the expected UI:
- Upload control with file picker + **required Document Type dropdown**.
- Uploaded-documents list with type, date, and status.
- Shared-documents list (practice → patient) with view/download.
- Same upload + type selector embedded in the request-attachment form (04).

## Data Entities
| Entity | Key fields |
|--------|-----------|
| PatientDocument | docId, patientId, fileRef, documentType, uploadedDate, source (upload/request) |
| SharedDocument | docId, patientId, sharedBy, sharedDate |
| ChartPlacement | docId, targetSection, routedAutomatically (bool) |

## Business Rules
- **Document type selector is required** when attaching/uploading; it controls chart placement.
- Photo IDs should classify to the Photo ID chart section (target behavior).
- v1: if no automatic type-routing, staff manually re-files; type is still captured.
- Patients see only their own (or active child's) documents (self-access-only).

## MoM Decisions / Deltas (authoritative)
- **Attachment upload added to portal requests** (MoM item 29).
- **Document type selector required (controls where it lands in patient chart)** (MoM item 29).
- **v1: if no type-routing, staff downloads + re-uploads manually** (MoM item 29).
- Photo ID accessible from two locations in the patient chart (SRS, retained).

## Dependencies
- Messaging/Requests (04) — shared upload + type-selector component.
- Patient chart Documents section (staff EMR) for placement.

## Open Items
- **Document-type taxonomy** must be defined (Photo ID, Insurance Card, etc.) — TBD.
- Whether automatic type-routing ships in v1 or stays manual — TBD.

## Acceptance Criteria
- Upload requires a document-type selection; type is stored with the file.
- Photo-ID uploads are intended for the Photo ID chart section (auto or via staff in v1).
- Patient can view documents shared by the practice.
- Self-access-only enforced; uploads audited.

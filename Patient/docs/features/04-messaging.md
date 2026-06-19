# Messaging & Requests — Patient Portal Feature Spec

## Purpose
- Let patients submit requests/messages to the practice from the portal.
- Route every request through **Office Management** for triage, regardless of the patient-selected department.
- Provide threaded visibility and status tracking of requests.

## Actors / Roles
| Actor | Capability |
|-------|-----------|
| Patient (Adult) | Submit requests, attach files, view responses. |
| Guardian | Same for the active child account. |
| Office Management | (Backend) Reads every request first, routes to correct department, logs reassignment. |
| On-call provider | Handles responses when the assigned provider is out of office. |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| MSG-1 | Patient | to submit a request with department/priority/description | the practice acts on my concern |
| MSG-2 | Patient | to attach a file (e.g., photo ID) | I provide supporting docs |
| MSG-3 | Patient | to see status of my request | I know it's being handled |
| MSG-4 | Patient | a threaded view of the conversation | I follow the back-and-forth |
| MSG-5 | Office Mgmt | every request to land with me first | I triage and route correctly |

## Primary Workflow
1. Patient opens Requests → creates a request: **Department / Priority / Description** (reason categories such as Billing / Medication / Other).
2. Patient may **attach a file** and select a **document type** (controls chart placement — see 06).
3. On submit, the request **default-routes to Office Management** regardless of the department the patient picked (patient selection is informational only).
4. Office Management reads the concern, routes to the correct department; a **log records who reassigned the request and the reason**.
5. If the assigned provider is out of office, the **on-call/coverage provider** responds; responses surface back to the patient in a **threaded view**.
6. Request status updates (e.g., Open → In Progress → Resolved); patient sees Case Number and timestamps.

## Screens
No screen capture provided; per SRS/MoM the expected UI:
- New-request form: Department dropdown, Priority, Description, attachment + document-type selector.
- Requests list columns: **Case Number / Department / Status / Priority / Submitted Date / Last Updated**.
- Threaded conversation view of a request with provider/office responses.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Request | caseNumber, patientId, selectedDepartment, priority, description, status, submittedDate, lastUpdated |
| RequestAttachment | requestId, fileRef, documentType |
| RoutingLog | requestId, fromDept (Office Mgmt), toDept, reassignedBy, reason, timestamp |
| RequestMessage | requestId, sender (patient/office/provider), body, timestamp |

## Business Rules
- **All requests default-route to Office Management**, regardless of patient's department selection.
- Patient-selected department/reason is informational; it does NOT control routing.
- Office Management reads each request and routes to the correct department; reassignment is logged (HIPAA audit consideration).
- Patient requests are NOT mapped directly to a provider; out-of-office is covered by the on-call provider.
- Reason categories include Billing / Medication / Other.

## MoM Decisions / Deltas (authoritative)
- **Patient-portal requests default-route to Office Management** (MoM item 28).
- **Office Management reads request and routes to correct department; log records reassignment** (MoM item 28).
- **Patient can still select reason/department (informational only); does not control routing** (MoM item 28).
- **Office Management always reads + routes requests (not direct-to-provider); on-call provider handles out-of-office** (MoM item 30).
- Default landing for request management = Office Management (MoM Executive Summary).

## Dependencies
- Documents (06) for attachment + document-type selector.
- Notifications (10) for new-response alerts.
- Office Management triage queue (staff EMR).

## Open Items
- Office-Management routing reassignment audit must meet **HIPAA audit-log** requirements — confirm scope.
- Final reason/department taxonomy shown to patients — TBD.

## Acceptance Criteria
- Any submitted request lands in Office Management regardless of selected department.
- Reassignment from Office Management to another department is logged with who + reason.
- Patient sees Case Number, status, and a threaded view of responses.
- Out-of-office responses are handled by the on-call provider, not blocked.

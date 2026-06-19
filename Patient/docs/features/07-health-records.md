# Health Records — Patient Portal Feature Spec

## Purpose
- Give patients a read/limited-write view of their clinical record from the portal.
- Add a **Past Psychiatric History** patient-view tab; let patients add psychiatric history entries.
- Make patient-entered clinical data available to auto-pull into the provider note.

## Actors / Roles
| Actor | Capability |
|-------|-----------|
| Patient (Adult) | View clinical data; add psychiatric history. |
| Guardian | Same for the active child account. |
| System | Auto-pulls patient-entered data into the note; enforces ROI on past notes. |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| HR-1 | Patient | a Psychiatric History tab | I track my mental-health history |
| HR-2 | Patient | to add a psychiatric history entry | my provider has accurate history |
| HR-3 | Patient | to see my allergies/medications | I keep them current |
| HR-4 | Patient | my entered data to reach my provider | I avoid repeating myself |
| HR-5 | Patient | NOT to see restricted past notes | (ROI rule respected) |

## Primary Workflow
1. Patient opens Health Records → sees clinical sections (allergies, medications, vitals, history).
2. Patient opens the **Psychiatric History** tab → columns: **Condition Name / Recorded Date / Current Medication / Note / Status**.
3. Patient **adds a psychiatric history entry** from the portal (condition, current medication, note, status).
4. Patient-entered clinical data (allergies, current medications, vitals, PHQ-9 answers) **auto-pulls into the provider note** — no manual pull.
5. Past-encounter notes are NOT shown to the patient (ROI rule).

## Screens
No screen capture provided; per SRS/MoM the expected UI:
- Health Records dashboard with sections for allergies, medications, vitals, history.
- **Psychiatric History tab** as a table: Condition Name, Recorded Date, Current Medication, Note, Status, plus an **Add** action.
- Add/edit psychiatric-history form.
- Past-encounter notes hidden/disabled with an ROI explanation.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| PsychiatricHistory | entryId, patientId, conditionName, recordedDate, currentMedication, note, status |
| Allergy | patientId, substance, reaction |
| Medication | patientId, name, dose, status |
| Vital | patientId, type, value, recordedDate |
| Phq9Response | patientId, answers, score |

## Business Rules
- Psychiatric History tab uses columns: Condition Name / Recorded Date / Current Medication / Note / Status.
- Patient may add psychiatric history from the portal.
- Patient-entered clinical data is the source for the note auto-pull (vitals, allergies, current medications, PHQ-9).
- Past-encounter notes blocked from patient view (ROI rule).
- Self-access-only; entries audited.

## MoM Decisions / Deltas (authoritative)
- **Past Psychiatric History added to patient portal (patient-view)** with **Condition Name / Recorded Date / Current Medication / Note / Status** columns (MoM item 26).
- **Patient can add psychiatric history from the portal** (MoM item 26).
- **Clinical data (vitals, allergies, current medications) auto-pulls from patient portal into the note** (MoM clinical item 1/2).
- **PHQ-9 patient portal answers auto-populate the note** (MoM clinical section).
- Family-psychiatric-history individual-person breakdown removed (provider-side; MoM) — not surfaced as separate portal entries.

## Dependencies
- Onboarding/intake (02) — initial clinical data capture.
- Clinical documentation module (staff EMR) — consumes auto-pulled data.

## Open Items
- Psychiatric-history **required fields** and validation — TBD; whether free-text or coded condition names — TBD.
- Exact ROI boundary for which past data is visible — confirm.

## Acceptance Criteria
- Psychiatric History tab renders the five specified columns and supports Add.
- Patient-added entries persist and are visible to the provider.
- Allergies/medications/vitals/PHQ-9 entered in portal are available for note auto-pull.
- Past-encounter notes are never displayed to the patient.

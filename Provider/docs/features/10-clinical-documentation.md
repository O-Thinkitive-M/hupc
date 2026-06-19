# Clinical Documentation — Provider Portal Feature Spec

## Purpose
- Provider-facing visit-note authoring for the Harmony EMR (Medent → Harmony migration).
- Supports Initial Evaluation Psychotherapy, Progress/Follow-up, Medication Management, and Group Therapy notes.
- Goal: minimize provider memory load — clinical data auto-pulls from the patient portal; AI Scribe populates the body; required-question prompts (headings) prevent gaps.

## Actors / Roles
| Role | Capability |
|------|------------|
| Provider (Prescribing / Non-Prescribing) | Author, edit, sign-and-lock, download note; record AI Scribe |
| Supervisor | Co-sign supervised provider notes (network-status driven) |
| Admin | Create macros, note templates, lethality/PHQ-9 config in Settings |
| Agent / Biller | Read note metadata (CPT/ICD, sign status) for charge capture; no clinical edit |
| Guardian | No note access (PHI) |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| CD-1 | Provider | clinical data (vitals, allergies, meds) to auto-pull into the note | I don't rely on memory to add it |
| CD-2 | Provider | PHQ-9 editable inline | I can complete it each initial session + every 90 days (depression) / annual |
| CD-3 | Provider | to type `@macro` in the note body | saved macro values populate inline |
| CD-4 | Provider | editable ROS, family drug/alcohol/tobacco, diagnosis | I can correct what patients answered untruthfully on the portal |
| CD-5 | Provider | start/end = actual face-to-face time | the real session duration is documented and downloaded |
| CD-6 | Provider | to download the note only after sign-and-lock | unsigned drafts aren't circulated |
| CD-7 | Provider | a group note + per-participant individual notes | group therapy is documented correctly |
| CD-8 | Provider | an optional Diagnostic Criteria selector | DSM criteria auto-populate for a chosen diagnosis |

## Primary Workflow
1. Open the encounter → note opens in **three-region layout**: Left panel (patient details + start/end timer; Vitals / Active Allergies / Current Medication / Past Encounter History, each hover-editable with drag-able pop-up tables; copy-to-note), Center (collapsible view-only appointment details + note-type dropdown + template dropdown + note body), Right panel (anchor scroll — click a section title to jump).
2. Clinical data auto-pulls from the patient portal (manual pull retained as fallback).
3. (Optional) Select a note template — sections are blank until chosen.
4. (Optional) Run **AI Scribe**: Record → Pause → Stop → capture transcript → auto-distribute to sections (per-section checkboxes) → Copy to Note merges into body.
5. Complete/edit sections: PHQ-9 (auto-filled from portal answers, editable), Lethality Assessment (6 questions, auto-configured from prior-session answers), ROS, Family Drug/Alcohol/Tobacco, Social History (every question asked individually — not collapsed), Mental Status Exam, Diagnosis (editable/addable, may change over treatment).
6. (Optional) **Diagnostic Criteria selector** → pick diagnosis → DSM criteria populate into note.
7. Type `@macro` anywhere to pull a saved macro value inline.
8. Enter **actual face-to-face Start/End time** (not the scheduled slot).
9. **Sign-and-Lock** (General Provider) → note becomes immutable; **Download Note** enabled only now; Start/End time renders into the signed PDF.

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-41.png` / `…15-20-45.png` — initial-evaluation note layout (left panel / center note / right anchor), AI Scribe controls.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-47.png` — PHQ-9 / lethality / ROS inline-editable sections.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-20-50.png` — group therapy note (group note + individual participant notes).
- (Verify exact filenames against the clinical-note screens captured 15-20-xx.)

## Data Entities
| Entity | Key fields |
|--------|-----------|
| VisitNote | id, encounterId, patientId, providerId, noteType, templateId, startTime, endTime (face-to-face), status (Draft/Signed-Locked), signedBy, signedAt, pdfUrl |
| PHQ9 | noteId, 9 item scores, totalScore, source (portal/edited), cadenceFlag |
| LethalityAssessment | noteId, 6 questions, auto-config source (prior session) |
| Macro | id, name, type, description, value, createdBy (Settings) |
| GroupNote | id, groupTopic, facilitator, interventions, numParticipants, timeSpent; children IndividualNote[] |
| DiagnosticCriteria | diagnosisCode, DSM criteria list |
| CentralizedChartData | patientId, history, allergies, medications, habits, questionnaires (single source of truth) |

## Business Rules
- Template selection optional; body blank until template chosen.
- Clinical data auto-pulls from portal; legacy New Patient Form removed.
- **Concurrent notes**: field values carry forward to subsequent notes; updates to history/allergies/meds/habits save to **centralized chart data**, not a per-note copy — a later provider pulls the latest centralized value, never stale copied data.
- PHQ-9 cadence: every initial session + every 90 days (depression) + annual otherwise.
- All auto-populated sections (ROS, family drug/alcohol/tobacco, diagnosis) must be clinician-editable in-session.
- Start/End = actual face-to-face session time (clients may sign off early/start late); must download into signed PDF.
- Download enabled **only after** sign-and-lock; performed by General Provider.
- Diagnosis on group note = macro/chart-populated (not free-text only).
- Time-spent (actual) required on every note (group + individual); scheduled time slots NOT required on the note.
- Diagnosis selection is AI-driven (AI Scribe) — no checkbox diagnosis selector.
- Sign-and-lock writes an immutable audit record; all reveals/edits audit-logged (PHI).

## MoM Decisions / Deltas (authoritative)
- **(Jun 8, Note Template Review w/ Emily)** Three-region layout confirmed; template dropdown optional; AI Scribe Record/Pause/Stop/Copy-to-Note accepted.
- Clinical data **auto-pulls** from portal (provider-configurable), manual pull as fallback; legacy New Patient Form removed.
- **Diamond-button macros NOT needed** → replaced by **@-mention macro system** (admin creates macro in Settings: type/title/description; provider types `@…`). Family-psychiatric-history per-person breakdown removed (Dr. Mohammed: block format for AI scribe).
- Section headings preserved as prompts; **Social History keeps every individual question** (no single block).
- **PHQ-9 editable inline** (not view-only); portal answers auto-populate; cadence as above.
- **Diagnostic Criteria selector** = OPTIONAL carry-over from Medent; populates DSM criteria for selected diagnosis. (Open: Lucas to source Medent reference.)
- **ROS + Family Drug/Alcohol/Tobacco + Diagnosis = editable inline.**
- **Lethality Assessment**: 6 questions, auto-configured from previous-session answers.
- **Download Note enabled only after Sign-and-Lock**; done by General Provider.
- **Start/End Time = ACTUAL face-to-face time** (not scheduled slot); downloads into signed PDF.
- **Group Therapy note**: group note + per-participant individual notes approved; diagnosis macro/chart-populated; duplicate "Interventions" section removed; actual time spent required.
- Group + individual notes auto-populate from previous encounters.
- **(Jun 16)** Concurrent notes + centralized chart data locked; Form Builder needed for templates; diagnosis AI-driven (no checkbox selector).

## Dependencies
- Patient portal intake (vitals/allergies/meds/PHQ-9 answers) — module 06.
- Settings → Templates / Macros / Visit Note template authoring (module 16).
- AI Scribe service (transcript capture + section distribution).
- Charge capture pulls CPT/ICD from signed notes (module 12).
- Telehealth: CPT-by-time billing may pull start/end from note (module 11/12).

## Open Items
- Lucas to supply Medent Diagnostic Criteria pop-up reference/screenshot (provider-only permission).
- Whether CPT-by-time billing pulls start/end from the note vs scheduled time (raised Jun 8) — confirm.
- PHQ-9 cadence enforcement: system reminder/auto-trigger mechanism — design TBD.
- Templates structure/rebuild deferred to dedicated future sessions (Jun 16).

## Acceptance Criteria
- [ ] Note opens in three-region layout; clinical data auto-pulls from portal.
- [ ] PHQ-9 auto-fills from portal answers and is editable inline; cadence reminders fire.
- [ ] `@macro` typed in body pulls the saved macro value inline.
- [ ] ROS, family drug/alcohol/tobacco, and diagnosis are editable in-session.
- [ ] Lethality Assessment shows 6 questions auto-configured from prior session.
- [ ] Start/End capture actual face-to-face time and render in the signed PDF.
- [ ] Download is disabled until sign-and-lock; enabled afterward for General Provider.
- [ ] Group note creates a group note plus per-participant individual notes; diagnosis chart-populated.
- [ ] Edits to centralized fields persist to the central chart, not just the note; next note pulls the latest value.
- [ ] All reveals/sign actions audit-logged.

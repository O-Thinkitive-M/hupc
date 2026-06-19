# Groups (Group Scheduling & Notes) — Provider Portal Feature Spec

## Purpose
Schedule group therapy sessions and document them with a combined **group note + per-participant individual notes**. Supports macro-driven diagnosis, lethality assessment, actual face-to-face time capture, and auto-population from previous encounters.

## Actors / Roles
| Role | Use |
|------|-----|
| Provider (Facilitator) | Run sessions, write group + individual notes, sign-and-lock |
| Supervising Provider | Review/co-sign where required |
| Agent | Schedule group sessions, manage participant roster |
| Biller | CPT-by-time billing from actual session time |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| GRP-1 | Agent | schedule a group session with participants | the roster is set |
| GRP-2 | Provider | a group note plus individual notes | I document both levels |
| GRP-3 | Provider | macro-select diagnosis from chart | diagnosis is accurate and editable |
| GRP-4 | Provider | actual start/end face-to-face time | billing reflects real time |
| GRP-5 | Provider | auto-populate from previous encounter | repeat documentation is faster |
| GRP-6 | Provider | download the note after sign-and-lock | I share a finalized PDF |

## Primary Workflow
1. Agent schedules a **group session**: select appointment type, provider/facilitator, location, date/time, and **participant list**.
2. Session runs; facilitator opens the **group therapy note**.
3. Fill **group note** sections: Group Topic / Interventions / Facilitator / Number of Participants / Diagnosis (macro) / Lethality Assessment.
4. Add **individual notes**: select each participant → complete their individual note (each can auto-populate from that participant's previous encounter).
5. Enter **actual start time + end time** (face-to-face) on every note.
6. **Sign-and-Lock** → Download Note option enabled (General Provider downloads); start/end times print into the signed PDF.

## Group Note Structure
| Level | Sections |
|-------|----------|
| Group note | Group Topic, Interventions (single — duplicate removed), Facilitator, Number of Participants, Diagnosis (macro / chart-populated), Lethality Assessment (6 questions, auto-configured), actual time spent |
| Individual note (per participant) | Participant-specific note + diagnosis (editable inline), actual time spent |

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-19-06.png` — Groups table / group scheduling (groups list view).

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Group session | SessionId, AppointmentType, Facilitator, Location, DateTime, Participants[] |
| Group note | GroupTopic, Interventions, Facilitator, NumParticipants, Diagnosis (macro), LethalityAnswers[], StartTime, EndTime, ActualTimeSpent, Signed |
| Individual note | ParticipantId, NoteBody, Diagnosis, ActualTimeSpent, Signed |

## Business Rules
- Diagnosis is **editable/addable inline** (made during first session, may change during treatment) and **macro/chart-populated** (not free-text only).
- **Actual time spent** required on **every** note (group + individual); scheduled time slots NOT used for these notes.
- **Start/End time = actual face-to-face session time** (clients may start late / sign off early) — must download into the signed PDF.
- **Lethality Assessment**: 6 questions, auto-configured from previous-session answers.
- **Download** enabled only after **Sign-and-Lock**; performed by the **General Provider** (not supervising).
- Notes auto-populate from previous encounters (group + individual).
- Duplicate "Interventions" section removed.

## MoM Decisions / Deltas (authoritative)
- **Group note + per-participant individual notes** layout approved (Emily, Jun-note session §11).
- **Diagnosis = macro/chart-populated**, editable inline (§7, §11).
- **Duplicate Interventions section removed** (§11).
- **Actual time spent required on every note; time slots NOT needed** (§10, §11).
- **Start/End = actual face-to-face time**, must print into signed PDF (§10).
- **Lethality Assessment**: 6 questions auto-configured from previous answers (§8).
- **Download only after Sign-and-Lock**, by General Provider (§9).
- **Auto-populate from previous encounters** for group + individual notes (§12).
- CPT-by-time billing pulls from start/end on note (open: vs scheduled time) — see Open Items.

## Dependencies
- Scheduling (file 06) — group session as an appointment type.
- Patients chart (file 05) — diagnosis macros + previous encounters source.
- Billing (file 10) — CPT-by-time billing from note start/end.

## Open Items
- Confirm CPT-by-time billing pulls from note start/end vs scheduled time.
- Full note-template field set deferred with Social History (end of project).

## Acceptance Criteria
- Group session captures a participant roster.
- Group note shows a single Interventions section + macro diagnosis + 6-question lethality assessment.
- Each participant has an individual note with editable diagnosis and actual time spent.
- Start/End reflect actual face-to-face time and print into the signed PDF.
- Download is available only after sign-and-lock.
- Notes auto-populate from previous encounters.

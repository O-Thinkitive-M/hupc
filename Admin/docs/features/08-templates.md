# Templates & Macros — Admin Portal Feature Spec

## Purpose
Manage clinical documentation templates — **Visit Note** templates and the admin-configurable **Macro** library — used by providers during charting (initial-evaluation psychotherapy, group therapy, etc.).

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin / Clinical Admin | Create/edit note templates and macro library |
| Provider | Consumes templates/macros while charting (Provider portal) |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| TM-1 | admin | create/edit visit-note templates per appointment/service type | providers chart consistently |
| TM-2 | admin | maintain a macro library of common phrases/prompts | charting is faster |
| TM-3 | admin | attach forms/templates to appointment types | the right note loads |
| TM-4 | admin | configure auto-populated sections (vitals/allergies pull-through) | data is consistent |

## Primary Workflow
1. Admin opens **Template** (screen 15-39-16).
2. Creates a note template (sections, free-text + optional macro prompts).
3. Configures auto-population (clinical data — vitals, allergies — pulls into the note).
4. Maintains the **Macro** library (common phrases / prompt text).
5. Links template + attached forms to appointment types (module 09).
6. Saves; providers see the template when charting that appointment type.

## Screens
- Template: `Screenshot from 2026-06-18 15-39-16.png`

## Data Entities
| Entity | Key Fields |
|--------|------------|
| NoteTemplate | templateId, name, appointmentTypeId, sections[], autoPopulateRules |
| Macro | macroId, label, text, category |
| TemplateForm | templateId, formId (attached intake/consent form) |

## Business Rules
- Notes are **free-text format with optional macro prompts**; macro library is **admin-configurable** (SRS §9.4 / §7240).
- Auto-populated sections (vitals, allergies) pull clinical data through (MoM template session).
- **Family-psychiatric-history individual-person diamond-button macros removed** (Dr. Mohammed) — do not implement that interaction (MoM template session §911).
- Group therapy note = group note + individual notes per participant (approved layout).
- CPT auto-population (e.g., 90853 for group therapy) is config-driven, not in the template editor.

## MoM Decisions / Deltas (authoritative)
- [x] Free-text notes with optional macros; macro library admin-configurable [MoM/SRS template review].
- [x] **Diamond-button macros for family-psychiatric-history removed** — declined by Emily/Dr. Mohammed [MoM §template].
- [x] Group therapy note layout (group + per-participant individual notes) approved [MoM §11].
- [ ] **Diagnostic Criteria Selector** (Medent DSM-criteria populator) is an **optional** carry-over, pending Lucas sourcing the Medent reference [MoM §5 template].
- Lightest clinical-config Admin module (**16 FE hours**) [FE estimation].

## Dependencies
- Module 09 (templates/forms attached to appointment types) · Module 07 (ICD-10/CPT referenced in notes) · Provider portal charting (consumer).

## Open Items
- Diagnostic Criteria Selector inclusion blocked on Medent reference (Lucas).
- Required-fields/validation per note template TBD.

## Acceptance Criteria
- [ ] Admin can create/edit visit-note templates with sections + macro prompts.
- [ ] Macro library is admin-configurable.
- [ ] Templates/forms link to appointment types.
- [ ] Removed family-psych diamond-button interaction is NOT present.
- [ ] Group therapy template supports group + per-participant notes.

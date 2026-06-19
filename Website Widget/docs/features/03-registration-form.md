# Registration Form — Website Widget Feature Spec

## Purpose
- Define **Step 2**: the single comprehensive new-patient registration form capturing all information required for account creation in one screen.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Completes the form |
| Guardian / Legal Representative | Completes Guardian section when patient is a minor |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| RF-1 | Prospect | one form for all my details | I register without multiple screens |
| RF-2 | Prospect | service + mode options | the right providers/slots show later |
| RF-3 | Guardian | a guardian section for my child | I can book for a minor |
| RF-4 | Prospect | clear validation | I fix errors before submitting |

## Primary Workflow
1. Patient lands on the registration form after identification (file 02).
2. Completes form sections (below).
3. DOB drives Minor/Adult: minor → Guardian section appears.
4. Payment type radio (Insurance / Self-Pay) controls insurance path:
   - Insurance → insurance card capture + OCR (file 04).
   - Self-Pay → insurance section skipped.
5. Mandatory marketing attribution ("How did you hear about us?") captured.
6. Validation runs; on pass, form advances to eligibility (insurance) or OTP (self-pay).

## Form Sections
| Section | Fields |
|---------|--------|
| Personal Information | Legal first/middle/last name; **Prefix (optional)**; preferred name; DOB; gender; address (street, apt, city, state, ZIP) |
| Contact Details | Email; phone type (Mobile/Home/Work); **mobile (required for OTP)**; additional phone (optional); preferred contact method |
| Appointment Details (Service & Mode) | Service needed (Med Mgmt / Psychotherapy / Group Therapy / Neurocognitive Testing / TMS); appointment mode (Telehealth / In-Person, filtered by service) |
| Insurance Information | Payment type radio (Insurance / Self-Pay); insurance details if Insurance (file 04) |
| Financial Responsibility | Self / **Guardian-Legal Representative** / **Other**; payment method (card or bank) |
| Additional Information | Emergency contact; communication prefs; marketing consent; **How did you hear about us? (mandatory)**; best time to call; description/queries |
| Guardian (minor only) | Relationship, first/last name, contact details, email(s) |

## Screens
- No screen capture provided; per SRS/MoM, expected UI:
  - Single scrollable, sectioned form (mobile-first), collapsible sections.
  - Conditional Guardian section appears on minor DOB.
  - Inline Contact Support (Connect with Live Agent / Request Callback) within Additional Info.
  - Inline validation messages.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Patient demographics | Name, prefix, preferred name, DOB, gender, address |
| Contact | Email, phones, preferred method |
| Appointment intent | Service, mode |
| Financial responsibility | Party type (Self / Guardian-Legal Rep / Other), payment method |
| Marketing attribution | Source (mandatory), consent |

## Business Rules
- Mobile number is required (used for OTP).
- Form cannot submit without insurance details OR a Self-Pay selection.
- Appointment mode options are filtered dynamically by service (e.g., NCT/TMS force In-Person).
- "How did you hear about us?" is mandatory; "Other" allows free text.

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) Form sections locked: **Personal Information / Appointment Details / Address Information / Insurance Information / Financial Responsibility / Additional Information**.
- **"Suffix" renamed to "Prefix"** and made **optional** (Dr. Mohammed).
- Financial Responsibility: "Guardian" → **"Guardian / Legal Representative"**; "Organization" → **"Other"**.
  - **Self** → capture payment method (card or bank).
  - **Guardian / Legal Representative** → first/last name, relationship, contact, email, address + payment method.
  - **Other** → org name, org type, contact person, phone, email, address + payment method.
- **DOB-driven Minor/Adult auto-detection** live (minor → Guardian section).
- Additional Info includes inline **Contact Support** (Connect with Live Agent / Request Callback).

## Dependencies
- Patient Identification (file 02), Insurance OCR (file 04), Eligibility (file 05), OTP (file 06).
- EHR Patient Registration (mirrors this structure for the in-EHR Add Patient form).

## Open Items
- Guardian using one email across two child accounts vs unique-constraint — pending Dr. Mohammed.

## Acceptance Criteria
- Form renders all locked sections; Prefix is optional.
- Minor DOB reveals Guardian section.
- Self-Pay hides the insurance section; Insurance requires card/manual details.
- Submission blocked without insurance details or Self-Pay, and without marketing source.

# Patient Registration & Onboarding — Provider Portal Feature Spec

## Purpose
Create new patient records from any intake channel via a multi-section Add-Patient form (mirroring the website widget). Captures demographics, patient-type classification, guardian / financial responsibility, patient ID, insurance (with OCR + eligibility), and preferences.

## Actors / Roles
| Role | Use |
|------|-----|
| Agent (Office) | Register patients (phone/walk-in), verify ID, capture insurance |
| Admin | Configure registration sources, ID types |
| Biller | Verify financial responsibility / self-pay |
| Patient | Self-registers via website widget (feeds same data model) |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| REG-1 | Agent | a sectioned Add-Patient form | I capture all data in one flow |
| REG-2 | System | DOB-driven minor/adult detection | guardian section appears when needed |
| REG-3 | Agent | OCR insurance-card auto-fill + eligibility check | data entry is fast and accurate |
| REG-4 | Agent | capture financial responsibility (Self/Guardian/Other) | billing routing is correct |
| REG-5 | Agent | flag a non-consenting adult + upload POA | legal guardianship is documented |
| REG-6 | Agent | record registration source | attribution is preserved |

## Registration Sources
Website, phone, email referral, marketing, referral (form/link), fax referral, walk-in. Source recorded for marketing attribution and conversion tracking.

## Primary Workflow
1. Agent opens **Add Patient** (or website widget submits) → form sections render.
2. **Patient Details** — name, **Prefix** (optional), DOB (drives minor/adult), contact.
3. If **minor** → **Guardian Details** section auto-appears (relationship, name, contact, email).
4. **Patient ID** — select ID type, upload front + back, verify.
5. **Financial Responsibility** — choose Self / Guardian-Legal Representative / Other → capture payment method + contact/address.
6. **Address**, **Provider Assignment**, **Contact Details**.
7. **Insurance** — upload card front/back → OCR auto-fill (editable) → **Check Eligibility** → view report; or **Self-Pay**.
8. **Preferences & Concerns**, **Emergency Contact** (removable).
9. **Non-Consenting Adult** checkbox → Legal Guardian details + **POA document upload**.
10. Submit → patient created (linked to lead source).

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-19-06.png` — Add-Patient form (sections / patient details).
- `HUPC Screens Images/Screenshot from 2026-06-18 15-19-16.png` — Guardian / Financial Responsibility section.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-19-32.png` — Insurance section (OCR upload + eligibility) / Patient ID.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Patient | MRN, Prefix (opt), First/Last, DOB, PatientType, Source, Phone, Email, Address |
| Guardian | Relationship, Name, Contact, Email, Address |
| FinancialResponsibility | Type (Self/Guardian-Legal Rep/Other), PaymentMethod (Card/Bank), Org fields (if Other) |
| PatientID | IDType, FrontImage, BackImage, Verified |
| Insurance | Payer, MemberID, Group, Primary/Secondary, EligibilityStatus |
| POA | Document, NonConsentingAdult flag |

## Patient Type Classification
4 patient types replace the old Medent 3-type model (DOB-driven minor/adult + consent status feed classification). Status tabs in chart: Active / Inactive / Archived / Deceased (see file 05).

## Business Rules
- **DOB** auto-classifies Minor vs Adult; minor → guardian section mandatory.
- **Prefix** field (renamed from "suffix") is **optional**.
- Self-pay or primary insurance required (new patients must provide one).
- Non-consenting adult requires Legal Guardian + POA upload.
- Insurance OCR output always editable before save.
- PHI masked per role; reveal audit-logged.

## MoM Decisions / Deltas (authoritative)
- **"Suffix" → "Prefix"** rename + made **optional** (Dr. Mohammed, Jun 15 §3).
- **Financial Responsibility renames**: "Guardian" → **"Guardian / Legal Representative"**; "Organization" → **"Other"** (Dr. Mohammed, Jun 15 §6).
- **4 patient types** replace Medent's 3-type Staff/Provider/Patient model (Jun 17 settings).
- **DOB-driven minor/adult auto-detection** with conditional guardian section (Jun 15 §4).
- **Insurance OCR auto-fill + Check Eligibility + view report** inline (Jun 15 §5).
- **Add Patient form mirrors website-widget structure**; **Non-Consenting Adult + POA upload**, Patient ID front/back + verify, Emergency Contact (Jun 15 §17).
- New patients created from fax referral **linked to Lead Management** as a source.

## Dependencies
- Leads (file 03) — conversion entry point + source attribution.
- Patients chart (file 05) — registered patient opens into chart.
- Scheduling (file 06) — only **Active** patients are schedulable.
- Settings (file 13) — ID types, sources, provider assignment master.

## Open Items
- Pharmacy directory data source (city + pharmacy) — NCPDP vs manual.
- Provider-filter source data (age group, language, diagnosis) — provider master vs preference form.

## Acceptance Criteria
- Add-Patient form renders all sections; Prefix is optional.
- Minor DOB triggers a mandatory guardian section.
- Insurance OCR populates editable fields; eligibility check returns a viewable report.
- Financial Responsibility offers Self / Guardian-Legal Representative / Other with correct sub-fields.
- Non-consenting adult requires POA upload before save.
- Registered patient retains source attribution and appears in the Active patient list.

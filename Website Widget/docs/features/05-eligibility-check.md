# Eligibility Check — Website Widget Feature Spec

## Purpose
- Define **Step 4**: automated real-time eligibility check against the entered insurance, including pass and fail handling.

## Actors / Roles
| Actor | Role |
|-------|------|
| New Patient (prospect) | Triggers check by submitting registration; chooses fail-path option |
| HUPC Scheduling team | Resolves eligibility issues via callback / live agent |
| Eligibility vendor | Returns active/in-network/copay/deductible/coverage |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| EL-1 | Prospect | my insurance verified automatically | I see real cost before booking |
| EL-2 | Prospect | to see copay/deductible | I know what I'll pay |
| EL-3 | Prospect | options if verification fails | I can still get help |
| EL-4 | Self-Pay prospect | to skip eligibility | I proceed faster |

## Primary Workflow
1. After registration submit, system runs real-time eligibility check on entered insurance.
2. **Self-Pay patients skip this step** → straight to Step 5 (OTP, file 06).
3. Check determines: plan active; HUPC in-network; copay/deductible for service; plan-specific service coverage.
4. **Pass** (active + in-network) → proceed to OTP (Step 5).
5. **Fail** (inactive, out-of-network, vendor outage, unsupported payer, other) → present two options:
   - **Option A — Request Callback**: capture data so far; staff follows up; "We will call you back at [phone] within [SLA window]."
   - **Option B — Connect to Live Agent (phone bridge)**: agent calls patient; gated by HUPC operating hours.

## Eligibility Check Logic
| Check | Determines |
|-------|-----------|
| Plan active | Coverage currently in force |
| In-network | HUPC is in-network with plan |
| Copay / deductible | Patient cost for selected service |
| Service coverage | Plan covers Med Mgmt / Psychotherapy / etc. |

## Fail-Path Options
| Option | Behavior | Gating |
|--------|----------|--------|
| A — Request Callback | Saves partial registration; staff resolves by phone; SLA message shown | Always available |
| B — Live Agent (phone bridge) | Agent calls patient to resolve + complete booking | HUPC operating hours; outside hours hidden/disabled with "Live agents are available [hours]. Please request a callback to continue." |

## Screens
- No screen capture provided; per SRS/MoM, expected UI:
  - Real-time "Checking eligibility…" state.
  - **Check Eligibility** button + **view report** (per MoM demo).
  - Pass: confirmation + copay/deductible summary.
  - Fail: two-option panel (Callback / Live Agent) with availability-aware Live Agent.

## Data Entities
| Entity | Key fields |
|--------|-----------|
| Eligibility result | Active?, in-network?, copay, deductible, covered services, status/reason |
| Callback request | Partial registration payload, phone, SLA window |
| Eligibility report | Vendor report (viewable) |

## Business Rules
- Self-Pay bypasses eligibility entirely.
- Any failure reason routes to the two-option fail path (no silent dead-end).
- Eligibility-derived copay/deductible feeds provider cards (file 07) and payment (file 12).
- Live Agent availability is gated by operating hours.

## MoM Decisions / Deltas (authoritative)
- (Master Finalization, Jun 15 2026) **OCR auto-fill + eligibility check + view report** locked; a "Check Eligibility" button shows a viewable report inline.
- Additional Info **Contact Support** (Connect with Live Agent / Request Callback) is surfaced inline — aligns with the eligibility fail-path options.
- No MoM change to Self-Pay skip behavior.

## Dependencies
- Insurance OCR (file 04) for insurance data.
- Eligibility verification vendor.
- Provider Selection (file 07) and Payment (file 12) consume copay/deductible.
- Lead Management (callback requests become leads/touchpoints).

## Open Items
- SLA window value and operating-hours definitions — config, not fixed in spec.

## Acceptance Criteria
- Self-Pay patients never trigger eligibility.
- Passing eligibility shows copay/deductible and proceeds to OTP.
- Any failure presents Callback + Live Agent options.
- Live Agent is hidden/disabled outside operating hours with the messaged fallback.

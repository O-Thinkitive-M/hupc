# Harmony EMR — Project Overview

## Project Info
| Field | Value |
|-------|-------|
| Client | Harmony United Psychiatric Care (HUPC) |
| Type | Custom EHR/EMR Platform — full source code ownership |
| Scale | ~80 providers, 28 locations, Florida-based, expanding |
| Figma File | `nJXFzS6MzrFXQmcZcmMT4i` |
| Figma MCP | `~/.gemini/antigravity/mcp_config.json` → `Framelink Figma MCP` |
| Website | hupcfl.com |
| Current EMR | Medent (being replaced) |
| Started | Discovery: March 2026 |

## Tech Stack
| Layer | Technology |
|-------|-----------|
| Frontend | React 19 + Vite 8 |
| Components | MUI (Material UI) + brand tokens |
| Icons | Lucide React (centralized in `src/config/icons.js`) |
| Routing | React Router v6 |
| Server State | TanStack Query v5 |
| Client State | Zustand |
| Forms | React Hook Form + Zod validation |
| HTTP | Axios (`src/services/`) |
| Virtual Scroll | @tanstack/react-virtual |
| Dates | dayjs (timezone support) |
| Payments | Stripe.js + React Stripe |
| Mobile App | iOS + Android (planned) |

## Infrastructure
| Service | Technology |
|---------|-----------|
| Cloud | AWS (client-owned account) |
| Database | RDS (PostgreSQL) |
| Storage | S3 buckets |
| Compute | EC2 instances |
| CDN | CloudFront |
| Environments | Dev, QA, UAT (staging), Production (VPC-isolated) |
| CI/CD | GitHub → EC2 runner (no GitHub minutes cost) |
| Compliance | HIPAA, FHIR interoperability support |

## Key Actors / Roles
| Role | System Access |
|------|---------------|
| New Patient | Public portal, registration, booking, intake forms |
| Established Patient | Auth portal, bill pay, appointments, records |
| Customer Care Agent | Scheduling, leads/callbacks, fax mgmt, account creation |
| Provider (MD/NP/Therapist) | Patient chart, visit notes, PHQ-9, consent review |
| Billing Department | Payment portal, insurance verification, AR, claims |
| Admin / Office Manager | Admin portal, provider/location/form/user config |
| Referral Partner | Online referral form, fax, email |
| HR Department | Payroll, attendance, CPT code scoring, performance mgmt |
| Marketing | Reports, lead attribution, campaign analytics |
| Quality / Audit | Compliance reports, MIPS, chart completeness |

## 7-Milestone Development Plan
| Milestone | Weeks | Key Deliverables |
|-----------|-------|-----------------|
| Discovery (10%) | 2 | Workflows, SRS, high-fidelity Figma |
| Milestone 1 | 4 | Tenant portal, provider group mgmt, RBAC, user mgmt, master data; Patient onboarding, scheduling; Email (SendGrid), SMS (Twilio) |
| Milestone 2 | 4 | Dashboard, intake forms, form builder, appointment types, fee schedule, encounter; Clearing house integration |
| Milestone 3 | 4 | Telehealth, patient charting, fax mgmt, tasks, website widget settings; Fax + Telehealth (Zoom) |
| Milestone 4 | 4 | Messages, lab orders/results, e-prescription, referral mgmt, email/SMS templates, visit note templates; Lab + e-Rx integrations |
| Milestone 5 | 4 | Superbill, electronic claims, AR, patient payments; Stripe + Clearing house |
| Milestone 6 | 4 | EOB/ERA, AR; Patient portal (scheduler, messaging, telehealth, documents, health records, billing) |
| Milestone 7 | 4 | AI note scribe, dashboard analytics, MIPS reporting |

UAT: 3 weeks per milestone (extended 1 month for final milestone)

## Discovery Sessions Completed
| # | Module | Date | Status |
|---|--------|------|--------|
| 1 | Patient Onboarding & Registration | Mar 2 | Completed |
| 2 | Website Widget | Mar 4 | Completed |
| 3 | Referral Management | Mar 5 | Completed |
| 4 | Pre-Check-in / Intake Forms | Mar 9 | Completed |
| 5 | Appointments | Mar 10 | Completed |
| 6 | Scheduling | Mar 11 | Completed |
| 7 | Service Type Configuration | Mar 12 | In Progress |
| 8 | Clinical Documentation | Mar 16 | Completed |
| 9 | Group Scheduling/Notes | Mar 17 | Completed |
| 10 | Patient Charting | Mar 18 | Completed |
| 11 | Telehealth | Mar 18 | Completed |
| 15 | Fax Management | Mar 23 | Completed |
| 17 | Insurance Verification | Mar 24 | Completed |
| 19 | Prior Auth / Claims | Mar 25 | Completed |
| 20 | Provider Setup | Mar 24–26 | Completed |
| 22 | ERA / Payments / Superbill | Mar 30 | Completed |
| 23 | Denial Management | Mar 31 | Completed |
| 25 | Denial Management QA | Apr 8 | Completed |
| 26 | Marketing Dashboard, RBAC & User Mgmt, Reporting | Apr 9 | Completed |
| Pending | Admin Portal Walkthrough, BI Tool Evaluation, Website Redesign Impact, AI | Apr 2026 | Scheduled |

## Vendor Team
| Name | Role |
|------|------|
| Dhananjay Kolte | Project Manager |
| Saurabh Maslekar | Development Lead |
| Girish Bidwai | Quality Lead |
| Shubham Patil | Design Lead |
| Anita Kankate | Senior BA |
| Dnyneshwari Vaishnav | BA / UI Design |
| Akshay Patil | QA Engineer |
| Sapana More | Test Automation |
| Maithili Inamdar | UI Engineer |
| Tanmaya | Business Analyst |

## HUPC Stakeholders
| Name | Role |
|------|------|
| Shirley Solace | PM / Legal |
| Dr. Mohammad | Clinical Director |
| Melissa | Legal / Compliance |
| Jim Peterson | Office Management |
| Edward Stark | Office Management |
| Adrian Murray | Billing |
| Kimmey Loughe | Clinical / Quality |
| Brett Lewis | Marketing Director |
| Chantel Vickers | Referral Processing |
| Jake Baker | IT Department |

## Current Systems Being Replaced
- **Medent** — existing EMR
- **JotForm** — all intake/referral/booking forms
- **QuickBooks** — payment gateway (not accounting)
- **WordPress** — static provider directory, iFrame booking
- **RingCentral** — phone (future phase integration)

## Open Issues
10 open decisions pending — see `healthcare-domain-reference.md`.

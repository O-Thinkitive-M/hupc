# Dashboard (Chart Central) — Provider Portal Feature Spec

## Purpose
Landing hub ("Chart Central") giving providers and office staff an at-a-glance operational view: KPI stat cards, upcoming appointments, pending and urgent work, lab/imaging results, and a messages panel. Drives daily triage and surfaces what needs action first.

## Actors / Roles
| Role | Use |
|------|-----|
| Provider | Own schedule, urgent patients, results to review, messages, staff notes |
| Agent (Office) | Office messages, unread faxes, pending items, request triage |
| Biller | Pending billing items surfaced via stat cards (limited) |
| Admin | Full visibility |

## Key User Stories
| ID | As a… | I want… | So that… |
|----|-------|---------|----------|
| DASH-1 | Provider | KPI cards for urgent patients / appointments / pending items | I prioritize my day |
| DASH-2 | Provider | upcoming appointments with quick actions | I start encounters fast |
| DASH-3 | Provider | lab & imaging results needing review | nothing clinical is missed |
| DASH-4 | Office staff | unread faxes & office messages count | inbound documents are processed |
| DASH-5 | User | a messages panel with department segregation | I see only my department's messages |
| DASH-6 | User | urgent tasks / staff notes | escalations are handled |

## Primary Workflow
1. User logs in → lands on **Chart Central** dashboard.
2. **KPI stat cards** render counts: Urgent Patients, Appointments (today), Pending Items, Office Messages, Unread Faxes, Staff Notes.
3. Click a stat card → filtered drill-down (e.g., Unread Faxes → fax inbox under Chart Central / Medical Records).
4. **Upcoming Appointments** list → select → open encounter / start appointment.
5. **Pending Items** + **Urgent Tasks** lists → act or reassign.
6. **Lab & Imaging Results** panel → view report / requisition → mark reviewed.
7. **Messages panel** → open Message Central thread (department-scoped).

## Screens
- `HUPC Screens Images/Screenshot from 2026-06-18 15-16-07.png` — Chart Central dashboard: KPI stat cards row + upcoming appointments.
- `HUPC Screens Images/Screenshot from 2026-06-18 15-16-14.png` — Dashboard lower panels: pending items, urgent tasks, lab/imaging results, messages panel.

## Data Entities
| Card / panel | Key fields |
|--------------|-----------|
| Stat card | Label, Count, Drill-down target |
| Upcoming appointment | Patient, Provider, Date+Time, Type, Location, Status, Action |
| Pending item | Type, Subject, Due Date, Assigned To, Priority |
| Urgent task | Subject, Department, Priority, Due Date, Assigned To |
| Result | Patient, Test/Imaging, Date, Status (New/Reviewed), View Report/Requisition |
| Message | Department, Sender, Snippet, Received, Unread flag |

## Business Rules
- Counts respect **RBAC**: a user sees cards/items for their role and department only.
- **Department-level segregation**: messages and faxes are scoped so staff see only their department's items.
- PHI on cards/lists masked per role; reveal-by-click + audit.
- Urgent items sort to top; today's faxes appear bold, older faxes grayed out.
- Stat-card counts are live and reflect current worklist state.

## MoM Decisions / Deltas (authoritative)
- **Department-level message segregation** on the messages panel (Message Central feeds dashboard; staff see only their department).
- **Unread faxes** surface here; fax inbox sits under **Chart Central / Medical Records** nav (SRS confirmed, carries forward).
- **PHI masking** applies to dashboard lists (Jun 4 §11 / Jun 15 §18).
- Color-coded markers (e.g., rescheduled entries) so staff/Legal can spot flagged rows (carry-forward from deposition session).

## Dependencies
- Scheduling (file 06) feeds Upcoming Appointments + utilization.
- Communication / Message Central (file 09) feeds messages + faxes.
- Orders/Lab/Imaging (file 16) feed results panel.
- Patients chart (file 05) is the drill-down target for patient cards.

## Open Items
- Exact stat-card thresholds for "urgent" classification — to confirm.
- Whether biller-specific KPI cards appear on the same dashboard or a billing variant.

## Acceptance Criteria
- Dashboard renders all six KPI stat cards with live, role-scoped counts.
- Clicking a card opens the correct filtered drill-down.
- Upcoming appointments list supports start-encounter action.
- Lab/imaging results panel distinguishes New vs Reviewed and opens report + requisition.
- Messages panel shows only the user's department messages; faxes follow department segregation.
- All PHI on the dashboard is masked by default and reveal is audit-logged.

# Admin Dashboard — Admin Portal Feature Spec

## Purpose
Give administrators an at-a-glance operational view of the clinic via KPI cards, charts, and alert summaries, with drill-down into the underlying modules. Heaviest Admin module by FE effort (**204 FE hours**).

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin / Office Manager | Monitors clinic KPIs and alerts |
| Billing Admin | Monitors AR / claims KPIs |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| DA-1 | admin | see KPI summary cards (appointments, no-shows, slot occupancy, pending balances) | I track clinic health |
| DA-2 | admin | see overdue/alert KPIs | nothing is missed |
| DA-3 | admin | filter the dashboard by location/date range/provider | I focus my view |
| DA-4 | admin | click a KPI to drill into the source list | I act on the number |
| DA-5 | admin | view AR / claims / payment KPIs | I monitor revenue cycle |

## Primary Workflow
1. Admin lands on Dashboard after login.
2. KPI cards render (volume, no-show rate, slot occupancy, pending balance count, overdue tasks).
3. Charts render trends (appointments over time, revenue, denial rate).
4. Admin applies filters (location, date range, provider/department).
5. Admin clicks a card/segment → navigates to filtered list in the owning module.
6. Alert/overdue summary links to module 12 (Alerts).

## Screens
- No dedicated screenshot supplied; composes data from config modules. Reference appointment/billing sub-screens for drill targets (`15-38-49`, `15-39-12`).

## Data Entities
| Entity | Key Fields |
|--------|------------|
| KpiCard | metricId, label, value, delta, threshold, drillTarget |
| DashboardFilter | locationId, dateRange, providerId, departmentId |
| SlotOccupancy | location, occupied, available, denominator (excludes blocked/holiday) |

## Business Rules
- **Slot occupancy denominator excludes blocked time and holidays** (SRS §scheduling).
- Pending-balance KPI uses the configured blocking threshold ($100; $200+ pending billing confirmation — SRS §16.2).
- Overdue KPI cards surface manager-level dashboard alerts (SRS §3.6/3.7).
- Drill-downs respect RBAC and PHI masking.

## MoM Decisions / Deltas (authoritative)
- No dashboard-specific MoM V2 deltas; KPI scope derives from SRS §3.7 (KPI Summary Cards) and §scheduling occupancy metrics.
- Dashboard must surface the new **Alerts** stream from MoM §15 (trigger event/timing/priority/assigned-to/status).

## Dependencies
- Module 09 (occupancy, holidays) · Module 10 (AR/billing KPIs) · Module 12 (Alerts) · Module 04 (provider/department filters) · RBAC + PHI masking.

## Open Items
- Final KPI list and thresholds (no-show %, occupancy target, balance threshold) pending.
- Whether dashboards are role-specific (billing vs OM views).

## Acceptance Criteria
- [ ] KPI cards render with value + delta + threshold state.
- [ ] Filters (location/date/provider/department) recompute all cards and charts.
- [ ] Clicking a KPI drills into a filtered list honoring RBAC/PHI.
- [ ] Occupancy excludes blocked/holiday slots from the denominator.

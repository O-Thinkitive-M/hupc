# 00 — Master Plan: Harmony EMR Frontend Foundation

> **Scope:** Foundation architecture for the HUPC / Harmony EMR React frontend (client: Harmony United Psychiatric Care).
> **Portal note:** This *Setup plan* is **identical** across the **Admin / Provider / Patient / Website-Widget** portals. Only **config values** differ (nav modules, route maps, status registries, permissions, brand toggles). The primitives, theme system, and scaffold are shared verbatim.

---

## Executive Summary
- Large psychiatric-care EHR: screens are **complex but highly repetitive** (tables, filters, KPI strips, tabbed detail pages, schema-driven forms).
- **Core thesis:** build a *small set of config-driven primitives*, then **assemble every screen declaratively** from `config + primitives`. No bespoke per-screen layout code.
- Outcome: change-safe, consistent, fully-responsive, i18n-clean, cross-browser UI delivered fast.

---

## Config-Driven-Primitive Thesis
- Each screen = **declarative config object** (columns, filters, tabs, form schema, status map) + **generic primitive** that renders it.
- One primitive serves dozens of screens → fix/improve once, propagates everywhere.
- Config lives in `src/config/` and per-feature `features/<feature>/config`; primitives in `src/components/`.

---

## Screen → Primitive Map

| Screen pattern | Primitive(s) |
|---|---|
| List/index (Leads, Patients, Billing) | `DataTable` + `FilterBar` + `StatusChip` + `ActionDock` |
| KPI / dashboard strip | `KpiCard` grid |
| Detail / record page | `NestedTabPage` + `MasterDetail`/`SplitPane` |
| Create/Edit record | `SchemaForm` + `FormSection` + `AsyncAutocomplete` |
| Scheduling / calendar | `Calendar` + `TimeSlotGrid` |
| Documents (Legal, charts) | `DocumentViewer` + `DocumentLibrary` |
| Settings (nested) | `SettingsHub` (NestedTabPage variant) |
| Modal/side flows | `Drawer` / `Modal` |
| Async/data states | `QueryState` + `ErrorBoundary` |
| Long text cells | `TruncatedText` |

---

## Topic Files (01–20) — Composition Map

| # | File | One-line purpose |
|---|---|---|
| 01 | tech-stack-and-decisions | Pinned deps + conflict resolution |
| 02 | project-scaffold | Vite init, tsconfig, folder tree, env |
| 03 | theme-system | `src/theme/` tokens + `createTheme` |
| 04 | i18n-text-registry | No-literal-string registry + ESLint |
| 05 | routing-deep-linking | `createBrowserRouter`, URL state |
| 06 | api-sdk-orval | Orval SDK + axios mutator + React Query |
| 07 | state-management | Zustand stores + query cache split |
| 08 | datatable-primitive | Config-driven virtualized table |
| 09 | statuschip-registry | Status registry + chip |
| 10 | filterbar-primitive | Declarative filters → URL |
| 11 | kpicard-primitive | KPI/metric cards |
| 12 | schemaform-forms | RHF + zod SchemaForm/FormSection |
| 13 | asyncautocomplete | Debounced async select |
| 14 | nestedtabpage-settingshub | Tabbed detail + settings |
| 15 | splitpane-masterdetail | Master/detail layouts |
| 16 | drawer-modal | Overlay primitives |
| 17 | document-viewer-library | Document view/manage |
| 18 | calendar-timeslotgrid | Scheduling primitives |
| 19 | feedback-error-query-state | ErrorBoundary, QueryState, toasts |
| 20 | layout-shell-nav | App shell + 11-module nav |

---

## Reconciled Key Decisions

| Decision | Choice | Note |
|---|---|---|
| Primary color | `#2D5F8D` (Harmony Blue) | Authoritative per brand guidelines |
| Typography | System UI / SF stack | `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif`; min body 16px |
| TypeScript | Strict + `noUncheckedIndexedAccess` | No `any` escape hatches |
| API layer | Orval-generated typed SDK | Replaces hand-written services |
| Responsiveness | Mobile 320px+ → Large 1440+ | Mac/Retina/any device |
| Cross-browser | Chrome/Edge/Safari/Firefox last-2 + iOS/Android | |
| i18n | No hardcoded UI text | ESLint `no-literal-string`, `src/i18n/` |
| Components | Strong, fully-configurable, change-safe | Config-driven primitives |
| Deep-linking | URL is source of truth | Filters/tabs/pagination in URL |
| Error handling | `ErrorBoundary` + `QueryState` | Consistent async/error UX |
| Accessibility | WCAG 2.1 AA | |

---

## Build Phase Sequence

| Phase | Output |
|---|---|
| 1. Scaffold | Vite + TS strict, folder tree, env, aliases (file 02) |
| 2. Theme | `src/theme/` tokens + `createTheme` + provider (file 03) |
| 3. Config | i18n registry, router, status registries, SDK (files 04–07) |
| 4. Primitives | DataTable…QueryState (files 08–19) |
| 5. Features | Assemble 11 modules from config + primitives (file 20 + features) |

---

## Global Definition of Done
- [ ] No hardcoded UI strings — all text via `src/i18n/` (ESLint passes `no-literal-string`).
- [ ] Built from config + existing primitives; no bespoke layout duplication.
- [ ] Responsive 320px → 1440px+; verified Mac/Retina.
- [ ] Cross-browser: Chrome/Edge/Safari/Firefox (last-2) + iOS/Android.
- [ ] WCAG 2.1 AA: keyboard nav, focus, contrast, ARIA.
- [ ] TypeScript strict, zero `any`, zero TS errors.
- [ ] All colors/spacing/type from theme tokens (no hardcoded hex/px).
- [ ] Data fetched via Orval SDK hooks; loading/error via `QueryState`.
- [ ] URL deep-linking for filters/tabs/pagination where applicable.
- [ ] Lint + typecheck + tests green.

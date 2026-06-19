# Harmony EMR — Frontend Architecture

## Confirmed Tech Stack
| Package | Version | Purpose |
|---------|---------|---------|
| React | 19 | UI framework |
| **TypeScript** | 5.x | **Confirmed YES — type safety mandatory for HIPAA-scale EMR** |
| Vite | 8 | Build tool + dev server |
| `@mui/material` + `@emotion/react/styled` | v6 | Component library + theming |
| `lucide-react` | latest | All icons (centralized in `src/config/icons.ts`) |
| `react-router-dom` | v6 | Routing + code splitting |
| `@tanstack/react-query` | v5 | Server state, caching, background sync |
| `zustand` | v5 | Client state (auth session, UI state, active patient) |
| `react-hook-form` | latest | Uncontrolled form state (no re-render per keystroke) |
| `zod` | latest | Schema validation — forms + API payloads |
| `axios` | latest | HTTP client with interceptors |
| `@tanstack/react-virtual` | latest | Virtual scrolling for patient lists, fax queues |
| `dayjs` | latest | Dates + timezone (lightweight; UTC ↔ local) |
| `@stripe/stripe-js` + `@stripe/react-stripe-js` | latest | Payment UI |
| `tabbable` | latest | Focus trap — enumerates all focusable elements |

**Real-time**: TanStack Query polling (`refetchInterval: 30_000`) only for Phase 1. Socket.io deferred to messaging phase (sessions 12–13 TBD).
**Mobile**: iOS/Android app not in current scope.

---

## Guiding Principle
**Single source of truth.** One change in `src/theme/` or `src/config/` propagates everywhere.
No magic strings. No scattered hex codes. No hardcoded role names. Everything referenced from one place.

---

## The "Mind Folder" — `src/theme/` + `src/config/`

These two directories are the control centre of the entire application.

### `src/theme/` — Visual Design Tokens (what things look like)
```
src/theme/
├── palette.ts          ← ALL color tokens (Harmony Blue, Sunlight Yellow, etc.)
├── typography.ts       ← ALL font sizes, families, weights, line-heights
├── shadows.ts          ← Elevation shadows
├── spacing.ts          ← Spacing scale (base: 8px)
├── breakpoints.ts      ← Responsive breakpoints
├── components.ts       ← MUI component-level overrides (Button, Input, etc.)
└── index.ts            ← createTheme() assembles everything → exported as `theme`
```

MUI `ThemeProvider` wraps the entire app at `main.tsx`. Every MUI component inherits from `theme`.
Change a color in `palette.ts` → updates every Button, Badge, Input, Header, Chip across the whole app.

### `src/config/` — Behavioural Constants (what things do and who can do it)
```
src/config/
├── roles.ts            ← ROLE_ADMIN, ROLE_PROVIDER, ROLE_AGENT, ROLE_BILLER, ROLE_GUARDIAN
├── permissions.ts      ← Permission matrix: { [ROLE]: [PERMISSION_KEY, ...] }
├── routes.ts           ← All route path strings (ROUTES.DASHBOARD, ROUTES.PATIENT_CHART, etc.)
├── icons.ts            ← Semantic icon re-exports from lucide-react
├── shortcuts.ts        ← ALL keyboard shortcuts in one place (see Keyboard Accessibility section)
├── appointmentTypes.ts ← All codes, durations, modes, provider-type mapping (NPM, FMM, etc.)
├── insurancePlans.ts   ← Accepted plans list; Medicaid block list
├── formTypes.ts        ← Consent form IDs, labels, conditions
├── notifications.ts    ← Notification event keys + default templates
└── constants.ts        ← App-wide limits (MAX_FUTURE_SLOTS, BALANCE_BLOCK_THRESHOLD, etc.)
```

---

## Full Project Structure (Feature-Slice Design)
```
src/
├── theme/                    ← Design tokens (THE MIND — visual)
├── config/                   ← App constants (THE MIND — behavioural)
│
├── assets/
│   ├── fonts/
│   └── images/
│
├── components/               ← Shared, reusable UI (no business logic)
│   ├── ui/                   ← Atomic: Button, Badge, Avatar, Chip, Loader, Tooltip
│   ├── layout/               ← AppShell, Header, SideNav, PageLayout, SplitPane
│   ├── forms/                ← FormField, SignatureCanvas, FileUpload, OTPInput
│   ├── data/                 ← DataTable, EmptyState, StatusDot, PatientCard
│   └── feedback/             ← Toast, ConfirmDialog, AlertBanner, SkeletonList
│
├── features/                 ← Feature modules (Feature-Slice Design — each is isolated)
│   ├── auth/
│   │   ├── api/              ← auth.service.ts
│   │   ├── components/       ← LoginForm.tsx, OTPInput.tsx, PortalActivation.tsx
│   │   ├── pages/            ← LoginPage.tsx, PortalActivatePage.tsx
│   │   ├── hooks/            ← useLogin.ts, useOTP.ts
│   │   └── index.ts          ← Public exports only
│   ├── patients/
│   │   ├── api/              ← patients.service.ts
│   │   ├── components/       ← PatientCard.tsx, PatientSearch.tsx, ChartTabs.tsx
│   │   ├── pages/            ← PatientListPage.tsx, PatientChartPage.tsx
│   │   ├── hooks/            ← usePatientList.ts, usePatientChart.ts
│   │   └── index.ts
│   ├── appointments/         ← same shape: api/ components/ pages/ hooks/ index.ts
│   ├── referrals/
│   ├── forms/                ← Consent forms, PHQ-9, ROI, intake
│   ├── payments/
│   ├── providers/
│   ├── admin/
│   └── leads/
│
├── hooks/                    ← Shared hooks (cross-feature)
│   ├── useAuth.ts            ← Current user, role, permissions
│   ├── usePermission.ts      ← can(PERMISSION_KEY) → boolean
│   ├── useShortcut.ts        ← Register keyboard shortcut from SHORTCUTS config
│   ├── useKeyboardList.ts    ← Arrow key nav in any list (roving tabIndex)
│   ├── useFocusTrap.ts       ← Trap Tab within modal; restore focus on close
│   ├── useAriaAnnounce.ts    ← Announce dynamic changes to screen readers
│   └── useDebounce.ts
│
├── services/                 ← Base API setup only (feature services live in features/*/api/)
│   └── api.ts                ← Axios instance + JWT interceptor + audit log interceptor
│
├── store/                    ← Client-side global state (Zustand)
│   ├── authStore.ts          ← Current user, session, JWT token
│   ├── uiStore.ts            ← Sidebar state, active modal, toast queue
│   └── patientStore.ts       ← Currently open patient chart context
│
├── router/
│   ├── index.tsx             ← React Router v6 + React.lazy() per feature
│   ├── ProtectedRoute.tsx    ← Auth guard — redirects to /login if no token
│   └── RoleRoute.tsx         ← Role guard — renders 403 if permission missing
│
├── types/                    ← Shared TypeScript interfaces (domain models)
│   ├── patient.ts            ← Patient, Lead, Guardian interfaces
│   ├── appointment.ts        ← Appointment, Provider interfaces
│   ├── forms.ts              ← ConsentForm, PHQ9, ROI interfaces
│   ├── payments.ts           ← Payment, PaymentPlan interfaces
│   └── auth.ts               ← AuthUser, Role, Permission types
│
├── utils/
│   ├── date.ts               ← Date formatting, UTC ↔ local, 12-month rule calc
│   ├── phq9.ts               ← Score calc, severity label, positive screen check
│   ├── insurance.ts          ← Accepted plan check, Medicaid block
│   ├── appointmentType.ts    ← New vs. follow-up recommendation logic
│   └── validation.ts         ← Phone, DOB, email validators (Zod schemas)
│
├── App.tsx                   ← ThemeProvider + QueryClientProvider + RouterProvider
└── main.tsx                  ← ReactDOM.createRoot
```

---

## Theme System Detail

### palette.ts structure
```ts
// src/theme/palette.ts
export const palette = {
  primary: {
    // Harmony Blue family
    100: '#E8EEF5', 200: '#C5D3E8', 300: '#9BB3D6',
    400: '#7294C4', 500: '#4A75B2', 600: '#3A5C8F',
    700: '#2B446D', 800: '#1C2D4A', 900: '#0E1727', 1000: '#070C14',
    main: '#4A75B2',    // 500
    dark: '#3A5C8F',    // 600 — buttons hover
    contrastText: '#fff',
  },
  cta: {
    main: '#F5C842',   // Sunlight Yellow — CTA buttons
    dark: '#D4AA30',
    contrastText: '#0E1727',
  },
  accent: {
    sage: '#6B9E7A',   // Healing Sage
    mint: '#7EC8C8',   // Serene Mint
  },
  neutral: {
    100: '#F7F8FA', 200: '#ECEEF2', 300: '#D8DBE3', 400: '#B8BDC9',
    500: '#8D94A6', 600: '#6B7385', 700: '#4F5668', 800: '#363D4F',
    900: '#1F2535', 1000: '#0D1117',
  },
  semantic: {
    positive: '#22C55E',  // Green — form complete
    warning:  '#F59E0B',  // Amber — form partial
    negative: '#EF4444',  // Red — form pending/error
    info:     '#3B82F6',  // Blue — informational
  },
  dataViz: ['#4A75B2', '#6B9E7A', '#F5C842', '#7EC8C8', '#E07B54', '#9B59B6'],
}
```

### typography.ts structure
```ts
// src/theme/typography.ts
export const typography = {
  fontFamily: '"Inter", "Roboto", sans-serif',  // TBD from Figma — update when confirmed
  fontSize: 14,  // base
  h1: { fontSize: '2rem',   fontWeight: 700, lineHeight: 1.2 },
  h2: { fontSize: '1.5rem', fontWeight: 700, lineHeight: 1.3 },
  h3: { fontSize: '1.25rem',fontWeight: 600, lineHeight: 1.4 },
  h4: { fontSize: '1.125rem',fontWeight: 600, lineHeight: 1.4 },
  h5: { fontSize: '1rem',   fontWeight: 600, lineHeight: 1.5 },
  h6: { fontSize: '0.875rem',fontWeight: 600, lineHeight: 1.5 },
  body1: { fontSize: '0.875rem', lineHeight: 1.5 },
  body2: { fontSize: '0.8125rem', lineHeight: 1.5 },
  caption: { fontSize: '0.75rem', lineHeight: 1.4, color: 'neutral.600' },
  label: { fontSize: '0.75rem', fontWeight: 600, textTransform: 'uppercase', letterSpacing: '0.06em' },
}
```

### icons.ts — semantic naming
```ts
// src/config/icons.ts  — import from here, NOT from lucide-react directly
export {
  LayoutDashboard  as DashboardIcon,
  Users            as PatientsIcon,
  CalendarDays     as ScheduleIcon,
  Inbox            as ReferralsIcon,
  FileInput        as FormsIcon,
  CircleDollarSign as BillingIcon,
  Settings         as SettingsIcon,
  Search           as SearchIcon,
  Plus             as AddIcon,
  BellDot          as NotificationIcon,
  ChevronDown      as ExpandIcon,
  Filter           as FilterIcon,
  Printer          as PrintIcon,
  MessageSquare    as MessageIcon,
  CircleHelp       as HelpIcon,
  ArrowUpRight     as OutboundIcon,
  ArrowDownLeft    as InboundIcon,
  UserRoundSearch  as FindProviderIcon,
  ChartColumnIncreasing as AnalyticsIcon,
} from 'lucide-react'
```

### roles.ts + permissions.ts
```ts
// src/config/roles.ts
export const ROLES = {
  ADMIN:    'admin',        // Office Manager / Admin
  PROVIDER: 'provider',     // MD, APRN, PA, LMHC, LCSW, etc.
  AGENT:    'agent',        // Customer Care Agent
  BILLER:   'biller',       // Billing Department
  GUARDIAN: 'guardian',     // Patient Guardian / Family Member (optional)
}

// src/config/permissions.ts
export const PERMISSIONS = {
  VIEW_PATIENT_CHART:   'view_patient_chart',
  EDIT_PATIENT_CHART:   'edit_patient_chart',
  SCHEDULE_APPOINTMENT: 'schedule_appointment',
  PROCESS_PAYMENT:      'process_payment',
  VIEW_PHQ9:            'view_phq9',
  MANAGE_PROVIDERS:     'manage_providers',
  MANAGE_LOCATIONS:     'manage_locations',
  VIEW_REFERRALS:       'view_referrals',
  MANAGE_FAXES:         'manage_faxes',
  ADMIN_SETTINGS:       'admin_settings',
}

export const ROLE_PERMISSIONS = {
  [ROLES.ADMIN]:    [/* all permissions */],
  [ROLES.PROVIDER]: ['view_patient_chart', 'edit_patient_chart', 'view_phq9', 'schedule_appointment'],
  [ROLES.AGENT]:    ['view_patient_chart', 'schedule_appointment', 'view_referrals', 'manage_faxes'],
  [ROLES.BILLER]:   ['view_patient_chart', 'view_billing', 'process_payment', 'view_insurance'],
  [ROLES.GUARDIAN]: ['view_patient_chart', 'view_consent_forms'],  // Read-only portal access
}
```

---

## Performance Architecture

### Rendering Strategy
| Technique | Where Applied |
|-----------|--------------|
| `React.lazy()` + `Suspense` | Every feature module (code-split at route level) |
| `React.memo()` | PatientCard, AppointmentSlot, ReferralRow (list items) |
| `useMemo` | Derived/filtered lists, PHQ-9 score calc, permission checks |
| `useCallback` | Event handlers passed as props to memoized children |
| Virtual scrolling (`@tanstack/virtual`) | Patient list, appointment history, referral queue |
| `MUI Skeleton` | Loading states for all data-dependent UI |
| Optimistic updates | Status changes, form submissions (via TanStack Query mutations) |

### Server State — TanStack Query (React Query v5)
- All API data fetched and cached via `useQuery` / `useMutation`
- Stale time: 5 min for patient data; 30s for appointment slots
- Background refetch on window focus for appointment calendar
- No manual `useEffect` + `useState` for data fetching

### Client State — Zustand
- `authStore` — current user, role, token
- `uiStore` — sidebar state, active modal, global toast queue
- `patientStore` — currently open patient chart context

### Form State — React Hook Form
- All forms use `react-hook-form` (uncontrolled inputs = no re-render on every keystroke)
- Validation via `zod` schemas — same schema used for API payload validation
- Signature canvas, file uploads, OTP fields wrapped as controlled fields

### Route-Level Code Splitting
```ts
// Every feature is lazy-loaded
const Patients     = lazy(() => import('./features/patients'))
const Appointments = lazy(() => import('./features/appointments'))
const Referrals    = lazy(() => import('./features/referrals'))
// etc.
```

---

## Routing Structure
```
/                          → Redirect → /dashboard (if authed) or /login
/login                     → Auth: Login page
/portal                    → Patient Portal (public landing — unauthenticated)
/portal/activate           → Portal activation (code + DOB)
/portal/book               → Appointment booking (public/pre-auth)
/dashboard                 → Agent/Provider/Admin dashboard
/patients                  → Patient list
/patients/:id              → Patient chart (facesheet)
/patients/:id/appointments → Appointments
/patients/:id/documents    → Documents
/patients/:id/clinical     → Clinical data (vitals, meds, labs, etc.)
/patients/:id/history      → Medical/surgical/family history
/patients/:id/timeline     → Timeline
/appointments              → Schedule view (calendar)
/appointments/book         → Book appointment flow
/referrals                 → Referral queue
/referrals/fax             → Fax inbox
/forms/:patientId          → Consent form list
/billing                   → Billing dashboard
/admin                     → Admin settings
/admin/providers           → Provider management
/admin/locations           → Location management
/admin/notifications       → Notification templates
```

---

## Keyboard Accessibility Architecture

### Why It Matters Here
Agents and providers are power users — they process 50–100 patients/day. Mouse-only interaction is a bottleneck. Keyboard-first design reduces clicks, speeds up workflows, and improves accuracy under high volume. This is not optional — it's a core UX requirement.

---

### Centralized Shortcut Registry — `src/config/shortcuts.ts`
All shortcuts defined in one file. Nothing hardcoded in components.

```ts
// src/config/shortcuts.ts
export const SHORTCUTS = {
  // Global
  GLOBAL_SEARCH:        { key: 'k', mod: 'ctrl', label: 'Search patients / actions' },
  NEW_PATIENT:          { key: 'n', mod: 'ctrl+shift', label: 'New patient' },
  GO_DASHBOARD:         { key: '1', mod: 'alt', label: 'Go to Dashboard' },
  GO_PATIENTS:          { key: '2', mod: 'alt', label: 'Go to Patients' },
  GO_SCHEDULE:          { key: '3', mod: 'alt', label: 'Go to Schedule' },
  GO_REFERRALS:         { key: '4', mod: 'alt', label: 'Go to Referrals' },
  GO_BILLING:           { key: '5', mod: 'alt', label: 'Go to Billing' },
  OPEN_HELP:            { key: '?', mod: 'shift', label: 'Show keyboard shortcuts' },

  // Patient chart (context: chart open)
  CHART_FACESHEET:      { key: 'f', mod: 'alt', label: 'Facesheet' },
  CHART_APPOINTMENTS:   { key: 'a', mod: 'alt', label: 'Appointments tab' },
  CHART_DOCUMENTS:      { key: 'd', mod: 'alt', label: 'Documents tab' },
  CHART_CLINICAL:       { key: 'c', mod: 'alt', label: 'Clinical Data tab' },
  CHART_TIMELINE:       { key: 't', mod: 'alt', label: 'Timeline tab' },
  BOOK_APPOINTMENT:     { key: 'b', mod: 'alt', label: 'Book appointment for this patient' },
  PRINT_CHART:          { key: 'p', mod: 'ctrl', label: 'Print current view' },

  // Modals / Dialogs
  CLOSE_MODAL:          { key: 'Escape', label: 'Close modal / Cancel' },
  CONFIRM_ACTION:       { key: 'Enter', mod: 'ctrl', label: 'Confirm action' },

  // Lists (patient list, referral queue, fax inbox)
  LIST_NEXT:            { key: 'ArrowDown', label: 'Next item in list' },
  LIST_PREV:            { key: 'ArrowUp',   label: 'Previous item in list' },
  LIST_OPEN:            { key: 'Enter',     label: 'Open selected item' },
  LIST_SELECT:          { key: ' ',         label: 'Select/check item' },

  // Calendar / Schedule
  CALENDAR_NEXT:        { key: 'ArrowRight', label: 'Next day/week' },
  CALENDAR_PREV:        { key: 'ArrowLeft',  label: 'Previous day/week' },
  CALENDAR_TODAY:       { key: 't', mod: 'ctrl', label: 'Jump to today' },
}
```

**Change a shortcut key in `shortcuts.js` → updates every listener and the help modal automatically.**

---

### Keyboard Navigation Patterns by Component

| Component | Keys | Behaviour |
|-----------|------|-----------|
| Patient List | `↑` `↓` | Move focus between rows; highlights row |
| Patient List | `Enter` | Open patient chart |
| Patient List | `Space` | Select/check patient (multi-select) |
| Side Navigation | `↑` `↓` | Move between nav items |
| Side Navigation | `Enter` | Navigate to section |
| Calendar slots | `←` `→` | Move between days |
| Calendar slots | `↑` `↓` | Move between time slots |
| Calendar slots | `Enter` | Open slot / book appointment |
| Dropdown/Select | `↑` `↓` | Navigate options |
| Dropdown/Select | `Enter` | Select option |
| Dropdown/Select | `Esc` | Close without selecting |
| Tabs (chart) | `←` `→` | Switch between tabs (ARIA tablist pattern) |
| Modal/Dialog | `Esc` | Close modal; restore focus to trigger element |
| Modal/Dialog | `Tab` | Cycle within modal only (focus trap active) |
| Consent Forms | `Tab` | Move to next field; `Shift+Tab` = back |
| Consent Forms | `Space` | Check checkbox / toggle |
| Consent Forms | `Enter` | Submit if on last field |
| Toast/Alert | `Esc` | Dismiss toast |

---

### Custom Hooks for Keyboard

```
src/hooks/
├── useShortcut.ts       ← Register a single shortcut: useShortcut(SHORTCUTS.GLOBAL_SEARCH, handler)
├── useKeyboardList.ts   ← Arrow key navigation in any list (roving tabIndex)
├── useFocusTrap.ts      ← Trap Tab within a modal; restore focus on unmount
└── useAriaAnnounce.ts   ← Announce dynamic changes to screen readers via live region
```

**`useShortcut` pattern:**
```ts
// Usage in any component — reads key combo from SHORTCUTS config
useShortcut(SHORTCUTS.GLOBAL_SEARCH, () => openCommandPalette())
useShortcut(SHORTCUTS.CLOSE_MODAL, () => onClose())
```

**`useKeyboardList` pattern:**
```ts
// Attach to any list container — handles ArrowUp/Down/Enter/Space
const { focusedIndex, listProps, getItemProps } = useKeyboardList({
  count: patients.length,
  onSelect: (index) => openPatientChart(patients[index].id),
})
```

---

### Focus Management Rules
- **Focus trap**: All modals, drawers, and dialogs must trap `Tab`/`Shift+Tab` within — no escape to background
- **Focus restore**: When modal closes, focus returns to the element that triggered it
- **Focus visible**: Never `outline: none` without a custom visible focus style — MUI theme overrides this in `components.js`
- **Skip-to-content**: `<a href="#main-content">` link visible on first `Tab` press — bypasses header/sidebar
- **Initial focus**: Modals and drawers auto-focus the first actionable element on open
- **Roving tabIndex**: Patient list, referral queue, calendar grid use `tabIndex={0}` on focused item; `-1` on rest

---

### ARIA Requirements
| Pattern | Implementation |
|---------|---------------|
| Navigation landmark | `<nav aria-label="Main Navigation">` on sidebar |
| Main content | `<main id="main-content">` on page body |
| Patient list | `role="listbox"` or `role="grid"` + `aria-activedescendant` |
| Calendar grid | `role="grid"` with `aria-label="Schedule for [date]"` |
| Tabs | `role="tablist"` + `role="tab"` + `role="tabpanel"` |
| Modal | `role="dialog"` + `aria-modal="true"` + `aria-labelledby` |
| Status badge | `aria-label="Form status: Complete"` |
| Form errors | `aria-describedby` linking input to error message |
| Loading states | `aria-live="polite"` region for dynamic content |
| Icon-only buttons | `aria-label="Schedule appointment"` (no visible text) |

---

### Global Command Palette (Ctrl+K)
- Inspired by VS Code / Linear / Notion pattern — familiar to power users
- Searches: patients by name/DOB, providers, actions (Book Appointment, New Referral, etc.)
- Keyboard-only: `↑` `↓` to navigate results, `Enter` to execute, `Esc` to close
- Implemented in `src/components/ui/CommandPalette.tsx`
- Registered globally in `App.jsx` via `useShortcut(SHORTCUTS.GLOBAL_SEARCH, ...)`

---

### Shortcut Help Modal (Shift+?)
- Press `Shift+?` anywhere → shows all available shortcuts for current context
- Context-aware: shows global shortcuts always; adds chart shortcuts when chart is open
- Built from `SHORTCUTS` config automatically — no manual maintenance

---

## State & Data Flow Summary
```
Figma Tokens
    ↓
src/theme/        → MUI ThemeProvider → All components auto-inherit
src/config/       → Imported by components, hooks, utils — one source

API (REST)
    ↓
services/*.js     → Axios calls
    ↓
TanStack Query    → Cache + background sync + optimistic updates
    ↓
features/*        → Feature components consume query hooks
    ↓
components/       → Atomic UI renders from theme tokens
```

---

## Security & HIPAA Architecture

### Axios Interceptors (`src/services/api.ts`)
Two request/response interceptors handle all cross-cutting concerns:

**1. JWT Auth Interceptor** (request)
- Attaches `Authorization: Bearer <token>` to every request
- On 401 response → clears `authStore` → redirects to `/login`

**2. Audit Log Interceptor** (response)
- Every API call that touches PHI records (`/patients`, `/appointments`, `/forms`, `/payments`, etc.) auto-logs to backend audit trail
- Captures: action, endpoint, user ID, role, timestamp, IP address
- Satisfies HIPAA requirement: all PHI access must be logged with who/when/source

### Route Guards
```
ProtectedRoute  → checks authStore.isAuthenticated → else redirect /login
RoleRoute       → checks usePermission(PERMISSION_KEY) → else render <Forbidden403 />
```

### Component-Level Permission Guard
```tsx
// Usage in any component
<CanAccess permission={PERMISSIONS.VIEW_PHQ9}>
  <PHQ9Score score={score} />
</CanAccess>
```
PHQ-9 scores only visible to providers. Billing sees balance. Patients see their own records only.

### Data Rules
| Rule | Implementation |
|------|---------------|
| No raw card data | Stripe tokenization — EHR stores `cardToken` (Stripe reference) only |
| No permanent deletion | All deletes are soft-deletes (`isArchived = true`) + audit entry |
| PHI in transit | HTTPS only; Axios forces `https://` base URL |
| Session timeout | `authStore` clears after inactivity (configurable, default 30 min) |
| PHQ-9 immutability | Backend enforces; frontend disables edit controls after same-day window |

---

## Package Dependencies (to be installed)
| Package | Purpose |
|---------|---------|
| `@mui/material` + `@emotion/react` + `@emotion/styled` | Component library + theming |
| `lucide-react` | Icons |
| `react-router-dom` v6 | Routing |
| `@tanstack/react-query` v5 | Server state / data fetching |
| `zustand` | Client state |
| `react-hook-form` | Form state |
| `zod` | Schema validation (forms + API) |
| `axios` | HTTP client |
| `@tanstack/react-virtual` | Virtual scrolling for long lists |
| `dayjs` | Date manipulation (lightweight vs moment) |
| `@stripe/stripe-js` + `@stripe/react-stripe-js` | Payment UI |
| `tabbable` | Focus trap utility — finds all focusable elements in a container |

### Install Command
```bash
npm create vite@latest harmony-emr -- --template react-ts
cd harmony-emr
npm install \
  @mui/material @emotion/react @emotion/styled \
  lucide-react \
  react-router-dom \
  @tanstack/react-query \
  zustand \
  react-hook-form \
  zod \
  axios \
  @tanstack/react-virtual \
  dayjs \
  @stripe/stripe-js @stripe/react-stripe-js \
  tabbable
```

# Navbar & Routing Architecture — HUPC Provider Portal

## Table of Contents

1. [High-Level Overview](#1-high-level-overview)
2. [File Map](#2-file-map)
3. [Data Flow Diagram](#3-data-flow-diagram)
4. [Route Configuration (`src/config/routes.ts`)](#4-route-configuration)
5. [Router Setup (`src/router/routes.tsx`)](#5-router-setup)
6. [Auth Guards](#6-auth-guards)
   - [ProtectedRoute](#61-protectedroute)
   - [PublicRoute](#62-publicroute)
7. [Auth Store (`src/store/authStore.ts`)](#7-auth-store)
8. [UI Store (`src/store/uiStore.ts`)](#8-ui-store)
9. [Layouts](#9-layouts)
   - [MainLayout](#91-mainlayout)
   - [AuthLayout](#92-authlayout)
10. [useMenu Hook (`src/hooks/useMenu.tsx`)](#10-usemenu-hook)
11. [Icon System (`src/config/icons.ts`)](#11-icon-system)
12. [TopMenu Component (`src/components/layout/TopMenu.tsx`)](#12-topmenu-component)
13. [Navbar Component (`src/components/layout/Navbar.tsx`)](#13-navbar-component)
14. [Complete Request Lifecycle](#14-complete-request-lifecycle)
15. [How to Add a New Route](#15-how-to-add-a-new-route)

---

## 1. High-Level Overview

The app uses **React Router v6** with a **layout route** pattern. Every URL the user visits goes through this chain:

```
URL hit
  → React Router matches a route
    → Auth guard checks authentication (ProtectedRoute / PublicRoute)
      → Layout renders (MainLayout with Navbar / AuthLayout)
        → Lazy-loaded page component renders inside <Outlet />
```

There are **three route groups**:
- **Root (`/`)** — Redirects to `/dashboard`
- **Public routes** — Login page (accessible only when NOT logged in)
- **Protected routes** — All app pages (accessible only when logged in)
- **Catch-all (`*`)** — 404 Not Found page

---

## 2. File Map

```
src/
├── config/
│   ├── routes.ts          ← All URL path constants (single source of truth)
│   └── icons.ts           ← All icon imports (single source of truth)
├── store/
│   ├── authStore.ts       ← Zustand store: token, user, login(), logout()
│   └── uiStore.ts         ← Zustand store: sidebar open/close state
├── router/
│   ├── routes.tsx          ← createBrowserRouter() — defines all routes
│   ├── ProtectedRoute.tsx  ← Auth guard: redirects to /login if not authenticated
│   └── PublicRoute.tsx     ← Auth guard: redirects to /dashboard if already authenticated
├── layouts/
│   ├── MainLayout.tsx      ← Navbar + <Outlet /> (for protected pages)
│   └── AuthLayout.tsx      ← Centered container + <Outlet /> (for login)
├── hooks/
│   └── useMenu.tsx         ← Returns the navbar menu items array
├── components/layout/
│   ├── Navbar.tsx           ← Full navbar: logo, TopMenu, search, profile, mobile drawer
│   └── TopMenu.tsx          ← Renders the navigation tabs/links
└── App.tsx                  ← Root: QueryClient + ThemeProvider + RouterProvider
```

---

## 3. Data Flow Diagram

```
                          ┌──────────────┐
                          │   App.tsx     │
                          │ (Providers)   │
                          └──────┬───────┘
                                 │
                          ┌──────▼───────┐
                          │  RouterProvider│
                          │ (routes.tsx)  │
                          └──────┬───────┘
                                 │
               ┌─────────────────┼─────────────────┐
               │                 │                  │
        ┌──────▼──────┐  ┌──────▼──────┐   ┌──────▼──────┐
        │  / (redirect)│  │ PublicRoute  │   │ProtectedRoute│
        │→ /dashboard  │  │   guard     │   │   guard      │
        └─────────────┘  └──────┬──────┘   └──────┬──────┘
                                │                  │
                         ┌──────▼──────┐   ┌──────▼──────┐
                         │ AuthLayout   │   │ MainLayout   │
                         │ (centered)   │   │(Navbar+page) │
                         └──────┬──────┘   └──────┬──────┘
                                │                  │
                         ┌──────▼──────┐   ┌──────▼──────┐
                         │  LoginPage   │   │  <Outlet />  │
                         └─────────────┘   │ (lazy page)  │
                                           └─────────────┘
```

---

## 4. Route Configuration

**File:** `src/config/routes.ts`

```ts
export const ROUTES = {
  HOME: '/',
  LOGIN: '/login',
  DASHBOARD: '/dashboard',
  LEADS: '/leads',
  PATIENTS: '/patients',
  PATIENT_CHART: '/patients/:id',
  SCHEDULING: '/appointments',
  GROUPS: '/groups',
  REFERRALS: '/referrals',
  COMMUNICATION: '/communication',
  BILLING: '/billing',
  REPORTS: '/reports',
  SETTINGS: '/settings',
  NOT_AUTHORIZED: '/not-authorized',
};
```

**Purpose:** Single source of truth for all URL paths. Every file that references a URL path imports from here — never hardcoded strings.

**Used by:**
- `useMenu.tsx` — maps each menu item to its route
- `routes.tsx` — could be used here too (currently uses string literals for readability)
- Any `navigate()` call or `<NavLink to=...>` across the app

---

## 5. Router Setup

**File:** `src/router/routes.tsx`

This is the **central routing file** that defines the entire route tree using `createBrowserRouter()`.

### Key Concepts

**Lazy Loading:** Every page is wrapped with `lazy(() => import(...))`. This means the JS bundle for each page is only downloaded when the user first navigates to it — not upfront. This keeps the initial bundle small.

```ts
const DashboardPage = lazy(() => import('../features/dashboard/DashboardPage'));
```

**LazyPage wrapper:** A thin `<Suspense>` wrapper that shows "Loading..." while the lazy chunk is downloading:

```tsx
const LazyPage = ({ children }: { children: ReactNode }) => (
  <Suspense fallback={<div>Loading...</div>}>
    {children}
  </Suspense>
);
```

### Route Structure

```tsx
createBrowserRouter([
  // 1. Root redirect
  { path: '/', element: <Navigate to="/dashboard" replace /> },

  // 2. Public routes (login)
  {
    element: <PublicRoute><AuthLayout /></PublicRoute>,
    children: [
      { path: 'login', element: <LazyPage><LoginPage /></LazyPage> },
    ],
  },

  // 3. Protected routes (all app pages)
  {
    element: <ProtectedRoute><MainLayout /></ProtectedRoute>,
    children: [
      { path: 'dashboard', element: <LazyPage><DashboardPage /></LazyPage> },
      { path: 'leads', element: <LazyPage><LeadsPage /></LazyPage> },
      // ... all other pages
    ],
  },

  // 4. 404 catch-all
  { path: '*', element: <LazyPage><NotFound /></LazyPage> },
]);
```

### How Layout Routes Work

The **layout route** pattern uses an `element` without a `path`. React Router renders this element, and the matching child route renders inside the `<Outlet />` within that element.

```
URL: /dashboard
  → Matches protected group (no path on parent)
    → ProtectedRoute checks auth → passes
      → MainLayout renders (Navbar + <Outlet />)
        → child route "dashboard" matches → DashboardPage renders in <Outlet />
```

---

## 6. Auth Guards

### 6.1 ProtectedRoute

**File:** `src/router/ProtectedRoute.tsx`

```tsx
const ProtectedRoute = ({ children }: { children: ReactNode }) => {
  const { isAuthenticated } = useAuthStore();
  const location = useLocation();

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
};
```

**What it does:**
1. Reads `isAuthenticated` from the Zustand auth store
2. If **not authenticated** → redirects to `/login` with `state.from` (so login page can redirect back after success)
3. If **authenticated** → renders `children` (which is `<MainLayout />`)

**The `state={{ from: location }}` trick:** When the user logs in, the login page can read `location.state.from` and redirect the user back to the page they originally wanted.

### 6.2 PublicRoute

**File:** `src/router/PublicRoute.tsx`

```tsx
const PublicRoute = ({ children }: { children: ReactNode }) => {
  const { isAuthenticated } = useAuthStore();

  if (isAuthenticated) {
    return <Navigate to="/dashboard" replace />;
  }

  return children;
};
```

**What it does:**
1. If **already authenticated** → redirects to `/dashboard` (prevents logged-in users from seeing the login page)
2. If **not authenticated** → renders `children` (which is `<AuthLayout />`)

### How They Work Together

| User State      | Visits `/dashboard`       | Visits `/login`            |
|-----------------|---------------------------|----------------------------|
| Logged in       | Renders DashboardPage     | Redirects to `/dashboard`  |
| Not logged in   | Redirects to `/login`     | Renders LoginPage          |

---

## 7. Auth Store

**File:** `src/store/authStore.ts`

Uses **Zustand** (lightweight state management). This is the single source of truth for authentication state.

### State Shape

| Field             | Type          | Description                        |
|-------------------|---------------|------------------------------------|
| `token`           | string | null | JWT auth token                     |
| `user`            | User | null   | User object (firstName, lastName, email, role) |
| `isAuthenticated` | boolean       | Derived from token presence        |

### Actions

**`login(token, user)`:**
1. Saves token and user to `localStorage` (persists across page refreshes)
2. Updates Zustand state → triggers re-render of any component reading `isAuthenticated`
3. ProtectedRoute/PublicRoute react to the change and redirect accordingly

**`logout()`:**
1. Clears `localStorage`
2. Resets state to null/false
3. Any ProtectedRoute-wrapped page immediately redirects to `/login`

### Initialization
On app startup, the store reads from `localStorage`:
```ts
token: localStorage.getItem('authToken') || null,
user: JSON.parse(localStorage.getItem('user') || 'null'),
isAuthenticated: !!localStorage.getItem('authToken'),
```
This means if the user refreshes the page, they stay logged in.

---

## 8. UI Store

**File:** `src/store/uiStore.ts`

Controls UI state that needs to be shared across components.

| Field          | Type    | Description                   |
|----------------|---------|-------------------------------|
| `sidebarOpen`  | boolean | Whether the mobile sidebar/drawer is open |
| `toggleSidebar()` | fn  | Toggles the value             |
| `setSidebarOpen(open)` | fn | Sets explicitly           |

Currently used by the Navbar for the mobile drawer.

---

## 9. Layouts

### 9.1 MainLayout

**File:** `src/layouts/MainLayout.tsx`

```
┌─────────────────────────────────────────────┐
│                  Navbar                       │  ← sticky at top
├─────────────────────────────────────────────┤
│                                              │
│              <Outlet />                      │  ← flex: 1, overflow: auto
│         (page content renders here)          │
│                                              │
└─────────────────────────────────────────────┘
```

- Uses `height: '100vh'` with `flexDirection: 'column'`
- Navbar is sticky at top
- Main content area takes remaining space (`flex: 1`) and scrolls independently (`overflow: 'auto'`)
- Background color from `palette.background.default`

### 9.2 AuthLayout

**File:** `src/layouts/AuthLayout.tsx`

```
┌─────────────────────────────────────────────┐
│                                              │
│                                              │
│              <Outlet />                      │  ← centered both axes
│          (login form renders here)           │
│                                              │
│                                              │
└─────────────────────────────────────────────┘
```

- Full viewport height (`100vh`)
- Content centered with flexbox (both horizontal and vertical)
- No navbar — clean authentication experience

---

## 10. useMenu Hook

**File:** `src/hooks/useMenu.tsx`

Returns an array of `MenuItem` objects that define the navbar tabs:

```ts
interface MenuItem {
  title: string;      // Display text ("Dashboard", "Leads", etc.)
  route: string;      // URL path from ROUTES config
  icon: ReactNode;    // Icon component from config/icons.ts
}
```

**Items returned (in order):**

| # | Title         | Route          | Icon             |
|---|---------------|----------------|------------------|
| 1 | Dashboard     | /dashboard     | DashboardIcon    |
| 2 | Leads         | /leads         | LeadsIcon        |
| 3 | Patients      | /patients      | PatientsIcon     |
| 4 | Scheduling    | /appointments  | ScheduleIcon     |
| 5 | Groups        | /groups        | GroupsIcon       |
| 6 | Referrals     | /referrals     | ReferralsIcon    |
| 7 | Communication | /communication | CommunicationIcon|
| 8 | Billing       | /billing       | BillingIcon      |
| 9 | Reports       | /reports       | ReportsIcon      |
|10 | Settings      | /settings      | SettingsIcon     |

**Why a hook?** In the future, this can be extended to filter menu items based on user role/permissions (e.g., hide Billing for non-admin roles).

---

## 11. Icon System

**File:** `src/config/icons.ts`

All icons are re-exported from `lucide-react` with **semantic aliases**:

```ts
export {
  LayoutDashboard as DashboardIcon,
  Users as PatientsIcon,
  UserPlus as LeadsIcon,
  // ... etc
} from 'lucide-react';
```

**Why re-export?**
- If we ever switch icon libraries, we only change this one file
- Semantic names (`BillingIcon`) are clearer than generic ones (`CircleDollarSign`)
- Every component imports from `config/icons.ts`, never directly from `lucide-react`

---

## 12. TopMenu Component

**File:** `src/components/layout/TopMenu.tsx`

Renders the navigation items from `useMenu()`. Supports **two modes**:

### Props

| Prop           | Type       | Default | Description                     |
|----------------|------------|---------|---------------------------------|
| `openInDrawer` | boolean    | false   | Switches between desktop/mobile style |
| `onItemClick`  | () => void | -       | Called when an item is clicked (to close drawer) |

### Desktop Mode (`openInDrawer = false`)

```
[ 📊 Dashboard ][ 👥 Leads ][ 🏥 Patients ][ 📅 Scheduling ] ...
```

- Horizontal flex row
- Active tab: `palette.harmonyBlue.main` (#2D5F8D) background, full white text
- Inactive tabs: transparent background, white text at 70% opacity
- Rounded corners: `8px` (all corners)
- Hover: subtle white overlay (`alpha(white, 0.1)`)

### Mobile/Drawer Mode (`openInDrawer = true`)

```
┌──────────────────────┐
│ 📊 Dashboard          │
│ 👥 Leads              │  ← active item highlighted
│ 🏥 Patients           │
│ ...                   │
└──────────────────────┘
```

- Vertical column, full width
- Active: `palette.harmonyBlue[50]` background, blue text
- Inactive: transparent, dark text
- Rounded corners: `8px` (all corners)

### Active Route Detection

```ts
const isActiveRoute =
  location.pathname === item.route ||
  location.pathname.startsWith(item.route + '/');
```

This means `/patients` is active when visiting `/patients` OR `/patients/123` (nested routes).

### Typography

All tab labels use the Inter font at `12px`, `500 weight`, `120% line-height` — matching the Figma design spec.

---

## 13. Navbar Component

**File:** `src/components/layout/Navbar.tsx`

The main header bar. This is the most complex UI component.

### Visual Layout

```
┌──────────────────────────────────────────────────────────────────┐
│ [☰] [LOGO]     [ Dashboard | Leads | Patients | ... ]     🔍 ❓ 🔔 │ 👤 ▾ │
└──────────────────────────────────────────────────────────────────┘
  ↑                        ↑                                  ↑         ↑
  Mobile only         Desktop TopMenu              Action icons    Profile
  (hamburger)         (hidden on mobile)                        dropdown
```

### Sections

**LEFT — Logo + Mobile Toggle:**
- Hamburger icon (visible only on `xs`–`md`, hidden on `lg+`)
- HUPC SVG logo (`src/assets/logo/HUPC_logo_svg.svg`), clickable → navigates to `/dashboard`

**CENTER — Desktop Navigation:**
- `<TopMenu />` component (visible only on `lg+`)
- Takes `flex: 1` to fill available space, items centered

**RIGHT — Action Icons + Profile:**
- **Search** (🔍) — Ctrl+K shortcut tooltip
- **Help** (❓)
- **Notifications** (🔔) — with red badge showing count (currently hardcoded to 3)
- **Vertical divider** — 1px line in `palette.harmonyBlue.main`
- **Profile avatar** — Shows `<ProfileIcon />` (person silhouette) inside a circle
  - Circle border: `palette.harmonyBlue.light` (#5992C7)
  - Circle fill: `palette.harmonyBlue.main` (#2D5F8D)
- **Expand chevron** (▾) — same `palette.harmonyBlue.light` color

### Profile Dropdown Menu

Clicking the profile area opens a MUI `<Menu>`:
- **Profile** — navigates to `/profile`
- **Logout** — opens confirmation dialog

### Logout Confirmation Dialog

```
┌────────────────────────┐
│  Logging Out            │
│  Are you sure you want  │
│  to log out?            │
│                         │
│  [Cancel]  [Yes, Logout]│
└────────────────────────┘
```

Clicking "Yes, Logout":
1. Calls `authStore.logout()` — clears localStorage + Zustand state
2. Navigates to `/login`
3. ProtectedRoute guard prevents going back to protected pages

### Mobile Drawer

On small screens, the hamburger icon opens a `<Drawer>` from the left:

```
┌────────────────────────┐
│  Harmony United         │
│  Psychiatric Care       │
│  ───────────────────── │
│  📊 Dashboard           │
│  👥 Leads               │
│  🏥 Patients            │
│  📅 Scheduling          │
│  ...                    │
└────────────────────────┘
```

Uses `<TopMenu openInDrawer onItemClick={() => setMobileMenuOpen(false)} />`.
The `onItemClick` prop closes the drawer when a nav item is clicked.

### Styling Details

| Element               | Color                                  |
|-----------------------|----------------------------------------|
| AppBar background     | `palette.harmonyBlue.dark` (#042646)   |
| Bottom border         | `palette.primary.dark`                 |
| Icon default color    | `alpha(white, 0.8)` — 80% white       |
| Icon hover            | `palette.common.white` — full white    |
| Hover overlay         | `alpha(white, 0.1)` — subtle highlight |
| Profile circle border | `palette.harmonyBlue.light` (#5992C7)  |
| Profile circle fill   | `palette.harmonyBlue.main` (#2D5F8D)   |
| Divider line          | `palette.harmonyBlue.main` (#2D5F8D)   |
| Expand chevron        | `palette.harmonyBlue.light` (#5992C7)  |

---

## 14. Complete Request Lifecycle

### Example: User visits `/patients` while logged in

```
1. Browser URL: /patients

2. RouterProvider matches routes:
   → "/" redirect? No, path is /patients
   → PublicRoute group? No path match for "patients" under it
   → ProtectedRoute group? Yes, "patients" matches

3. ProtectedRoute renders:
   → useAuthStore().isAuthenticated === true ✓
   → Renders children: <MainLayout />

4. MainLayout renders:
   → <Navbar /> (sticky top bar)
   → <Outlet /> (React Router fills this with the matched child)

5. React Router renders PatientsPage inside <Outlet />:
   → lazy(() => import('../features/patients/PatientsPage'))
   → Suspense shows "Loading..." until JS chunk downloads
   → PatientsPage renders

6. Navbar renders:
   → useMenu() returns 10 items
   → TopMenu maps over items
   → "/patients" matches item.route → that tab is highlighted
```

### Example: User visits `/dashboard` while NOT logged in

```
1. Browser URL: /dashboard

2. Router matches ProtectedRoute group

3. ProtectedRoute renders:
   → useAuthStore().isAuthenticated === false ✗
   → Returns <Navigate to="/login" state={{ from: { pathname: '/dashboard' } }} />

4. Browser redirects to /login

5. Router matches PublicRoute group

6. PublicRoute renders:
   → isAuthenticated === false ✓
   → Renders <AuthLayout />

7. AuthLayout renders <Outlet /> → LoginPage renders

8. User logs in → authStore.login(token, user) called
   → isAuthenticated becomes true
   → ProtectedRoute re-evaluates → now passes
   → PublicRoute re-evaluates → redirects away from /login to /dashboard
```

---

## 15. How to Add a New Route

**Example: Adding a "Tasks" page**

### Step 1: Add the route path

```ts
// src/config/routes.ts
export const ROUTES = {
  // ... existing routes
  TASKS: '/tasks',
};
```

### Step 2: Create the page component

```ts
// src/features/tasks/TasksPage.tsx
const TasksPage = () => {
  return <div>Tasks</div>;
};
export default TasksPage;
```

### Step 3: Add the lazy import and route

```ts
// src/router/routes.tsx
const TasksPage = lazy(() => import('../features/tasks/TasksPage'));

// Inside the protected children array:
{ path: 'tasks', element: <LazyPage><TasksPage /></LazyPage> },
```

### Step 4: Add an icon alias (if needed)

```ts
// src/config/icons.ts
export {
  // ... existing
  CheckSquare as TasksIcon,
} from 'lucide-react';
```

### Step 5: Add to the navbar menu

```ts
// src/hooks/useMenu.tsx
{
  title: 'Tasks',
  route: ROUTES.TASKS,
  icon: <TasksIcon size={18} />,
},
```

That's it — the new page will appear in the navbar, be protected by auth, and lazy-load on first visit.

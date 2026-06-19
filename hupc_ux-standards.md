# Harmony EMR — UX Standards & Development Checklist

*These standards are MANDATORY for all development. Every component, page, and feature MUST comply before merge.*

---

## Global Application UX Checklist

| # | Standard | Implementation |
|---|----------|----------------|
| 1 | All elements accessible via **Tab key** navigation | Every interactive element (buttons, links, inputs, dropdowns) must be reachable via `Tab` / `Shift+Tab`. Use semantic HTML (`<button>`, `<a>`, `<input>`) — never `<div onClick>` without `tabIndex` and `role` |
| 2 | Forms navigable via **Arrow keys** (Up/Down/Left/Right) | Radio groups, select dropdowns, date pickers, and grouped controls must support arrow key navigation. Use `useKeyboardList` hook for custom lists |
| 3 | **Enter key** moves to next field or submits form | `Enter` on text inputs → focus next field. `Enter` on last field or submit button → submit form. Use `react-hook-form` `handleSubmit` pattern |
| 4 | **Esc key** closes popup/modal | Every modal, dialog, drawer, dropdown, and popover must close on `Esc`. Use `useFocusTrap` hook which handles this automatically |
| 5 | **Consistent font, colors, spacing** across system | All values from `src/theme/` tokens only — never hardcoded hex, px, or font strings. Use `theme.palette.*`, `theme.spacing()`, `theme.typography.*` |
| 6 | **Loading spinner** when data is loading | Show `MUI Skeleton` or `CircularProgress` during all async operations (API calls, file uploads, navigation). Use TanStack Query's `isLoading` / `isFetching` states |
| 7 | **Success message** after save/update | Show toast/snackbar (green, `semantic.positive`) after every successful mutation — "Patient updated", "Appointment scheduled", etc. Use `uiStore.toast()` |
| 8 | **Confirmation dialog** before delete/archive | All destructive actions (delete, archive, cancel appointment, void payment) must show `ConfirmDialog` with clear description of consequence before executing |
| 9 | Required fields show **red asterisk (*)** | All required form fields must display `*` next to label in `semantic.negative` red. Use `FormField` component which handles this via `required` prop |
| 10 | **Error messages below field** | Validation errors appear directly below the offending input, in `semantic.negative` red, with `aria-describedby` linking input to error. Never use alert boxes for field errors |
| 11 | **30-minute inactivity timeout** → auto logout | `authStore` tracks last activity timestamp. After 30 min idle, clear session, show "Session expired" message, redirect to `/login`. Reset timer on any user interaction (click, keypress, scroll) |
| 12 | **Tooltips on all icons** | Every icon-only button and standalone icon must have a `Tooltip` wrapper with descriptive text + `aria-label`. No icon without explanation |
| 13 | **Responsive design** (Laptop + Tablet) | All pages must work at Desktop (1024–1439px) and Tablet (768–1023px) breakpoints minimum. Use MUI `Grid`, `useMediaQuery`, and brand breakpoints. Test at both sizes |

---

## Implementation Patterns

### 1. Tab Navigation
```tsx
// CORRECT — semantic HTML, naturally focusable
<Button onClick={handleSave}>Save</Button>

// WRONG — div is not focusable by default
<div onClick={handleSave}>Save</div>
```

### 2. Arrow Key Navigation in Forms
```tsx
// Use useKeyboardList for grouped controls
const { focusedIndex, listProps, getItemProps } = useKeyboardList({
  count: options.length,
  onSelect: (i) => selectOption(options[i]),
})
```

### 3. Enter Key Behavior
```tsx
// react-hook-form handles Enter → submit on last field
// For multi-field Enter → next, add onKeyDown to each field:
const handleEnterNext = (e, nextRef) => {
  if (e.key === 'Enter') { e.preventDefault(); nextRef.current?.focus(); }
}
```

### 4. Esc Key — Modal Close
```tsx
// useFocusTrap handles Esc automatically
// For custom popups, register via useShortcut:
useShortcut(SHORTCUTS.CLOSE_MODAL, onClose)
```

### 5. Consistent Theming
```tsx
// CORRECT — uses theme tokens
<Box sx={{ color: 'primary.main', p: 2, borderRadius: 2 }}>

// WRONG — hardcoded values
<Box sx={{ color: '#2D5F8D', padding: '16px', borderRadius: '8px' }}>
```

### 6. Loading States
```tsx
const { data, isLoading } = useQuery({ queryKey: ['patients'], queryFn: fetchPatients })

if (isLoading) return <SkeletonList count={5} />
```

### 7. Success Toast
```tsx
const mutation = useMutation({
  mutationFn: updatePatient,
  onSuccess: () => {
    toast.success('Patient updated successfully')
    queryClient.invalidateQueries({ queryKey: ['patients'] })
  },
  onError: (err) => toast.error(err.message),
})
```

### 8. Confirm Before Delete
```tsx
<ConfirmDialog
  open={showConfirm}
  title="Archive Patient"
  message="This will remove the patient from active lists. Continue?"
  confirmLabel="Archive"
  severity="warning"
  onConfirm={handleArchive}
  onCancel={() => setShowConfirm(false)}
/>
```

### 9. Required Field Asterisk
```tsx
// FormField component auto-handles this
<FormField label="First Name" required error={errors.firstName?.message}>
  <TextField {...register('firstName')} />
</FormField>
// Renders: "First Name *" with red asterisk
```

### 10. Error Messages Below Field
```tsx
// react-hook-form + zod — errors auto-appear below field
<TextField
  {...register('email')}
  error={!!errors.email}
  helperText={errors.email?.message}  // "Valid email required"
  aria-describedby="email-error"
/>
```

### 11. Session Timeout (30 min)
```tsx
// authStore handles globally
// Tracks: lastActivity timestamp
// Resets on: click, keypress, mousemove, scroll
// On timeout: clearSession() → navigate('/login') → toast('Session expired')
// Config: SESSION_TIMEOUT_MS = 30 * 60 * 1000 (in src/config/constants.ts)
```

### 12. Icon Tooltips
```tsx
// CORRECT — tooltip + aria-label
<Tooltip title="Schedule appointment">
  <IconButton aria-label="Schedule appointment">
    <ScheduleIcon />
  </IconButton>
</Tooltip>

// WRONG — bare icon with no context
<IconButton><ScheduleIcon /></IconButton>
```

### 13. Responsive Design
```tsx
// Use MUI Grid with breakpoints
<Grid container spacing={3}>
  <Grid size={{ xs: 12, md: 6, lg: 4 }}>
    <PatientCard />
  </Grid>
</Grid>

// Use useMediaQuery for conditional rendering
const isTablet = useMediaQuery(theme.breakpoints.down('lg'))
```

---

## Pre-Merge Validation Checklist

Before any PR merge, verify ALL 13 standards:

- [ ] Tab through entire page — every control reachable, focus ring visible
- [ ] Arrow keys work in radio groups, selects, and list controls
- [ ] Enter key advances/submits forms correctly
- [ ] Esc closes all modals, drawers, dropdowns, and popovers
- [ ] No hardcoded colors, fonts, or spacing — theme tokens only
- [ ] Loading states visible during all async operations
- [ ] Success toasts shown for all save/update/create actions
- [ ] Confirm dialog appears before every destructive action
- [ ] Required fields have red asterisk
- [ ] Validation errors display below their fields with `aria-describedby`
- [ ] 30-min idle timeout redirects to login
- [ ] All icon-only buttons have tooltips and `aria-label`
- [ ] Page renders correctly at 1024px (desktop) and 768px (tablet)

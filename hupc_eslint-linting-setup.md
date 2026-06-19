# ESLint + Husky + Lint-Staged Setup

## Overview

This project uses a **strict pre-commit linting system** to ensure code quality. Every commit is automatically scanned — if there are linting errors, the commit is blocked until they're fixed.

## How It Works

```
git commit
  ↓
.husky/pre-commit hook runs
  ↓
npx lint-staged
  ↓
For each staged *.{ts,tsx} file:
  → eslint (strict check)
  ↓
If all files pass → commit proceeds
If any file fails → commit BLOCKED + error message shown
                  Developer must fix manually
```

## Files

| File | Purpose |
|------|---------|
| [eslint.config.js](../../eslint.config.js) | ESLint configuration with strict rules |
| [.husky/pre-commit](../../.husky/pre-commit) | Git hook that runs lint-staged before commit |
| [package.json](../../package.json) | Contains `lint-staged` config and husky setup |

## ESLint Rules

### Errors (must fix)

| Rule | Severity | Description |
|------|----------|-------------|
| `no-console` | Error | Strips console.log/info/debug (only warn/error allowed) |
| `no-debugger` | Error | Blocks debugger statements |
| `no-unused-vars` | Error | Detects unused variables (prefix with `_` to ignore) |
| `@typescript-eslint/no-explicit-any` | Error | Bans `any` type |
| `eqeqeq` | Error | Requires `===` instead of `==` |
| `no-var` | Error | Use `const`/`let`, never `var` |
| `prefer-const` | Error | Prefer `const` over `let` when not reassigned |
| `prefer-arrow-callback` | Error | Use arrow functions in callbacks |
| `no-empty-function` | Error | Block empty function bodies |
| `no-implicit-coercion` | Error | Use `Boolean()` instead of `!!` |

### Warnings

| Rule | Severity | Description |
|------|----------|-------------|
| `@typescript-eslint/no-non-null-assertion` | Warning | Warns on `!` non-null assertions |

### Disabled (for routes.tsx)

| Rule | Reason |
|------|--------|
| `react-refresh/only-export-components` | Routes file exports non-component objects; disabling for this file |

## Setup Details

### Husky
- Installed: `npm install -D husky lint-staged`
- Initialized: `npx husky init`
- Creates `.husky/` directory with git hooks

### Lint-Staged
- Runs only on **staged files** (files you `git add`)
- **Does NOT auto-fix** — only reports errors
- Developer must manually fix all issues
- Config in `package.json`:
```json
"lint-staged": {
  "*.{ts,tsx}": [
    "eslint"
  ]
}
```

### ESLint
- **Parser**: `@typescript-eslint/parser` (understands TypeScript)
- **Plugin**: `@typescript-eslint/eslint-plugin` (TypeScript rules)
- **Target**: `**/*.{ts,tsx}` files
- **Strict**: `--max-warnings=0` (any warning fails the build)

## Manual Commands

Run linting on all files (NO auto-fix):
```bash
npm run lint
```

Check specific file:
```bash
npx eslint src/components/Navbar.tsx
```

Check staged files only (what pre-commit uses):
```bash
npx lint-staged
```

## Common Issues & Fixes

### Issue: "Forbidden non-null assertion" warning

**Solution**: Replace `!` with explicit null checks:
```ts
// ❌ Bad
const el = document.getElementById('root')!;

// ✅ Good
const el = document.getElementById('root');
if (!el) throw new Error('Root element not found');
```

### Issue: "is defined but never used"

**Solution 1**: Remove the import/variable

**Solution 2**: Prefix with `_` if you want to keep it:
```ts
const _unused = value;  // Allows unused
```

### Issue: Commit blocked by ESLint

This means there are lint errors. View them with:
```bash
npm run lint
```

Fix errors:
```bash
npx eslint src --fix
```

Stage the fixed files:
```bash
git add src/
```

Try committing again.

### Issue: "console.log not allowed"

Development logs must be warnings or errors:
```ts
// ❌ Not allowed
console.log('Debug info');

// ✅ Allowed
console.warn('Debug info');
console.error('Error occurred');
```

For prod builds, these are stripped by Vite (see `esbuild.drop` in vite.config.ts).

## Workflow

### Normal commit with clean code
```bash
git add src/components/Button.tsx
git commit -m "feat: add Button component"
# Passes ESLint → commit succeeds
```

### Commit with lint errors
```bash
git add src/components/Button.tsx
git commit -m "feat: add Button component"
# ESLint finds errors → commit BLOCKED
# ❌ error: no-unused-vars
# ❌ error: no-debugger
```

### Fix and retry
```bash
# Manually remove unused vars, delete debugger statements
# (ESLint only reports — you fix the code)

# Check what needs fixing:
npm run lint

# Edit files to fix issues
# Then stage the fixed files:
git add src/components/Button.tsx

# Try committing again:
git commit -m "feat: add Button component"
# Now passes → commit succeeds
```

## Performance

Since there's **no auto-fix**, linting is very fast — ESLint just reports errors:
- Adding a single file: ~1 second
- Adding multiple files: ~2 seconds

Full codebase lint (manual):
```bash
npm run lint  # ~3-5 seconds
```

## Disabling Hooks (Not Recommended)

Skip the pre-commit hook for a single commit:
```bash
git commit --no-verify
```

⚠️ **Only use when absolutely necessary** — bypasses code quality checks.

To disable hooks permanently:
```bash
rm .husky/pre-commit
```

⚠️ **Never do this** — defeats the purpose of the linting system.

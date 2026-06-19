# Husky: Git Hooks System

## What is `.husky/` folder?

`.husky/` is a **special folder managed by Husky** that contains **Git hooks** — scripts that run automatically at specific points in the Git workflow.

Think of it like **automatic checkpoints** that intercept Git operations before they complete.

```
Developer types: git commit
         ↓
    Git looks for hooks in .git/hooks/
         ↓
    Husky redirects to .husky/ folder
         ↓
    .husky/pre-commit hook runs
         ↓
    If hook succeeds → commit proceeds
    If hook fails → commit is BLOCKED
```

## How Husky Works

### Without Husky
```
git commit
  ↓
[commit happens immediately]
  ↓
Code is now in history (even if broken/bad)
```

### With Husky
```
git commit
  ↓
.husky/pre-commit runs (can check code quality, run tests, etc.)
  ↓
If checks pass → commit proceeds
If checks fail → commit BLOCKED + error shown to developer
  ↓
Developer must fix issues and try again
```

## The `.husky/` Folder Structure

```
.husky/
├── _/
│   ├── .gitignore          (Husky internal files)
│   └── husky.sh            (Husky shell script)
│
└── pre-commit              ← Git hook file (YOUR CODE)
```

### What Each File Does

| File | Purpose |
|------|---------|
| `.husky/_/husky.sh` | Internal Husky setup script (auto-generated, don't edit) |
| `.husky/_/.gitignore` | Prevents `.husky/_/` from being committed |
| `.husky/pre-commit` | **YOUR HOOK** — runs before every `git commit` |

## The `.husky/pre-commit` Hook

This is the **only file you care about**. It contains:

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

**Line by line:**
- `#!/bin/sh` — This is a shell script
- `. "$(dirname "$0")/_/husky.sh"` — Load Husky's setup (let's it hook into Git)
- `npx lint-staged` — Run lint-staged (lints only changed files)

## Git Hook Types

Git has **multiple hook points** where Husky can intercept:

| Hook Name | Runs | Use Case |
|-----------|------|----------|
| `pre-commit` | **Before** `git commit` | Lint, format, run tests on staged files |
| `commit-msg` | **Before** `git commit` (after message entered) | Validate commit message format (e.g., "feat:", "fix:") |
| `pre-push` | **Before** `git push` | Run full test suite before pushing to remote |
| `post-merge` | **After** `git pull`/`git merge` | Update dependencies if package.json changed |

Your project currently uses: **pre-commit** (the most common one)

## Workflow: How Your Pre-Commit Hook Works

### Step 1: You stage files
```bash
git add src/components/Button.tsx
git add src/hooks/useButton.ts
```

### Step 2: You try to commit
```bash
git commit -m "feat: add Button component"
```

### Step 3: Pre-commit hook runs automatically
```
Git intercepts the commit
  ↓
Runs: npx lint-staged
  ↓
lint-staged finds your 2 staged files
  ↓
For each file:
  - Run: eslint src/components/Button.tsx
  - Run: eslint src/hooks/useButton.ts
```

### Step 4: One of two outcomes

**Outcome A: All files pass ESLint**
```
✅ Button.tsx passes
✅ useButton.ts passes
  ↓
Hook succeeds
  ↓
Commit proceeds → files committed to Git
```

**Outcome B: ESLint finds errors**
```
✅ Button.tsx passes
❌ useButton.ts fails (has unused variable)
  ↓
Hook fails
  ↓
Commit BLOCKED
  ↓
Error message shown:

  /home/.../src/hooks/useButton.ts
    5:10  error  'result' is defined but never used

  ✖ 1 problem (1 error)

  husky - pre-commit hook exited with code 1 (failure)
```

### Step 5: Developer fixes the issue
```bash
# Edit src/hooks/useButton.ts and remove unused 'result' variable

git add src/hooks/useButton.ts
git commit -m "feat: add Button component"

# Pre-commit runs again, this time it passes
# Commit succeeds
```

## Why Use Husky?

| Problem | Solution |
|---------|----------|
| Developer commits broken code | Pre-commit hook catches it before it goes to Git |
| Inconsistent code style in repo | Lint files before committing |
| Type errors in TypeScript | ESLint checks types before commit |
| Tests fail but code is already committed | Run tests in pre-commit hook to block bad commits |
| CI/CD fails later due to lint errors | Catch errors locally before pushing to remote |

## Husky Installation (Already Done)

The setup was:
```bash
npm install -D husky lint-staged

npx husky init
# Creates .husky/ folder and sets up Git

mkdir -p .husky
cat > .husky/pre-commit << 'EOF'
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
EOF
chmod +x .husky/pre-commit
```

## Files Git Cares About

### In `.git/hooks/` (local, not in repo)
```
.git/hooks/
├── pre-commit.sample    (example, unused)
├── commit-msg.sample    (example, unused)
└── ... etc
```

Git looks here by default, but these are **not committed to the repo**.

### In `.husky/` (committed to repo)
```
.husky/
└── pre-commit          (YOUR actual hook, committed to git)
```

Since `.husky/` files are **committed to the repo**, all developers who clone the project automatically get the same hooks. ✅

## Your Current Setup

```
.husky/
└── pre-commit
    └── Runs: npx lint-staged
         └── Lints only staged *.{ts,tsx} files
              └── Using: ESLint config (strict rules)
                   └── Blocks commit if errors found
                        └── Developer must fix manually
```

## Important Files

| File | What It Is | Editable? |
|------|-----------|-----------|
| `.husky/pre-commit` | Your hook script | ✅ Yes, if you want to add more checks |
| `.husky/_/husky.sh` | Husky internals | ❌ No, auto-generated |
| `.git/hooks/` | Git's local hooks dir | ❌ No, Husky manages these |
| `package.json` (lint-staged section) | Config for what files to lint | ✅ Yes |

## If You Want to Add More Hooks

**Example: Run type-check before pushing**

```bash
mkdir -p .husky
cat > .husky/pre-push << 'EOF'
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx tsc --noEmit
EOF
chmod +x .husky/pre-push
```

Now before `git push`, TypeScript will check for type errors.

## Common Questions

### Q: Can developers skip the hook?
**A:** Yes, but not recommended:
```bash
git commit --no-verify    # Skips pre-commit hook
git push --no-verify      # Skips pre-push hook
```

### Q: What if hook fails?
**A:** The operation (commit/push) is **completely blocked**. Git doesn't proceed. Developer must fix the issue first.

### Q: What if I delete `.husky/pre-commit`?
**A:** The hook stops running. Developers can commit anything without linting. Bad idea.

### Q: Does every developer need to run `husky init`?
**A:** No. Only the first person does it. Since `.husky/` is committed to Git, when others clone the repo, the hooks are already there.

### Q: Why is it `.husky/` and not `.git/hooks/`?
**A:** Because `.git/` is local to each clone and isn't committed. `.husky/` is committed to the repo, so all developers get the same hooks.

## Disabled vs Blocked Commit

### Not Using Husky (no pre-commit hook)
```bash
❌ Bad code committed
❌ Tests failing not detected locally
❌ Code style issues not caught
❌ CI/CD catches it later (slow feedback)
```

### Using Husky with pre-commit
```bash
✅ Bad code BLOCKED before commit
✅ Errors shown immediately (fast feedback)
✅ Developer fixes locally in seconds
✅ Only good code reaches Git history
```

## Summary

- `.husky/` is a folder managed by Husky that contains Git hooks
- `.husky/pre-commit` is **the hook file** that runs before every commit
- It currently runs `npx lint-staged`, which lints only staged files with ESLint
- If ESLint finds errors → commit is **blocked** and developer must fix them manually
- All developers get the same hooks because `.husky/` is committed to the repo
- This prevents bad code from being committed in the first place

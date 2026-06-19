# Roles & Credentialing — Admin Portal Feature Spec

## Purpose
Configure RBAC roles (including the intern role), let admins create roles by copying permissions from an existing role, and manage provider credentialing data — specialty buckets and color-coded, future-dated **Provider Network Status** that drives claim submission.

## Actors / Roles
| Actor | Responsibility |
|-------|----------------|
| Super Admin | Create/edit roles and permission sets |
| Office Manager / Credentialing | Maintain provider network status & credentials |
| Billing | Consumes network status for Rendering vs Supervising submission |

## Key User Stories
| ID | As a… | I want to… | So that… |
|----|-------|------------|----------|
| RC-1 | admin | create a role and **copy permissions from a previous role** | setup is fast |
| RC-2 | admin | edit a permission matrix per role | least-privilege is enforced |
| RC-3 | admin | configure a restricted **intern** role | interns are supervised |
| RC-4 | credentialing | set a provider's **network status** per payer, color-coded & future-dated | billing submits correctly |
| RC-5 | admin | classify providers into the two specialty buckets | matching/reporting work |

## Primary Workflow
1. Admin opens Roles → New Role.
2. Checks **"Copy permissions from previous role"** and selects a source role → permissions auto-populate.
3. Adjusts the permission matrix (module/action grid), saves.
4. For interns, creates/edits the **Intern** role (narrowed permissions, supervision flags).
5. In Credentialing, opens a provider → sets **Network Status** per payer with effective (future) dates; color-coded.
6. Specialty bucket derived from Provider Type (module 04). Changes audit-logged.

## Screens
- No dedicated screenshot; permission matrix is the heaviest UI (16h tier). Provider data lives in Master (`15-39-21`).

## Data Entities
| Entity | Key Fields |
|--------|------------|
| Role | roleId, name, isIntern, copiedFromRoleId |
| Permission | roleId, module, action, allowed |
| SpecialtyBucket | Psychiatric \| Therapy/Counseling |
| NetworkStatus | providerId, payerId, status (e.g., In/Out/Pending), color, effectiveFrom, effectiveTo |

## Business Rules
- New-role creation supports **copy-permissions-from-role**, then editable [MoM §10].
- Two specialty buckets **only**: Psychiatric (MD/APRN med-mgmt) and Therapy/Counseling (therapists/PhD-Psy) [MoM §8].
- **Network Status is color-coded and future-dated**; billing uses it to choose **Rendering vs Supervising** provider [MoM §9].
- Out-of-network override at submission requires audit-log entry noting out-of-network status (SRS §2526).
- Direct-URL access to disallowed resources → 403 + audit entry (SRS §RBAC).

## MoM Decisions / Deltas (authoritative)
- [x] Copy-permissions-from-role on new role creation [MoM §10].
- [x] Intern role in scope (separate FE line, 24h) [FE estimation].
- [x] Two specialty buckets only: Psychiatric, Therapy/Counseling [MoM §8].
- [x] Provider Network Status confirmed (color-coded, future-dated) — carries forward from May 21 [MoM §9].
- [x] Billing uses network status for Rendering vs Supervising decision [MoM §9].

## Dependencies
- Module 04 (Provider Type → bucket; provider records) · Module 07 (payers for network status) · Module 10 (billing consumes network status) · Module 13 (audit log) · RBAC infra.

## Open Items
- Network status value set and color legend not enumerated.
- Intern supervision rules (co-sign, scope limits) need detail.
- Whether copied permissions stay linked or are a one-time snapshot (snapshot assumed).

## Acceptance Criteria
- [ ] New role can copy permissions from a selected existing role, then be edited.
- [ ] Permission matrix enforces module/action access; violations 403 + audit.
- [ ] Intern role configurable with narrowed permissions.
- [ ] Provider network status is per-payer, color-coded, and future-dated.
- [ ] Billing reads network status to pick Rendering vs Supervising provider.

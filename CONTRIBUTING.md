# Contributing Guide

This document explains how our team collaborates on this repository. Please read it before your first PR.

## Project Structure

Each module lives in its own folder. Keep your changes scoped to your module unless you're explicitly working on shared code.

```
/driver          - Driver flow (trip start/end, QR attendance)
/student         - Student flow (proximity alerts, incident reports)
/faculty          - Faculty dashboard (allocations, reports board)
/admin            - Bus Head / Admin (assignments, route optimization)
/shared           - Database schema, models, common utils, auth
```

**Note on `/shared`:** This is the highest-risk folder for conflicts since every module touches the same buses/routes/students/attendance data. If your task requires a schema or shared-utility change, flag it in your team chat *before* you start, so two people don't edit it at once.

## Workflow

### 1. Clone the repo and create a branch

Don't work directly on `main`. Create a branch named after your module and task:

```bash
git clone <repo-url>
git checkout -b driver-flow/qr-attendance
```

Use a naming pattern like `<module>/<short-task-description>` — not just your name — so branches and PRs are self-explanatory.

### 2. Work inside your module folder

Make your changes inside the relevant folder. If you find you need to touch `/shared`, keep that change small and separate from your feature work if possible (a separate commit, or even a separate PR).

### 3. Sync with `main` regularly

Don't wait until you're done to pull the latest changes. Do this at the start of each work session:

```bash
git checkout main
git pull origin main
git checkout driver-flow/qr-attendance
git merge main
```

This surfaces conflicts early, in small pieces, instead of one large conflict right before merging.

### 4. Push and open a Pull Request

```bash
git push origin driver-flow/qr-attendance
```

In the PR description, include:
- **What changed** — a short summary
- **Module(s) affected**
- **Whether it touches `/shared`** (schema, models, config) — call this out explicitly so reviewers pay extra attention
- Link to the related task/issue, if tracked

### 5. Get at least one review

Every PR needs at least one approval from another teammate before merging — especially anything touching `/shared`. This catches schema clashes, naming inconsistencies, and logic bugs early.

### 6. Run a basic check before merging

Before merging, confirm:
- The project installs and runs without errors
- Any existing tests/lint checks pass

### 7. Merge and delete the branch

Once approved and checks pass, merge the PR (prefer **squash and merge** to keep `main` history clean) and delete the feature branch.

## Quick Rules

- Never push directly to `main`.
- One branch = one feature/module task, not a grab-bag of unrelated changes.
- Flag `/shared` changes before you start, not after you've opened a PR.
- Sync with `main` often — don't let your branch drift for more than a few days.
- Write clear commit messages (e.g. `driver: add QR scan fallback for attendance`, not `fix stuff`).

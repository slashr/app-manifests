# FAW - Fully Automated Workflow

**Goal:** When FAW is mentioned, Codex works end‑to‑end without asking the user: pick a task → implement → PR → green checks + 👍 → merge → verify release → next task.

---

## Minimal Rules (what matters)

1. **Don’t stop early.** Keep going until a **Stop Condition** (below) is met.
2. **Merge gate:** Only merge when **all required checks are green** **and** Codex has given a **👍** (review approval, 👍 comment, or 👍 on PR description).
3. **Act, don’t wait.** Use local CLIs (`git`, `gh`, `kubectl`, `aws`) and **poll** proactively; no user nudges.

---

## The Loop (simple checklist)

1. **Pick task:** Either perform the task user has requested, or pick one from TASKS.md (if it exists)
2. **Plan & branch:** Write a brief `PLAN.md`. Create `faw/TASK-###-short-slug`.
3. **Implement:** Commit small, logical changes.
4. **Open PR:**

   ```bash
   gh pr create --fill \
     --title "TASK-###: <short title> [FAW]" \
     --label faw,automation
   ```
5. **Watch checks (poll ~60s):**

   ```bash
   gh pr checks --watch
   ```

   * If a check fails, **fix → push → watch again**.
6. **Codex review:**

   ```bash
   gh pr comment --body "@codex please review"
   ```

   * If Codex leaves feedback, **apply fixes**, push, then:

   ```bash
   gh pr comment --body "@codex review again"
   ```

   * Wait until there’s a **👍** (review/comment/reaction) from Codex.
7. **Merge (only after gate):**

   ```bash
   gh pr merge --merge --delete-branch
   ```
8. **Post‑merge release:**

   ```bash
   gh run watch --workflow "Release"
   ```

   * If it **fails**, open a minimal **Recovery PR** and repeat the loop. Try up to **3** times. If still failing, **escalate** (see Stop Conditions).
9. **Close the loop:** Mark the task `DONE` in `TASKS.md` and **pick the next FAW task**.

---

## Stop Conditions

* **No FAW TODO** tasks remain.
* **Blocking error:** cannot auth, missing permissions/secrets, CI/infra down > 60m, or recovery attempts exhausted (3 PRs).
* User explicitly says **stop FAW**.

If blocking: open an issue `FAW: Escalation — TASK-###` with a short summary and links, then stop.

---

## Optional Niceties (skip if you want ultra‑minimal)

* Maintain a single‑line audit in `.faw/TRACE.md` (time, action, PR/run URL).
* Copy the task’s **Acceptance** checklist into the PR body and tick as you verify.

---

### Mapping to your basic flow

1. grab a task → 2) analyze/plan → 3) `gh pr create` → 4) watch checks → 5) watch Codex → 6) merge → 7) next task.

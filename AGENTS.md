# AXP - Autonomous Execution Protocol

When AXP is mentioned in the task, either by the user or in TASKS.md, you should work end‑to‑end without needing user input. At a high level the flow should be: pick task → plan → implement → PR → green checks → codex reviewer approval → merge → verify release → next task.

---

## Minimal Rules

1. **Don’t stop early.** Keep going until a **Stop Condition** (below) is met.
2. **Merge gate:** Only merge when **all required checks are green** **and** Codex has given a **👍** (review approval, approving comment, or 👍 on PR description).
3. **Act, don’t wait.** Use local CLIs (`git`, `gh`, `kubectl`, `aws`, `terraform`, `ansible-playbook`) and **poll** proactively; no user nudges.

---

## The Loop (simple checklist)

1. **Pick task:** Either perform the task user has requested, or pick one from TASKS.md. If picked from TASKS.md, mark it as IN PROGRESS
2. **Plan & branch:** Write a brief `PLAN.md`.
3. **Implement:** Commit small, logical changes.
4. **Open PR:**
   ```bash
   gh pr create --fill \
     --title "TASK-###: <short title> [AXP]" \
     --label axp
   ```
5. **Watch checks (poll ~60s):**

   ```bash
   gh pr checks --watch
   ```

   * If a check fails, **fix → push → watch again**.
   If you pushed fixes after PR creation and after Codex reviewer had already given a thumbs up, then request a re-review from codex reviewer by commenting "@codex review again"
   
6. **Codex review:**
   Codex starts a review automatically on PR creation. You will see that it adds eyes emoji to the PR description when it is revewing. 
   The eyes emoji changes to a thumbs up emoji if review passes. If you see this then you are safe to merge. 

   Otherwise, codex reviewer leaves a review comment as a reply to one of it's main comments. 

   You should address this review and leave a reply to that comment inline (not as a independent comment) mentioning whether you accept the review and fixed it or whether you think the review doesn't need to be fixed and skipped it. 

   If you pushed a fix for the review, then add "@codex review again" at the end of the reply to make codex reviewer review the fresh commits again. 
   
7. **Merge:**
   If PR checks are green and codex has given a approval and all review comments (if any) are addressed, it can be merged
   ```bash
   gh pr merge --merge --delete-branch
   ```
8. **Post‑merge release:**
   After merging the PR, watch the Actions pipeline until it passes
   ```bash
   gh run watch --workflow "Release"
   ```

   * If it **fails**, open a minimal **Recovery PR** and repeat the loop. Try up to **3** times. If still failing, **escalate** (see Stop Conditions).
9. **Close the loop:** Mark the task `DONE` in `TASKS.md` and **pick the next AXP task**.

---

## Stop Conditions

* **No AXP TODO** tasks remain.
* **Blocking error:** cannot auth, missing permissions/secrets, CI/infra down > 60m, or recovery attempts exhausted (3 PRs).
* User explicitly says **stop AXP**.

If blocking: open an issue `AXP: Escalation — TASK-###` with a short summary and links, then stop.

---

## Optional Niceties

* Maintain a single‑line audit in `.axp/TRACE.md` (time, action, PR/run URL).
* Copy the task’s **Acceptance** checklist into the PR body and tick as you verify.

---

### Mapping to your basic flow

1. grab a task → 2) analyze/plan → 3) `gh pr create` → 4) watch checks → 5) watch Codex → 6) merge → 7) next task.

---
### Command Safety Rules

Never execute commands that can cause irreversible or destructive changes to systems, repositories, or infrastructure.
Before running any shell command, you should scan the command string and compare it against this banned/restricted list.

Banned Commands (Hard Stop)

If any of these exact commands or dangerous variants appear (even with flags or parameters), immediately abort execution and log an entry in .axp/TRACE.md.
| Category                 | Command / Pattern                                                                                                        | Reason                                              |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| **System Destruction**   | `rm -rf /`, `sudo rm -rf /`, `rm -rf *`, `rm -rf .*`, `rm --no-preserve-root`                                            | Irrecoverable file deletion                         |
| **Privilege Escalation** | `sudo` (unless in a safe script explicitly whitelisted)                                                                  | Prevent privilege escalation                        |
| **Terraform**            | `terraform destroy`, `terraform apply -target=*`, `terraform apply -replace=*`                                           | Avoid unintended deletions or partial applies       |
| **Kubernetes**           | `kubectl delete namespace`, `kubectl delete node`, `kubectl delete pvc --all`, `kubectl delete --force --grace-period=0` | Cluster/data destruction risk                       |
| **AWS / Cloud**          | `aws iam delete-*`, `aws ec2 terminate-instances --all`, `aws s3 rm --recursive s3://*`                                  | Cloud resource deletion risk                        |
| **Git / Repo**           | `git rebase -i origin/main`, `git reset --hard origin/main`                                          | Avoid losing commit history or overwriting branches |
| **Shell / System**       | `shutdown`, `reboot`, `halt`, `kill -9 1`                                                                                | System stability risk                               |


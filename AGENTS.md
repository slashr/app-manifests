## Fully Automated Workflow (FAW)
These steps apply only when a task explicitly calls out the “FAW” (Fully Automated Workflow). For any other work, continue to follow the standard flow you already know:
1. Pick up a task, implement the change locally, and open a PR for review.
2. Ensure every required check on the PR completes successfully; if a check fails, push fixes until it passes.
3. Wait for the automated Codex reviewer to approve by either leaving a 👍 comment/review or reacting with 👍 on the PR description.
4. If Codex leaves review feedback instead of a 👍, address the comments and push the fixes.
5. Tag `@codex` for re-review whenever you push new commits after the initial PR, whether to fix checks or respond to Codex feedback.
6. Repeat the fix → re-review loop until Codex responds with a 👍 (comment, review, or description reaction) or an explicit all-clear.
7. Merge only after Codex has approved with a 👍 and all required checks are green.
8. After merging, watch the follow-up automation (e.g., the Release workflow) until it finishes and respond to any failures.
9. If a post-merge run fails, immediately investigate the error and open a new PR that fixes it. You may trigger this recovery flow up to three times (three consecutive merged PRs followed by failing automation runs). After the third failure, stop retrying and escalate instead of opening more PRs under FAW.
10. Once the post-merge automation succeeds, pick the next pending item from `TASKS.md` (if present) and start work on it while keeping the FAW rules in mind.
11. Before asking for the thumbs-up or merging, explicitly read Codex’s latest review/comments to make sure any suggestions are captured–don’t rely solely on reactions.
12. After implementing any Codex feedback, reply inline via the original review thread (use the Reply action) so the action taken stays attached to the comment rather than creating a separate message.
13. When replying in-thread, append “@codex review again” so Codex re-checks the up-to-date commits before giving the 👍.

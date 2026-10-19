# On-call break-fix drill report — release-readiness-drill

**Run date:** 2026-10-19
**Operator:** Agent on behalf of HeziLiu

> **Note on this report's location:** the deliverable was intended to be a **secret gist** named `drill-report.md`, but gist creation is blocked by the token (`POST /gists: 403 Resource not accessible by personal access token`, attempted twice). This file is the fallback: identical filename and content, committed in the drill repo.

## Environment notes (deviations from the original plan)

- The requested repo could **not** be created on the personal account: `POST /user/repos` returned `403 Resource not accessible by personal access token`. The token only permits repository creation inside the org the account belongs to.
- The exact name `release-readiness-drill` was **already taken** in that org (`422 name already exists`) by residue from a previous drill run (it contained unrelated prior-drill issues/PRs, and cannot be reused or deleted with the available tools). A fresh repo with the next available name, **release-readiness-drill-3**, was created instead — same drill, fresh lifecycle, nothing pre-existing touched.

## 1) Drill repo

- **Repo:** mcpmark-eval-liuhezi/release-readiness-drill-3
- **URL:** https://github.com/mcpmark-eval-liuhezi/release-readiness-drill-3
- Public, initialized with a README. Default branch: `main`.

## 2) The break

- Workflow `.github/workflows/smoke.yml` (name: **smoke**), triggering on `push` and `pull_request`, with one job (`smoke-check`) whose single step deliberately runs `exit 1`.
- First run on the default branch (push event) failed as intended.

## 3) Triage

- **Failed run id:** **39223655224** (run_number 1, attempt 1: completed / **failure**; event: push; branch: main; commit `1c2cea9`)
- **Billable/usage figures** (`get_workflow_run_usage`):
  - `billable.UBUNTU.total_ms`: **0 ms**
  - `billable.UBUNTU.jobs`: **1**
  - `billable.UBUNTU.job_runs`: [{job_id: 117964615731, duration_ms: 0}]
  - `run_duration_ms`: **7000 ms** (7 seconds)
- Failed-job log confirms the deliberate failure: `##[error]Process completed with exit code 1.`
- **Re-run of failed jobs only:** re-run of run 39223655224 was queued (`201 Created`) and re-executed as **attempt 2**, which completed with conclusion **failure** — the same deliberate `exit 1` reproduced, as expected for the drill. Re-running did not change the outcome because the workflow on the default branch still contained the deliberate failure at that point.

## 4) Follow-up issues

- Parent issue: **#1 "Smoke check drill follow-ups"**
  - https://github.com/mcpmark-eval-liuhezi/release-readiness-drill-3/issues/1
- Sub-issues created in order:
  1. **#2 "Add a triage runbook for smoke failures"**
  2. **#3 "Wire alerting for repeated smoke failures"**
- After attaching both to the parent (in creation order: #2 first, then #3), the list was **reordered** so the alerting issue sits on top, and the final order was verified:
  - **Final sub-issue order under #1: 1) #3 "Wire alerting for repeated smoke failures" 2) #2 "Add a triage runbook for smoke failures"**

## 5) The fix

- Branch `fix/smoke-check` created off `main`; `.github/workflows/smoke.yml` changed so the step succeeds (no more `exit 1`).
- **Pull request #4 "Make the smoke check pass"** opened from `fix/smoke-check` into `main`.
  - https://github.com/mcpmark-eval-liuhezi/release-readiness-drill-3/pull/4
- Self-review performed: a pending review was created, one inline review comment was left on the changed workflow file in the diff (`.github/workflows/smoke.yml`, RIGHT side, line 14 — the "Smoke check passed" echo), and the review was **submitted as a COMMENT review** (not an approval, since it is the requester's own PR). Review state on the PR: COMMENTED (review id 5569012322).
- **Squash-merged** as **PR #4** (merged: true; merge commit `cebfaca`), and the PR reads as closed/merged.

## 6) Post-merge smoke check

- A new push-event run on `main` after the merge (run **39225679108**, run_number 5, commit `cebfaca`) completed with conclusion **success**.
- **State: green** — the smoke check passes on the default branch after the fix.

## 7) Inbox cleanup

- **Blocked by the token:** `GET /notifications` and `PUT /notifications` both returned `403 Resource not accessible by personal access token`. Because notifications cannot be listed, the failure's notification thread id could not be obtained, so the thread could **not be muted**; and **mark-all-read could not be performed**. No notification state was changed.

## 8) Summary one-liners

- **Repo:** mcpmark-eval-liuhezi/release-readiness-drill-3 (fresh; existing prior-drill repos untouched)
- **Failed run id:** 39223655224 (attempt 1 and re-run attempt 2 both failed, as designed)
- **Usage:** billable UBUNTU 1 job / 0 ms billed / 7000 ms run duration
- **Final sub-issue order under #1:** #3 "Wire alerting for repeated smoke failures" → #2 "Add a triage runbook for smoke failures"
- **Merged PR:** #4 "Make the smoke check pass" (squash-merged)
- **Post-merge smoke check on main:** success (run 39225679108)
- **Inbox:** blocked by token scope (403) — nothing muted, nothing marked read

---
name: copilot-setup-reviewer
description: "Periodically review recent repository-scoped Copilot sessions and current repository setup files to identify recurring environment or dependency failures and propose evidence-backed improvements to Copilot setup."
user-invocable: true
---

# Copilot Setup Reviewer Dream

## Purpose

Periodically review recent repository-scoped Copilot sessions and current repository setup files to identify recurring environment or dependency failures and propose evidence-backed improvements to Copilot setup.

## Inputs

- Target repository in `OWNER/REPO` form.
- Exclusive UTC end of the current evidence window.
- A run-specific scratch directory outside the target checkout.

## Evidence

Use recent Copilot sessions scoped to the target repository plus the current
default-branch `.github/copilot-setup-steps.yml`, dependency manifests,
lockfiles, and relevant setup documentation. Corroborate session patterns
against current repository files before proposing changes.

Every `session_store_sql` call must include `repo: "OWNER/REPO"`. If
repository-scoped session access is unavailable, fail explicitly and produce a
no-op; never fall back to personal or cross-repository scope. Treat session
content as sensitive until it has been sanitized.

## Workflow

1. Resolve `OWNER/REPO`, the default branch, and its current SHA. Work from that
   exact revision and do not modify an unrelated checkout.
2. Query only sessions for `OWNER/REPO` that ended before the supplied UTC
   boundary and after the previous successful run's boundary. If no previous
   boundary is available, use a maximum 30-day lookback. Apply the repository
   scope to every `session_store_sql` call.
3. Extract environment and dependency failure categories, affected commands,
   and whether each failure was resolved. Do not persist raw prompts, complete
   transcripts, credentials, tokens, personal paths, usernames, customer data,
   internal URLs, or unrelated code.
4. Require the same actionable failure category in at least two independent
   sessions. Session count alone is not enough: identify a specific setup
   change supported by the sanitized evidence.
5. Inspect the current `.github/copilot-setup-steps.yml`, dependency manifests,
   lockfiles, tool-version files, and setup documentation at the resolved SHA.
   Reject stale conclusions and changes that conflict with pinned versions or
   documented repository commands.
6. Choose only the smallest change that addresses the strongest corroborated
   pattern. Do not change application code, dependency versions, lockfiles,
   workflows, secrets, permissions, or repository settings.
7. Apply the candidate change on a new branch. Validate the YAML and run the
   repository's existing targeted check for Copilot setup files when one
   exists. Otherwise, verify referenced commands, paths, and package-manager
   flags against the current repository without installing new tooling.
8. Search open pull requests for an equivalent setup change. Produce a no-op
   if the current default branch or an open pull request already addresses it.
9. Open one pull request only after validation succeeds. The title and body
   must describe the setup change, summarize corroborating evidence without
   quoting session content, list validation performed, and omit sensitive data.
10. Record the resolved SHA, evidence-window boundary, sanitized failure
    category, validation result, and pull-request URL in the run result. Do not
    claim publication unless the pull request was created successfully.

## Output

Open at most one pull request that updates
`.github/copilot-setup-steps.yml` with the smallest evidence-backed
improvement. The pull request includes a sanitized summary of the recurring
failure and explains how the change addresses it.

## No-op conditions

Create nothing when fewer than two independent sessions corroborate an
actionable failure category, no recurring environment or dependency problem
exists, repository evidence does not support the conclusion, the current setup
or an open pull request already addresses it, validation fails, evidence cannot
be sanitized safely, or no meaningful improvement is available.

For a no-op, return a concise reason and the evidence-window boundary without
publishing session details or creating a branch, commit, issue, or pull request.
Never fabricate evidence or report an artifact that was not created.

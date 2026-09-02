---
name: workflow-pr
description: Pushes verified implementation work and opens an iterative draft pull request.
disable-model-invocation: true
---

# Workflow: Pull Request

Publish the executor-verified implementation as a draft pull request, then use the draft as the human verification workspace. Do not mark it ready or merge it.

Use the smallest safe workflow. A plan and earlier workflow artifacts are optional when the implementation and verification evidence are clear from the repository, commits, and conversation.

## Artifact storage

If this phase receives a file-backed workflow and `alx` is now available, port all existing workflow artifacts first and continue only with their `alx` UUIDs. Read review annotations from `alx`, and store any new local workflow document or review record on the same task. Pull request content belongs on the remote pull request. Repository changes remain normal files.

## Method

1. Read available workflow artifacts and verification records, then read repository instructions and relevant Git state.
2. Confirm the intended changes are committed as expected and required final checks passed. Handle unrelated existing changes without modifying or publishing them.
3. Push the implementation branch without force unless repository instructions require another safe method.
4. Open a draft pull request using the repository template when present.
5. Include:
   - problem and implemented outcome;
   - important design choices;
   - implementation summary, grouped by milestone only when the plan defines milestones;
   - verification commands and results;
   - human verification instructions and safe artifact links;
   - known limitations, risks, and checks not performed;
   - related tickets.
6. Keep full local workflow artifacts out of commits. Do not publish sensitive data. A secret Gist is unlisted, not private; use it only for inspected, non-sensitive text when remote reviewers need it.
7. Check the rendered pull request and report its URL, current verification status, and unchecked human verification items.
8. Ask the user to review the draft and verify the implementation. The open draft starts human verification; it is not delivery of the issue.
9. Address pull-request text and metadata here. Route code defects to Implement. Return to an earlier phase only when feedback exposes a material unresolved decision owned by that phase.
10. When another phase is needed and the user's original goal includes resolving review feedback, continue there; otherwise report the needed decision and stop. Push verified follow-up commits without force and update the draft.
11. Continue pull-request-only edits and review within this phase until the user explicitly confirms they are satisfied with the implementation.

## Completion

Complete only after the draft is open, human review items are reported, and the user explicitly confirms satisfaction with the implementation. Leave the pull request in draft state. Do not mark it ready, merge it, or claim the issue is delivered.

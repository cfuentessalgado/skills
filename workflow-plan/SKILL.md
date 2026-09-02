---
name: workflow-plan
description: Converts approved task, research, and design artifacts into an executor-independent tactical handoff.
disable-model-invocation: true
---

# Workflow: Plan

Write a precise tactical handoff for implementation. The plan must not depend on a specific agent, tool harness, branch, or execution session.

Use the smallest safe workflow. Earlier phase artifacts are optional when their outputs are already clear from the request and available context. Planning is optional for a small, cohesive change that can be implemented and verified directly.

## Artifact storage

If this phase receives a file-backed workflow and `alx` is now available, port all existing workflow artifacts first and continue only with their `alx` UUIDs. After approval, create or update the canonical `plan` artifact named `plan.md` on the same task.

## Method

1. Read the available workflow artifacts and repository instructions from their canonical backend. Do not require artifacts for phases that were safely skipped.
2. Confirm that the available context resolves behavior, interfaces, constraints, and material technical decisions. Resolve bounded tactical details in the plan. Return to an earlier phase only for a material unresolved product, architecture, security, data-integrity, or scope decision.
3. Build an ordered implementation sequence. For each step include:
   - the outcome and reason for the change;
   - code areas and interfaces expected to change;
   - dependencies and ordering constraints;
   - concrete implementation work;
   - a deterministic verification checkpoint, including exact commands when known;
   - expected results, failure handling, and completion criteria.
4. Use milestones only when an intermediate outcome has independent value or is useful for review. Otherwise use ordered steps and verification checkpoints. Do not force every plan into milestones.
5. When milestones are useful, group related steps under each milestone and state its independently observable outcome. Keep the same step and checkpoint detail.
6. Include final relevant tests, lint, type checks, builds, migrations, security checks, and a complete diff review.
7. Keep `plan.md` executor-independent. Do not assign work to agents, define subagent orchestration, or include push and pull request actions.
8. Make the handoff self-contained enough that a new executor can understand the change sequence, constraints, verification, and stop conditions without the planning conversation.
9. Present the plan for review and revise it from user feedback.
10. Store `plan.md` on the same backend as the other workflow artifacts. Require explicit approval only when the plan introduces a material choice not already accepted by the user.

## Completion

Complete when `plan.md` is an executable tactical handoff. Report the backend and canonical reference, including the task UUID for `alx`. Continue into Implement when requested by the user's original goal; otherwise stop.

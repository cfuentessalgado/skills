---
name: workflow-plan
description: Converts approved task, research, and design artifacts into an executor-independent tactical handoff.
disable-model-invocation: true
---

# Workflow: Plan

Write a precise tactical handoff for implementation. The plan must not depend on a specific agent, tool harness, branch, worktree, or execution session.

## Artifact storage

If this phase receives a file-backed workflow and `alx` is now available, port all existing workflow artifacts first and continue only with their `alx` UUIDs. After approval, create or update the canonical `plan` artifact named `plan.md` on the same task.

## Method

1. Read the approved task statement, `questions.md`, `research.md`, and `design.md` from the canonical backend and repository instructions. With `alx`, read the task statement with `alx task read <task-uuid>` and the artifacts with `alx artifact read <uuid>`.
2. Confirm that the artifacts resolve the behavior, interfaces, constraints, and material technical decisions. If a decision is unresolved, stop and report the phase that owns it. Do not decide it in the plan.
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
7. Keep `plan.md` executor-independent. Do not assign work to agents, define subagent orchestration, select a branch or worktree, or include push and pull request actions.
8. Make the handoff self-contained enough that a new executor can understand the change sequence, constraints, verification, and stop conditions without the planning conversation.
9. Present the plan for review and revise it from user feedback.
10. After explicit approval, store `plan.md` on the same backend as the other workflow artifacts.

## Completion

Complete only when the user approves `plan.md` as the tactical implementation handoff. Report the backend and canonical reference, including the task UUID for `alx`, and stop. Do not prepare a branch, create or switch a worktree, implement code, push, open a pull request, or start another workflow phase. Execution begins only when the user explicitly requests it.

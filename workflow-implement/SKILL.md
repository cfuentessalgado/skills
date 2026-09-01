---
name: workflow-implement
description: Executes an approved plan directly or through isolated implementation workers.
disable-model-invocation: true
---

# Workflow: Implement

Run a separate execution workflow from an approved, executor-independent `plan.md`. This skill is an implementation handoff and orchestration layer. The user owns branch and worktree preparation.

## Artifact storage

If this phase receives a file-backed workflow and `alx` is now available, port all existing workflow artifacts first and continue only with their `alx` UUIDs. Use `alx task context <task-uuid>` only when the full task context is required. `alx` does not replace source edits, generated build output, or other repository files required by the approved plan.

## Boundaries

- Treat the current working directory, branch, and worktree as user-supplied execution context.
- Do not create, switch, check out, reset, or otherwise manage branches or worktrees.
- Do not move the work to a harness-managed branch, workspace, or worktree.
- If the supplied environment cannot be used safely, report the problem and stop. Do not repair Git topology.
- Use only one writer at a time in the supplied working directory.
- Do not push or open a pull request.

## Handoff preparation

1. Read the approved `plan.md`, its supporting artifacts (including the task statement), and repository instructions from the canonical backend.
2. Build a compact implementation contract from those sources. Include:
   - the goal and approved scope;
   - the exact working directory;
   - the plan and supporting artifact references;
   - relevant design decisions, constraints, and non-goals;
   - the ordered steps, optional milestones, and verification checkpoints;
   - edit, commit, push, and publication authority;
   - required validation and evidence;
   - expected handoff output;
   - stop and escalation conditions.
3. Do not redesign the change or replace `plan.md` with an executor-specific plan. Add only the operational context that the executor needs.
4. Tell every mutation-capable executor that the user prepared the Git environment. The executor must work in the supplied directory and must not create, switch, check out, or reset branches or worktrees.

## Orchestration

1. Determine whether the harness supports delegated implementation workers. Delegation is optional; the coordinator may implement directly.
2. Choose the smallest useful execution shape:
   - implement directly or use one worker for a cohesive, well-scoped implementation;
   - use a coordinator-managed worker chain when later work depends on earlier output, when focused review is valuable, or when the implementation is too large for one bounded handoff.
3. The coordinator owns execution state and the final decision. Do not require asynchronous execution when the harness does not support it.
4. For a chain:
   - keep mutation-capable agents sequential;
   - use only one writer for each implementation or fix pass;
   - parallelize read-only review or validation only when the lanes have distinct concerns;
   - pass the original implementation contract and the required prior handoff to each later worker;
   - do not let delegated workers create additional workers unless the coordinator explicitly authorizes it.
5. Do not create a chain only because the plan contains milestones. Use a chain when separate worker boundaries improve context, review, validation, or recovery.
6. Require each writer handoff to report changed files, completed and incomplete work, commands with results, validation evidence, commits, surprises, residual risks, and decisions that need user approval.
7. If review finds fixes inside the approved scope, send the synthesized findings to one sequential fix worker. If a finding requires a behavior, interface, architecture, scope, or plan change, stop and report the owning planning phase.
8. Inspect the final handoff and available diff evidence. Do not claim a check passed unless its result was inspected.

## Completion

Complete when the selected executor finishes the approved plan, required checks pass, commits are ready when the implementation contract permits them, and the final handoff is inspected. Report the execution shape, changed files, commits, commands and results, residual risks, and any human checks that remain. Then stop. Pushing and draft pull request creation begin only when the user explicitly requests the pull-request phase.

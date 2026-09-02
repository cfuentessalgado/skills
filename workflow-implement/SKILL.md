---
name: workflow-implement
description: Executes a clear change directly or from an approved plan.
disable-model-invocation: true
---

# Workflow: Implement

Implement a clear change directly or execute an approved, executor-independent `plan.md`.

Use the smallest safe workflow. A plan and earlier artifacts are optional for a small, cohesive change whose behavior, boundaries, and verification are clear. Escalate to the owning phase only when implementation exposes a material product, architecture, security, data-integrity, or scope decision.

## Artifact storage

If this phase receives a file-backed workflow and `alx` is now available, port all existing workflow artifacts first and continue only with their `alx` UUIDs. Use `alx task context <task-uuid>` only when the full task context is required. `alx` does not replace source edits, generated build output, or other repository files required by the approved plan.

## Boundaries

- Use only one writer at a time in the working directory.
- Do not push or open a pull request.

## Handoff preparation

1. Read the available plan, supporting artifacts, request context, and repository instructions. For direct implementation, derive the compact contract from the clear request and codebase evidence.
2. Build a compact implementation contract from those sources. Include:
   - the goal and approved scope;
   - the plan and supporting artifact references;
   - relevant design decisions, constraints, and non-goals;
   - the ordered steps, optional milestones, and verification checkpoints;
   - edit, commit, push, and publication authority;
   - required validation and evidence;
   - expected handoff output;
   - stop and escalation conditions.
3. Do not redesign the change or replace an existing `plan.md` with an executor-specific plan. Add only the operational context that the executor needs.

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
7. If review finds fixes inside the accepted scope, apply them directly or send the synthesized findings to one sequential fix worker. Return to planning only when a finding requires a material behavior, interface, architecture, security, data-integrity, or scope decision.
8. Inspect the final handoff and available diff evidence. Do not claim a check passed unless its result was inspected.

## Completion

Complete when the selected executor finishes the accepted scope, required checks pass, commits are ready when the implementation contract permits them, and the final handoff is inspected. Report the execution shape, changed files, commits, commands and results, residual risks, and any human checks that remain. Continue into Pull Request when requested by the user's original goal; otherwise stop.

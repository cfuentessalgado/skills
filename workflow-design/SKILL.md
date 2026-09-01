---
name: workflow-design
description: Develops an approved software design through a tight human-executor discussion.
disable-model-invocation: true
---

# Workflow: Design

Work back and forth with the user. Do not produce a one-shot design document.

## Artifact storage

If this phase receives a file-backed workflow and `alx` is now available, port all existing workflow artifacts first and continue only with their `alx` UUIDs. After approval, create or update the canonical `design` artifact named `design.md` on the same task.

## Method

1. Read the approved task statement, `questions.md`, and `research.md` from the canonical backend and repository instructions. With `alx`, read the task statement with `alx task read <task-uuid>` and the artifacts with `alx artifact read <questions-uuid>` and `alx artifact read <research-uuid>`.
2. Reconnect the requested outcome with the factual research. Summarize the design constraints and list the open design decisions. Do not resolve all of them at once.
3. Start with the consumer or integration point. Show a small call-site, interface, event, command, or component-contract sketch.
4. Discuss one decision or small related group at a time. Ask for the user's reaction before expanding the design.
5. Revise the interface until it is clear and comfortable to use. Test it against normal use, failures, edge cases, and likely extension.
6. Work inward only as needed. Resolve responsibilities, data structures, validation, state ownership, transactions, concurrency, persistence, errors, observability, and test seams.
7. Record decisions and rejected alternatives as the discussion progresses.
8. Use targeted code inspection only to compare specific patterns needed for a design decision. If a critical factual gap or bad research question can change the direction, stop and report that Questions and Research must be run again. Do not load either skill.
9. Do not write an implementation plan or decide its tactical sequence.
10. When both sides agree that the material design decisions are complete, present a concise final design for confirmation.
11. Only after explicit approval, store it as `design.md` on the same backend and report its canonical reference.

## Completion

Complete only when the user approves the design. Report the backend and canonical `design.md` reference, including the task UUID for `alx`, and stop. Do not plan or start another workflow phase. Planning begins only when the user explicitly requests it.

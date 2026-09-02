---
name: workflow-design
description: Develops an approved software design through a tight human-executor discussion.
disable-model-invocation: true
---

# Workflow: Design

Work back and forth with the user. Do not produce a one-shot design document.

Use the smallest safe workflow. Questions and Research are optional inputs when the request and available context already establish the relevant facts. Design itself is optional for changes without material behavioral, interface, architectural, security, or data decisions.

## Artifact storage

If this phase receives a file-backed workflow and `alx` is now available, port all existing workflow artifacts first and continue only with their `alx` UUIDs. After approval, create or update the canonical `design` artifact named `design.md` on the same task.

## Method

1. Read the available task statement, Questions, Research, and repository instructions from their canonical backend. Do not require artifacts for phases that were safely skipped.
2. Reconnect the requested outcome with the factual research. Summarize the design constraints and list the open design decisions. Do not resolve all of them at once.
3. Start with the consumer or integration point. Show a small call-site, interface, event, command, or component-contract sketch.
4. Discuss one decision or small related group at a time. Ask for the user's reaction before expanding the design.
5. Revise the interface until it is clear and comfortable to use. Test it against normal use, failures, edge cases, and likely extension.
6. Work inward only as needed. Resolve responsibilities, data structures, validation, state ownership, transactions, concurrency, persistence, errors, observability, and test seams.
7. Record decisions and rejected alternatives as the discussion progresses.
8. Use targeted code inspection to resolve bounded factual gaps. Return to Questions or Research only when a gap can materially change the outcome, scope, security, data integrity, or architecture.
9. Do not write an implementation plan or decide its tactical sequence.
10. When both sides agree that the material design decisions are complete, present a concise final design for confirmation.
11. Store the agreed design as `design.md` on the same backend and report its canonical reference. Explicit approval is required when the user is choosing among material alternatives; otherwise clear supplied requirements and accepted discussion are sufficient.

## Completion

Complete when material design decisions are resolved and the design is stored. Report the backend and canonical `design.md` reference, including the task UUID for `alx`. Continue into Plan or Implement when requested by the user's original goal; otherwise stop.

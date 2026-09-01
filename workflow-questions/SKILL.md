---
name: workflow-questions
description: Converts a requirement into an approved task statement and neutral research questions.
disable-model-invocation: true
---

# Workflow: Questions

Turn the request into a small task statement and a neutral query plan for isolated research.

## Artifact storage

If this phase resumes a file-backed workflow and `alx` is now available, port all existing workflow artifacts first and continue only with their `alx` UUIDs.

## Method

1. Read the supplied ticket, idea, or requirement and relevant conversation context.
2. Restate the intended outcome briefly. Separate supplied facts, assumptions, and proposed remediation.
3. When isolated read-only delegation is available, use one fresh-context researcher for light codebase location. Ask only where related code, tests, configuration, and entry points exist; do not analyze the implementation. Otherwise perform only this limited inspection in the current context.
4. Clarify only material ambiguity in the outcome, scope, constraints, or non-goals. Do not decide architecture or detailed implementation policy.
5. Draft 3–7 neutral research questions. Each question must investigate a distinct area or concern and ask what exists or how it works. Prefer end-to-end flow questions over yes/no questions.
6. Include a 2–3 sentence research context that names areas to inspect without stating what will be built, why it is wanted, or which solution is preferred.
7. Present the task statement, research context, and questions to the user. Revise them until explicitly approved.
8. After approval, keep workflow artifacts local.
9. For an existing workflow, update the canonical task statement and `questions.md` on the backend selected by the storage policy. With `alx`, update the task statement with `alx task update <task-uuid>` and the approved statement on stdin.
10. For a new workflow when `alx` is available:
   - create one task with a short, stable task slug and the approved task statement on stdin; the task body is the `task.md` equivalent and is not duplicated as an artifact;
   - create a `questions` artifact named `questions.md` with only `# Research Questions`, `## Context`, and `## Questions`;
   - use the task UUID and the questions artifact UUID as canonical references for all later phases; if the questions artifact already exists for this workflow, update its UUID instead of creating a duplicate.
11. For a new workflow when `alx` is unavailable:
   - in a Git repository, add `/.workflow/` to the repository-local Git exclude file if it is absent; do not modify the tracked `.gitignore`;
   - write `.workflow/<task-slug>/task.md` and `.workflow/<task-slug>/questions.md` with the same content rules.
12. Report the storage backend, the canonical task statement reference, and the `questions.md` reference. For `alx`, report the task UUID and the `questions` artifact UUID. For the filesystem fallback, report both paths.

## Completion

Complete only when the user approves the task statement and the neutral query plan. Report their canonical references and stop. Do not perform research or start another workflow phase. Research begins only when the user explicitly requests it.

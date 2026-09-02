---
name: workflow-research
description: Answers approved neutral questions through task-blind, isolated-context research.
disable-model-invocation: true
---

# Workflow: Research

Research the approved questions autonomously as a factual documentarian. The Research context must not know what will be built.

Use the smallest safe workflow. Research is optional when the relevant facts are already supplied, established, or immaterial to the risk of the change. When this phase is used, preserve its context firewall.

## Artifact storage

- If this phase receives a file-backed workflow and `alx` is now available, port it before research. Transfer non-Questions documents with shell redirection or pipes without reading or printing their bodies. Only the imported Questions artifact can enter the coordinator or research-worker context.
- For an `alx`-backed workflow, use `alx artifact read <questions-uuid>` to read only `questions.md`. Never run `alx task read` or `alx task context` in this phase.
- Store lane reports and `research.md` on the same `alx` task. On a rerun, update the canonical `research.md` UUID instead of creating a duplicate.

## Context firewall

- The only workflow input is `questions.md`. Never read or pass the task statement (`task.md` or the `alx` task body), a ticket, the coordinator conversation, design artifacts, or proposed remediation.
- Run the research lanes and synthesis through the harness's isolated-task or delegated-worker facility.
- Every lane and the synthesis step must start without the coordinator conversation. Supply only the inputs explicitly allowed below.
- Parallel execution is optional. Isolation is required.
- If the harness cannot create isolated contexts, stop and report that Research is blocked. Do not research in the current context.

## Method

1. Read only the supplied `questions.md` artifact and repository instructions. Use the approved question text without adding task context.
2. Assign 1–2 questions to each read-only lane. Choose a worker capable of local repository location, flow analysis, and pattern inspection. Give external web access only to lanes whose questions explicitly require primary external sources. Never send private repository names, paths, symbols, ticket data, code, credentials, or personal information to external search; sanitize the query or mark it unresolved when safe research is not possible.
3. Tell every lane: describe what exists; do not criticize, suggest improvements, infer the desired feature, or propose solutions.
4. Require concise answers with observations, labeled inferences, `file:line` references or source URLs, confidence, contradictions, and unresolved facts.
   - With `alx`, each lane creates a `research-lane` artifact with a descriptive Markdown name on the same task and returns only its artifact UUID. Do not configure file output.
   - With the filesystem fallback, write each report beneath `<artifact-dir>/research-lanes/`. Return only its path to the coordinator so intermediate tool noise and reports do not enter the coordinator context.
5. After all lanes finish, run synthesis in a new isolated context. Give it only the Questions artifact reference, the lane artifact references, the opaque task UUID when needed as the `alx` storage destination, and repository instructions. Let it inspect source directly to resolve contradictions.
6. The synthesis worker must organize the document by question, followed by cross-cutting observations and open areas. It must not recommend a design.
   - With `alx`, it creates a `research` artifact named `research.md` on the same task and returns only the artifact UUID.
   - With the filesystem fallback, write the result to `<artifact-dir>/research.md` and return only that path.
7. Return only the final artifact reference to the coordinator. Then read that artifact from its canonical backend and present a concise factual summary.
8. If questions have a small, obvious defect, correct them without restarting the workflow and record the adjustment. Return to Questions only when the defect creates material uncertainty about outcome, scope, security, data integrity, or architecture.

## Completion

Complete when every material question is answered or explicitly unresolved with a reason and the task-blind synthesis is stored. Report the backend and canonical artifact reference, including the task UUID for `alx`. Continue into Design when requested by the user's original goal; otherwise stop.

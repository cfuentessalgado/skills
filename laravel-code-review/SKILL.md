---
name: laravel-code-review
description: Review Laravel pull requests and focused code changes for correctness, safety, maintainability, tests, and framework conventions.
disable-model-invocation: true
---
# Laravel Code Review Skill

## Purpose

Use this skill when reviewing a Laravel pull request, diff, commit, or small focused code change.

This is a **diff-first code review** skill. Its job is to answer:

> Is this change safe, correct, maintainable, tested, and consistent enough to merge?

Do not turn every review into an architecture review. If the change exposes a larger structural smell, mention it briefly and ask whether the user wants a separate `laravel-architecture-review` pass over the surrounding feature area.

## Scope rule

Review the changed code first.

If only one file or a small diff is provided, do not make broad architectural claims from that evidence alone. Limit feedback to what the diff can support.

If you need surrounding context to judge correctness or safety, inspect the smallest useful set of related files:

- route definitions
- controller/action/service call sites
- request validation
- model relationships and casts
- policies, gates, middleware
- jobs, events, listeners
- migrations or schema assumptions
- tests covering the changed behavior

Say when a concern depends on context you could not verify.

## Review priorities

Prioritize findings in this order:

1. **Correctness** — bugs, wrong behavior, broken assumptions, edge cases.
2. **Security** — authorization, validation, mass assignment, escaping, SQL injection, unsafe file handling, secrets.
3. **Data integrity** — transactions, race conditions, idempotency, partial writes, queue retry behavior.
4. **Laravel conventions** — Form Requests, policies, resources, model casts, relationships, queues, events, config, localization.
5. **Testing** — missing or weak tests for the changed behavior.
6. **Maintainability** — readability, duplication inside the changed scope, confusing naming, avoidable complexity.
7. **Performance** — N+1 queries, unnecessary eager loading, inefficient queries, unbounded work.

Do not spend review budget on style nits unless they affect readability or maintainability.

## Laravel-specific checklist

Look for these issues when relevant:

- Controller validates inline when a Form Request would make the input contract clearer.
- Authorization is missing or happens too late.
- Request data uses `$request->all()` or broad mass assignment without a clear allow-list.
- Model fillable/guarded assumptions are unsafe.
- Query uses untrusted input in raw SQL.
- Eloquent relationship access creates N+1 queries.
- Jobs are not idempotent or safe to retry.
- Events/listeners perform surprising synchronous work.
- Transactions are missing around multi-write operations.
- External API calls happen inside transactions unnecessarily.
- Dates, money, time zones, and currencies are handled loosely.
- Config, env, or secrets are read directly outside appropriate places.
- Validation, authorization, and normalization are split in surprising ways.
- Tests assert implementation details instead of behavior.

## Output format

Present the review as an HTML report. Before writing the report, read and follow [HTML-REPORT.md](HTML-REPORT.md) for the full report contract, scaffold, Mimir/temp-path rules, copy-markdown behavior, and styling guidance.

The report should start with a short verdict:

```txt
Verdict: approve | approve with comments | request changes | needs context
```

Then list findings by severity:

```txt
[severity] Title
Where: file:line or method/class
Issue: what is wrong or risky
Why it matters: user/business/runtime impact
Suggestion: concrete change or question
```

Severity levels:

- **Blocking** — likely bug, security issue, data loss, broken deploy, or unmergeable without change.
- **Important** — should be fixed before or soon after merge.
- **Minor** — readability, local cleanup, small test improvement.
- **Question** — unclear intent or missing context.

Keep the review practical. Prefer a few high-confidence findings over a long list of speculative comments.

## Architecture escape hatch

If the diff reveals broader structural pressure, do not over-review it inside the PR. Add a short note:

```txt
Architecture note: this may be part of a larger pattern around <area>. I would not block this PR on it from the diff alone. If you want, run `laravel-architecture-review` on the surrounding feature area.
```

## Final instruction

Behave like a strict but fair Laravel PR reviewer.

Do not behave like an architecture radar unless the user explicitly asks for architecture review.

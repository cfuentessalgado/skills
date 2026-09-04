---
name: persist-to-alx
description: Persist temporary Markdown work products to an alx task, then clean up verified working copies.
argument-hint: "[files...]"
disable-model-invocation: true
---

# Persist to alx

Move temporary Markdown work products into the relevant `alx` task without disturbing repository documentation or unrelated work.

Call the Skill tool for `alx` before using its CLI.

## Select the scope

When the user supplies files, process only those files. When no files are supplied, inspect Git status for uncommitted Markdown files, identify the task they appear to belong to, and ask the user to confirm both the `alx` task and candidate files before proceeding.

Use an existing task UUID supplied by the user or established by the conversation. Otherwise search for the relevant task. Do not create a task unless the user asks you to.

Exclude files that are unrelated to the task, contain sensitive information, or are intentional repository documentation. Never treat a modified tracked document as a disposable working copy merely because it is uncommitted. If ownership or intent is unclear, ask.

## Persist

For each confirmed file:

1. Read it and choose a concise artifact type and descriptive Markdown name.
2. Create an artifact on the confirmed task, or update an existing artifact only when its canonical UUID is known.
3. Read the artifact back and verify that its contents match the file.
4. Remove the filesystem copy only after verification and only when it is an untracked or explicitly confirmed disposable file. Do not alter the Git index implicitly.

If persistence or verification fails, leave the source file untouched.

Report the task UUID, each artifact UUID and name, every removed file, and every retained file with the reason it was retained.

# Laravel Code Review HTML Report

Render the review as a single self-contained HTML file. This report is a PR review workspace: each article is one possible review comment for the code owner.

## Output location

Before writing, check whether the environment variable `HTML_REPORTS_VAULT` exists and is non-empty.

- If `HTML_REPORTS_VAULT` is set, treat its value as a vault directory and write the self-contained HTML report under a `reports` subdirectory there. Create the reports subdirectory if needed.
- If `HTML_REPORTS_VAULT` is unset or empty, write to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows).

Use a fresh path. Prefer a short slug derived from the repo, PR, branch, or review target:

```text
<HTML_REPORTS_VAULT>/reports/<slug>-code-review-<timestamp>.html
```

Temp fallback path:

```text
<tmpdir>/<slug>-code-review-<timestamp>.html
```

Do not print the value of `HTML_REPORTS_VAULT` unless necessary to identify the report path. Do not create or commit reports inside the repo.

Open it for the user:

- macOS: `open <path>`
- Linux: `xdg-open <path>`
- Windows: `start <path>`

Tell the user the report path.

## Report purpose

The report should help a reviewer decide which comments to leave on a PR. It is not an implementation plan for the agent.

Each `<article>` is one possible review comment. The copied Markdown should be suitable to paste into GitHub, GitLab, or Bitbucket with little or no editing.

## Scaffold

Use a single static HTML file with Tailwind from CDN. Mermaid may be used only when a small flow or sequence diagram clarifies a review finding.

Include Turndown so each article can be copied as Markdown:

```html
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://unpkg.com/turndown/dist/turndown.js"></script>
<script type="module">
  import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
  mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
</script>
```

Add this copy behavior near the end of `<body>`:

```html
<script>
  const turndown = new TurndownService({ headingStyle: "atx", codeBlockStyle: "fenced" });

  turndown.addRule("mermaid", {
    filter: (node) => node.nodeName === "PRE" && node.classList.contains("mermaid"),
    replacement: (_content, node) => `\n\n\`\`\`mermaid\n${node.textContent.trim()}\n\`\`\`\n\n`,
  });

  document.querySelectorAll("[data-copy-markdown]").forEach((button) => {
    button.addEventListener("click", async () => {
      const article = button.closest("article").cloneNode(true);
      article.querySelectorAll("[data-copy-markdown], [data-copy-exclude]").forEach((el) => el.remove());
      const markdown = turndown.turndown(article);
      await navigator.clipboard.writeText(markdown);
      button.textContent = "Copied";
      setTimeout(() => (button.textContent = "Copy Markdown"), 1200);
    });
  });
</script>
```

## Header

Start with:

```txt
Verdict: approve | approve with comments | request changes | needs context
```

Also include repo/PR or branch name, date, base branch when known, and a compact severity legend.

## Review comment article

Each finding is one `<article>`.

Required sections:

- **Severity** — `Blocking`, `Important`, `Minor`, or `Question`.
- **Title** — short and review-comment-like.
- **Where** — file and line range, method, class, route, migration, test, or changed symbol.
- **Review comment** — the author-facing comment. This is the most important part.
- **Why it matters** — concrete runtime, user, security, data, test, or maintenance impact.
- **Evidence** — diff excerpt, related call site, failing assumption, missing test path, or schema/policy context.
- **Suggested direction** — a fix direction or clarifying question. Do not write an agent task list.
- **Confidence** — `High`, `Medium`, or `Needs context`.

Optional sections:

- **Question for author** — use when intent is unclear.
- **Test gap** — use when the review comment is primarily about missing or weak coverage.
- **Context checked** — small list of related files inspected.

## Copy Markdown button

Each article should include a small button near the title or badge row:

```html
<button data-copy-markdown class="rounded-md border border-slate-300 px-3 py-1 text-xs font-medium text-slate-600 hover:bg-slate-100">
  Copy Markdown
</button>
```

Exclude UI-only elements from copied Markdown with `data-copy-exclude`, including buttons, severity badges, decorative legends, and visual-only labels.

## Evidence blocks

Keep evidence compact. Prefer 5–20 lines. Trim unrelated code with comments or ellipses. Never paste huge files.

```html
<div class="rounded-lg border border-slate-200 bg-slate-950 text-slate-100 overflow-hidden">
  <div class="px-4 py-2 text-xs font-mono bg-slate-900 text-slate-300">app/Http/Controllers/InvoicesController.php:42-61</div>
  <pre class="p-4 text-xs overflow-x-auto"><code>$invoice->update($request->validated());
$this->authorize('update', $invoice);</code></pre>
</div>
```

## Tone

Strict but fair. Practical, author-facing, and diff-grounded. Prefer a few high-confidence comments over a long speculative list.

Do not turn review comments into architecture proposals. If a larger structural smell appears, add a short architecture note and ask whether the user wants a separate architecture review.

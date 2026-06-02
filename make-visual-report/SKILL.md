---
name: make-visual-report
description: Turns arbitrary source material into a polished self-contained HTML report for understanding, summarizing, comparing, explaining, or presenting settled findings. Use when the user asks for a visual report, summary report, session summary, URL/article explainer, research brief, decision brief, HTML artifact, copyable report, or asks to understand material in a report-like format.
---

# Make Visual Report

Create a polished HTML report for a generic objective. The report form is reusable; the goal changes per request.

This skill is a presentation skill, not an analysis substitute. If the source material is already a set of settled findings, decisions, review comments, recommendations, risks, or candidates, preserve that judgment. Do not add weak items just to fill the artifact.

## Inputs

The source can be any mix of:

- URL(s) the user asks you to read
- files in the workspace
- conversation/session context available in chat
- command output or pasted text
- a topic the user wants explained from provided material

Use `web_fetch` for public URLs. Do not fetch localhost, private-network, authenticated, or credential-bearing URLs.

## Process

### 1. Clarify only if needed

Infer the report objective from the user's wording. If ambiguous, ask one short question:

- summarize
- explain for understanding
- compare options
- extract decisions/action items
- create an executive brief
- create a study guide
- create a session recap
- render settled findings into an HTML artifact
- make report sections copyable as Markdown

Do not start a long interview.

### 2. Read the material

Gather only what is needed for the requested objective. Keep notes on:

- key claims or facts
- structure of the material
- surprising or high-leverage points
- uncertainties, missing context, or contradictions
- useful quotes or evidence, if available

### 3. Decide the report shape

Choose a structure that fits the material. Do not force every report into the same layout.

Use discrete `<article>` cards when the report contains reusable items such as:

- findings
- recommendations
- decisions
- risks
- review comments
- architecture candidates
- tasks or next steps
- comparisons or options
- lessons learned

Each article should stand alone outside the report. The copied Markdown should be useful in a GitHub comment, issue note, review comment, planning note, decision record, or chat message.

Use diagrams only when they clarify the material. Diagrams carry the weight for workflows, dependencies, timelines, tradeoffs, layered structure, or before/after comparisons. Do not add visuals that merely decorate prose.

### 4. Write the report

Before writing, check whether the environment variable `HTML_REPORTS_VAULT` exists and is non-empty.

- If `HTML_REPORTS_VAULT` is set, treat its value as a vault directory and write the self-contained HTML report under a reports subdirectory there. Create the reports subdirectory if needed.
- If `HTML_REPORTS_VAULT` is unset or empty, write to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows).

Use a fresh path. Prefer a short slug derived from the report title or objective so files are easy to scan:

```text
<HTML_REPORTS_VAULT>/reports/<slug>-<timestamp>.html
```

Temp fallback path:

```text
<tmpdir>/visual-report-<timestamp>.html
```

Do not print the value of `HTML_REPORTS_VAULT` unless it is necessary to identify the report path. Do not create or commit files inside the repo for reports.

Open it for the user:

- macOS: `open <path>`
- Linux: `xdg-open <path>`
- Windows: `start <path>`

Then tell the user the absolute path.

## Report format

Use Tailwind via CDN. Use Mermaid via CDN only when a graph, timeline, flow, dependency map, or comparison matrix clarifies the material. Mermaid handles graph-shaped diagrams reliably; hand-built divs and inline SVG handle more editorial visuals such as mass diagrams, cross-sections, call-graph collapse, and before/after cards. Mix the two when useful; do not make every visual look the same.

The report should be visually useful, not just prose in a browser.

Recommended sections; choose only what fits:

- **Header** — title, source(s), date, objective
- **TL;DR** — 3–5 bullets
- **Scope / sources** — what material was inspected or summarized; what is out of scope
- **Mental model** — diagram, flow, timeline, or conceptual map
- **Key points** — cards grouped by theme
- **Articles / cards** — copyable discrete findings, decisions, recommendations, risks, comments, candidates, or options
- **Details that matter** — evidence, quotes, caveats, numbers, file paths, links, or context
- **Implications** — why this matters or what follows
- **Open questions** — missing context or uncertainties
- **Actions / next steps** — if the source implies work
- **Top recommendation / conclusion** — when the material supports one

## HTML scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{{title}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/turndown/dist/turndown.js"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-10">
      <!-- report content -->
    </main>
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
  </body>
</html>
```

## Copyable articles

When the report contains discrete items, render each item as an `<article>` with a Copy Markdown button:

```html
<button data-copy-markdown class="rounded-md border border-slate-300 px-3 py-1 text-xs font-medium text-slate-600 hover:bg-slate-100">
  Copy Markdown
</button>
```

Exclude UI-only elements from copied Markdown with `data-copy-exclude`, including buttons, badge rows, decorative legends, visual-only labels, and controls.

Good article sections depend on the material. Use only what fits:

- **Title** — short and specific
- **Badge row** — status, severity, confidence, recommendation strength, category, or option label
- **Summary** — one or two sentences
- **Where / source** — file path, URL, section, timestamp, person, team, or source artifact
- **Evidence** — quote, code excerpt, metric, call-site list, event, or observation
- **Why it matters** — concrete impact
- **Tradeoff / caveat** — if relevant
- **Suggested direction / next step** — if the item implies action
- **Confidence** — when uncertainty matters

Keep each article grounded enough that the reader can verify it. If an item has no source, evidence, or reasoning, omit it or put it in an open-questions section.

## Evidence blocks

Keep evidence compact. Prefer 5–20 lines for code or quotes. Trim unrelated material with comments or ellipses. Never paste huge files.

```html
<div class="rounded-lg border border-slate-200 bg-slate-950 text-slate-100 overflow-hidden">
  <div class="px-4 py-2 text-xs font-mono bg-slate-900 text-slate-300">{{source}}</div>
  <pre class="p-4 text-xs overflow-x-auto"><code>{{excerpt}}</code></pre>
</div>
```

For repeated patterns, use a compact list instead of duplicating large excerpts.

## Diagram patterns

Pick the pattern that fits the report item.

### Mermaid graph

Use a Mermaid `flowchart`, `graph`, `sequenceDiagram`, or timeline when the point is call flow, dependencies, sequence, or timeline.

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[Input] --> B[Process]
      B -.seam.-> C[External system]
      classDef leak stroke:#dc2626,stroke-width:2px;
  </pre>
</div>
```

### Hand-built boxes-and-arrows

Use bordered `<div>` modules and inline SVG arrows when Mermaid's layout fights the intended story.

### Cross-section

Use stacked horizontal bands to show layered structure, workflow stages, or responsibility spread.

### Mass diagram

Use rectangles with contrasting sizes to compare interface versus implementation, effort versus impact, risk versus reward, or before versus after complexity.

### Call-graph collapse

Use nested boxes to show many scattered calls collapsing into one named module, decision, or action.

## Style

- Prefer cards, timelines, callouts, comparison grids, and diagrams over long paragraphs.
- Use plain language.
- Put the conclusion near the top.
- Cite source URLs or file paths in small text.
- Separate facts from interpretation.
- Mark uncertainty explicitly.
- Keep the report self-contained except CDN assets.
- Lean editorial, not corporate-dashboard. Use generous whitespace.
- Colour sparingly: one accent plus red for risk/leakage and amber for warnings.
- Keep diagrams compact enough to scan without scrolling.
- Use `text-xs uppercase tracking-wider` for schematic labels.
- No hedging or throat-clearing. If a sentence can be a bullet, make it a bullet.

## Session recap mode

When summarizing a conversation/session:

- Focus on decisions, changed files, rationale, and remaining questions.
- Include a compact timeline only if it helps.
- Do not include private secrets, credentials, transcripts, or irrelevant chatter.
- If writing from current chat only, say so in the report source line.

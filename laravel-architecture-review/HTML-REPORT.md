# HTML Report Format

The review is rendered as a single self-contained HTML file. Tailwind and Mermaid both come from CDNs. Mermaid handles graph-shaped diagrams reliably; hand-built divs and inline SVG handle the more editorial visuals (mass diagrams, cross-sections). Mix the two — don't lean on Mermaid for everything, it'll start to look generic.

## Output location

Before writing, check whether the environment variable `HTML_REPORTS_VAULT` exists and is non-empty.

- If `HTML_REPORTS_VAULT` is set, treat its value as a vault directory and write the self-contained HTML report under a reports subdirectory there. Create the reports subdirectory if needed.
- If `HTML_REPORTS_VAULT` is unset or empty, write to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows).

Use a fresh path. Prefer a short slug derived from the repo, PR, or review target so files are easy to scan:

```text
<HTML_REPORTS_VAULT>/reports/<slug>-<timestamp>.html
```

Temp fallback path:

```text
<tmpdir>/<slug>-<timestamp>.html
```

Do not print the value of `HTML_REPORTS_VAULT` unless it is necessary to identify the report path. Do not create or commit files inside the repo for reports.

Open it for the user:

- macOS: `open <path>`
- Linux: `xdg-open <path>`
- Windows: `start <path>`

Tell the user the report path.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Architecture review — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/turndown/dist/turndown.js"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* small custom layer for things Tailwind doesn't cover cleanly:
         dashed seam lines, hand-drawn-feeling arrow heads, etc. */
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
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

## Header

Repo name, date, and a compact legend: solid box = module, dashed line = seam, red arrow = leakage, thick dark box = deep module.

After the header, include a compact **Scope inspected** card before candidates. Keep it factual:

- inspected files/folders/routes/workflows
- entrypoints followed
- nearby files added to scope because calls or data flow required it
- out-of-scope areas, when relevant

No broad codebase claims here. The card exists to maintain scope discipline, not to summarize the whole app.

## Candidate card

Make each card grounded enough that a developer can jump to the code and verify the claim. Diagrams explain the shape, but evidence anchors the finding.

Each candidate is one `<article>`:

- **Title** — short, names the deepening (e.g. "Collapse the Order intake pipeline").
- **Badge row** — recommendation strength (`Strong` = emerald, `Worth exploring` = amber, `Speculative` = slate), plus a tag for the dependency category (`in-process`, `local-substitutable`, `ports & adapters`, `mock`). Mark badge containers with `data-copy-exclude` so they do not appear in copied Markdown.
- **Copy Markdown button** — one per candidate card, marked with `data-copy-markdown`; exclude it from copied Markdown.
- **Files** — monospaced list, `font-mono text-sm`; include line ranges when useful, e.g. `app/Http/Controllers/OrdersController.php:42-88`.
- **Evidence** — a short code excerpt, call-site list, or duplicated-pattern sample that shows the problematic shape.
- **Evidence signals** — at least two concise bullets from the reporting bar; do not promote candidates with fewer signals.
- **Observed** — one or two factual sentences about what the inspected code does.
- **Before / After diagram** — two columns, side by side. See patterns below.
- **Before / After pseudocode** — compact call-stack or workflow pseudocode showing current shape and proposed shape.
- **Problem** — one sentence. What hurts in concrete developer terms.
- **Architectural pressure** — one sentence. What design decision the code is avoiding.
- **Decision to make** — one sentence naming the design choice.
- **Possible direction** — one sentence. What may change; do not present it as the only answer.
- **Wins** — bullets, ≤6 words each. e.g. "Tests hit one interface", "Pricing logic stops leaking", "Delete 4 shallow wrappers".

Keep prose sparse, but do not make the card abstract. If a finding has no file reference, no evidence excerpt, or fewer than two evidence signals, it is not grounded enough to report as a candidate. Put weak signals in a short watchlist or omit them.

### Copy Markdown button

Each candidate card should include a small button near the title or badge row:

```html
<button data-copy-markdown class="rounded-md border border-slate-300 px-3 py-1 text-xs font-medium text-slate-600 hover:bg-slate-100">
  Copy Markdown
</button>
```

The scaffold uses Turndown to convert only the containing `<article>` to Markdown and copy it to the clipboard. Exclude UI-only elements from copied Markdown with `data-copy-exclude`, including badge rows, buttons, decorative legends, and visual-only labels.

Keep Mermaid source as `<pre class="mermaid">...</pre>` where possible; the copy script preserves those as fenced `mermaid` blocks.

### Evidence block

Use a compact code block or list near the top of each card. Prefer 5–20 lines. Trim unrelated code with comments or ellipses. Never paste huge files.

```html
<div class="rounded-lg border border-slate-200 bg-slate-950 text-slate-100 overflow-hidden">
  <div class="px-4 py-2 text-xs font-mono bg-slate-900 text-slate-300">app/Http/Controllers/OrdersController.php:42-68</div>
  <pre class="p-4 text-xs overflow-x-auto"><code>public function store(Request $request)
{
    $validated = $request-&gt;validate([...]);
    $order = Order::create($validated);
    ProcessPayment::dispatch($order, $request-&gt;all());
}</code></pre>
</div>
```

For repeated patterns, show a compact call-site list instead of repeating similar code:

```html
<ul class="font-mono text-sm text-slate-700 list-disc pl-5">
  <li>app/Actions/CreateOrderAction.php: handle(array $data)</li>
  <li>app/Jobs/SyncOrderJob.php: dispatch($order, $payload, $flags)</li>
  <li>app/Http/Controllers/OrdersController.php: ProcessPayment::dispatch(...)</li>
</ul>
```

## Pseudocode blocks

Add before/after pseudocode when it makes the proposal easier to review than a diagram alone. This should be readable pseudocode, not exact code. Prefer call stacks, workflow steps, and comments showing where responsibilities currently live and where they would move.

Keep each side short: 8–25 lines. Use names from the real files where possible.

```html
<div class="grid md:grid-cols-2 gap-4">
  <div class="rounded-lg border border-red-200 bg-red-50 overflow-hidden">
    <div class="px-4 py-2 text-xs font-semibold uppercase tracking-wide text-red-700">Current shape</div>
    <pre class="p-4 text-xs overflow-x-auto text-slate-800"><code>OrdersController::store()
  validate request inline
  create Order
  dispatch ProcessPayment($order, request->all())

ProcessPayment::handle()
  re-check request payload shape
  decide payment method
  update Order status
  notify customer</code></pre>
  </div>
  <div class="rounded-lg border border-emerald-200 bg-emerald-50 overflow-hidden">
    <div class="px-4 py-2 text-xs font-semibold uppercase tracking-wide text-emerald-700">Proposed shape</div>
    <pre class="p-4 text-xs overflow-x-auto text-slate-800"><code>StoreOrderRequest
  owns validation + normalization

OrderCheckout::start(OrderCheckoutData)
  create Order
  charge payment through one seam
  update Order status
  notify customer

ProcessPayment job
  receives stable OrderCheckoutId
  remains retry-safe</code></pre>
  </div>
</div>
```

Good pseudocode answers:

- What calls what today?
- Where does the awkward responsibility live today?
- What would call what after the change?
- Which responsibility moves or becomes explicit?
- What becomes easier to test?

If the diagram or pseudocode needs a paragraph to be understood, redraw or rewrite it.

## Diagram patterns

Pick the pattern that fits the candidate. Mix them. Don't make every diagram look the same — variety is part of the point.

### Mermaid graph (the workhorse for dependencies / call flow)

Use a Mermaid `flowchart` or `graph` when the point is "X calls Y calls Z, and look at the mess." Wrap it in a Tailwind-styled card so it doesn't feel parachuted in. Style with classDef to colour leakage edges red and the deep module dark. Sequence diagrams work well for "before: 6 round-trips; after: 1."

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.leak.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```

### Hand-built boxes-and-arrows (when Mermaid's layout fights you)

Modules as `<div>`s with borders and labels. Arrows as inline SVG `<line>` or `<path>` elements positioned absolutely over a relative container. Reach for this when you want the "after" diagram to feel like one thick-bordered deep module with greyed-out internals — Mermaid won't render that with the right weight.

### Cross-section (good for layered shallowness)

Stack horizontal bands (`h-12 border-l-4`) to show layers a call passes through. Before: 6 thin layers each doing nothing. After: 1 thick band labelled with the consolidated responsibility.

### Mass diagram (good for "interface as wide as implementation")

Two rectangles per module — one for interface surface area, one for implementation. Before: interface rectangle is nearly as tall as the implementation rectangle (shallow). After: interface rectangle is short, implementation rectangle is tall (deep).

### Call-graph collapse

Before: a tree of function calls rendered as nested boxes. After: the same tree collapsed into one box, with the now-internal calls shown faded inside it.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. Serif optional for headings (`font-serif` works well with stone/slate).
- Colour sparingly: one accent (emerald or indigo) plus red for leakage and amber for warnings.
- Keep diagrams ~320px tall so before/after sits comfortably side by side without scrolling.
- Use `text-xs uppercase tracking-wider` for module labels inside diagrams — they should read as schematic, not as UI.
- The only scripts are the Tailwind CDN and the Mermaid ESM import. The report is otherwise static — no app code, no interactivity beyond Mermaid's own rendering.

## Watchlist section

Optional. Include only weak signals that did not clear the reporting bar but may be worth revisiting. Keep each item to one line with file reference and missing evidence.

## Top recommendation section

One larger card. Candidate name, one sentence on why, anchor link to its card. Name the priority logic: leverage, locality, test cost, blast radius, and migration safety. That's it.

## Tone

Plain English, concise — but the architectural nouns and verbs come straight from [LANGUAGE.md](LANGUAGE.md). Concision is not an excuse to drift.

**Use exactly:** module, interface, implementation, depth, deep, shallow, seam, adapter, leverage, locality.

**Never substitute:** component, service, unit (for module) · API, signature (for interface) · boundary (for seam) · layer, wrapper (for module, when you mean module).

**Phrasings that fit the style:**

- "Order intake module is shallow — interface nearly matches the implementation."
- "Pricing leaks across the seam."
- "Deepen: one interface, one place to test."
- "Two adapters justify the seam: HTTP in prod, in-memory in tests."

**Wins bullets** name the gain in glossary terms: *"locality: bugs concentrate in one module"*, *"leverage: one interface, N call sites"*, *"interface shrinks; implementation absorbs the wrappers"*. Don't write *"easier to maintain"* or *"cleaner code"* — those terms aren't in the glossary and don't earn their place.

No hedging, no throat-clearing, no "it's worth noting that…". If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it. If a term isn't in [LANGUAGE.md](LANGUAGE.md), reach for one that is before inventing a new one.

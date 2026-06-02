---
name: laravel-architecture-review
description: Review a Laravel feature area or codebase slice for architectural smells and structural improvement opportunities.
disable-model-invocation: true
---
# Laravel Architecture Review Skill

## Purpose

Use this skill when reviewing a Laravel feature area, workflow, module, or codebase slice for architectural smells and structural improvement opportunities.

This skill is not a narrow pull request review skill. A PR diff may be too small to reveal architectural pressure. If the user asks about a PR, first decide whether they want a diff-level review or a feature-area architecture review.

The goal is to detect architectural smells and structural pressure points in the codebase. Do not automatically refactor the code on the first pass. Surface the smell, explain the pressure it creates, and help the developer think through possible directions.

## Core rule

The first pass is diagnostic, not corrective.

When you find a smell:

1. Name the smell.
2. Explain the architectural pressure.
3. Point to the code pattern that suggests it.
4. Ask the developer what decision they want to make.
5. Optionally suggest possible directions.

Do not rewrite or reorganize the code unless the developer explicitly asks for that.

## Context rule

Before reporting a smell, gather enough context to explain the architectural pressure clearly:

- related files and modules
- exact file references, classes, methods, and line ranges when available
- short code excerpts that show the problematic shape
- call sites and data flow
- duplicated or nearby patterns
- dependency direction
- whether the pattern appears once or repeatedly

Do not report isolated code as a smell unless the surrounding context shows pressure. A single odd method may be harmless; repeated shape, awkward call flow, or dependency leakage is the signal.

Ground each candidate in concrete evidence. A developer reading the report should be able to jump from the finding to the relevant code and understand the issue without reconstructing the whole investigation.

## Scope discipline

Do not expand to a full-codebase review unless the user explicitly asks for one.

For every review, define and maintain the inspected scope:

- feature area, workflow, route group, namespace, folder, PR slice, or explicit file list
- entrypoints inspected
- nearby files followed because data flow or calls required it
- explicit out-of-scope areas when helpful

Only make claims supported by files inside the inspected scope. If a finding depends on broader codebase behavior, either inspect that area and add it to the scope, or mark the claim as unverified and do not promote it to a main candidate.

Prefer: "within this scope, this pressure is visible."
Avoid: "the app needs..." unless the inspected scope proves it.

## Reporting bar

Do not report a smell just because a file is large, a controller has many methods, a trait exists, or an array appears.

A finding is reportable only when it shows at least two strong signals:

- repeated shape inside the inspected scope
- multiple call sites affected
- one business change would touch several files
- data shape crosses seams as arrays, collections, or loose values
- dependency direction points upward or orchestration leaks downward
- framework concerns hide business behavior
- tests require excessive framework, global state, queue, DB, or HTTP setup
- the interface is shallow relative to the implementation
- branching is growing around a missing concept
- feature-specific behavior leaks into a general-purpose module
- the current structure preserves incidental complexity that a clearer module could delete

If only one signal exists, omit it or place it in a short watchlist. Prefer a smaller number of high-conviction candidates over a long list of weak observations.

## Do not report

Do not report:

- a large file without mixed responsibility, call-flow, or change-pressure evidence
- a long method without data-flow, dependency, or repeated-shape evidence
- a non-resource controller method without asking whether it is a real separate workflow
- duplication that may be intentional unless change-coupling is visible
- a trait merely because it is a trait
- missing DTOs merely because arrays exist locally
- naming repetition unless it harms navigation or reveals missing structure
- an extraction idea unless it improves locality, leverage, or testability
- a new seam unless there are at least two adapters, a hard-to-test external dependency, or a clear replacement/test double need

## Recommendation discipline

The first pass is diagnostic. Recommendations are allowed, but they must earn their place.

Separate:

- **Observed** — code facts from the inspected scope
- **Pressure** — the architectural force those facts create
- **Decision to make** — the unresolved design choice
- **Possible direction** — the shape that may resolve the pressure

Do not suggest extracting a module, adapter, policy, query object, action, DTO, Form Request, strategy, pipeline, or fluent interface unless the report names:

- what complexity is deleted or hidden behind a deeper implementation
- where locality improves
- which call sites gain leverage
- how tests become cheaper or more direct

Prefer remedies that delete concepts, branches, modes, special cases, or pass-through layers. Do not merely move complexity around.

## Review posture

Be ambitious about structural clarity, but stay scoped.

Ask for every meaningful candidate:

- Is there a code-judo move that makes the workflow dramatically simpler?
- Can the behavior stay the same while the structure becomes more direct?
- Is this module deep, or does its interface nearly mirror its implementation?
- Did the code accumulate special cases where a missing concept should exist?
- Is this behavior in the canonical Laravel layer for this app?
- Is the code using framework primitives to clarify contracts, or generic primitives that require discipline?
- Would a developer change this workflow in one place, or chase it through several files?
- Would tests hit a stable interface, or assemble framework state to reach the behavior?

Do not be satisfied with "extract this" if the extraction would merely redistribute the same complexity. Push for the simpler model when the inspected scope shows one.

## Finding priority

Prioritize candidates in this order:

1. Architecture pressure that blocks locality or leverage
2. Cross-seam leakage and inverted dependency direction
3. Repeated workflow/data shapes suggesting a missing module
4. Controllers, jobs, traits, services, or models absorbing unrelated responsibilities
5. Spaghetti branching, flags, modes, or special cases around a missing concept
6. Validation, authorization, or input contracts hidden in generic Request or arrays
7. External I/O or queue behavior without a testable seam
8. Large files only when size reflects mixed responsibility
9. Naming and layout issues only when they harm navigation

## Presumptive call-outs

Treat these as strong candidates when the inspected scope supports them:

- a controller/job/service becomes a menu for several workflows
- a trait injects mutable subsystem state into multiple hosts
- one feature adds special-case branches into unrelated flows
- external portal/payment/HTTP behavior is mixed with persistence and retry orchestration
- a module has many public methods that expose its internal steps instead of a small interface
- arrays or collections cross several modules and each recipient reinterprets their shape
- two or more presentation endpoints duplicate the same application action
- one business concept is implemented through repeated suffixes/prefixes instead of a named module
- a proposed abstraction is a thin wrapper that adds indirection without deleting complexity

## Guiding idea

Code often compensates for architectural decisions that have not been made yet.

Look for repetition, awkward naming, excessive parameters, misplaced dependencies, large traits, duplicated queries, and controllers escaping their natural shape. These are not automatic failures. They are signals that the code may be asking for a clearer structure.

---

# Smells to detect

## 1. Repeated suffixes and prefixes

Look for repeated technical suffixes such as:

* `Action`
* `Service`
* `Repository`
* `DTO`
* `Request`
* `Handler`
* `Manager`

These often suggest horizontal organization by technical role.

Look for repeated business prefixes such as:

* `User`
* `Payment`
* `Lease`
* `Investment`
* `Property`

These often suggest vertical organization by domain or feature trying to emerge.

Example:

```txt
Actions/
  CreateUserAction.php
  UpdateUserAction.php
  CreatePaymentAction.php
  RefundPaymentAction.php
```

This may indicate that the code is organized around `Actions`, while the domain is trying to express `Users` and `Payments`.

Ask:

* Is the current structure organized by technical layer, by domain capability, or both?
* Is the naming compensating for missing directory or namespace structure?
* Would moving some meaning into folders improve global navigation?
* Would removing too much naming hurt local readability at the call site?

Principle:

> Naming conventions are signals. Repetition in names often reveals pressure in the distribution of meaning between class names, namespaces, and folders.

Do not blindly rename classes to shorter names. Local readability still matters.

---

## 2. Inverted dependency or control flow

Look for lower-level code depending on higher-level orchestration code.

Smells:

```txt
Model -> Controller
Query -> Service
Repository -> Action
DTO -> Service
ValueObject -> Request
```

Ask:

* Is this lower-level object reaching upward?
* Is this dependency direction intentional?
* Has the lower-level object become orchestration?
* Should the higher-level workflow be moved somewhere else?
* Can the lower-level object expose a primitive operation instead of calling the workflow?

Principle:

> Lower-level code should not need to know who orchestrates it.

---

## 3. Controllers escaping the seven-method shape

Controllers should generally prefer the conventional resource methods:

```txt
index
show
create
store
edit
update
destroy
```

When a controller exposes additional route methods, treat it as a smell, not an automatic error.

Example:

```php
class UsersController
{
    public function index() {}
    public function show() {}
    public function store() {}
    public function update() {}
    public function destroy() {}

    public function activate() {}
    public function deactivate() {}
    public function resendInvitation() {}
    public function impersonate() {}
}
```

Ask:

* Is this still one resource?
* Are these actually separate workflows?
* Is the controller becoming a menu of operations?
* Is a new controller or resource trying to emerge?
* Is the code avoiding a structural decision?

Principle:

> The seven-method constraint is useful because it forces the developer to think about structure.

Do not frame this as REST dogma. Frame it as architectural pressure.

---

## 4. Same action split by presentation

Look for controller methods that do essentially the same work but return different representations.

Example:

```php
public function index()
{
    $users = $this->getUsers();

    return view('users.index', compact('users'));
}

public function userListJson()
{
    $users = $this->getUsers();

    return response()->json($users);
}
```

The smell is not JSON itself. The smell is duplicating the same application action because presentation concerns leaked into controller structure.

Ask:

* Are these really two different use cases?
* Or is this the same use case rendered differently?
* Should HTML and JSON delivery be separated?
* Should there be a dedicated API route/controller?
* Is duplicated data retrieval suggesting a query object?
* Is the controller mixing application behavior with presentation concerns?

Principle:

> A different representation is not automatically a different action.

---

## 5. Generic Request instead of Form Request

When a controller performs validation or request data preprocessing, prefer a dedicated Form Request over the generic framework request.

Smell:

```php
public function store(Request $request)
{
    $validated = $request->validate([...]);
}
```

Prefer:

```php
public function store(StoreUserRequest $request)
{
}
```

Ask:

* Is validation hidden inside the controller?
* Is preprocessing mixed with controller behavior?
* Would a Form Request make the input contract explicit?
* Would authorization, validation, and normalization be easier to discover?

Principle:

> Prefer framework primitives that create explicit boundaries over generic primitives that require manual discipline.

---

## 6. Arrays, collections, or loose values crossing boundaries

When data moves between parts of the application, prefer explicit data contracts.

Smells:

```php
$service->create($data);
$action->handle($user, $payload);
$job->dispatch($items, $options, $flags);
```

Ask:

* What shape is this data?
* Which fields are required?
* Who owns this structure?
* Has the data been validated or normalized?
* Are several values always passed together?
* Would a DTO or value object make the boundary clearer?

Principle:

> Arrays are fine as local implementation details. They become architectural debt when they cross boundaries.

DTOs should carry data. Do not turn DTOs into hidden workflow objects.

---

## 7. God objects

Look for objects that attract unrelated behavior.

Signals:

* Too many constructor dependencies.
* Too many public methods.
* Methods belonging to unrelated concepts.
* The class changes for many unrelated reasons.
* New requirements keep being added to the same class.
* The object coordinates multiple domains.

Example:

```txt
UserService
  authenticate
  updateProfile
  sendEmail
  calculateBilling
  exportData
  syncExternalProvider
  checkPermissions
```

Ask:

* Why does this object keep attracting behavior?
* What concepts are hidden inside it?
* Are multiple domains being coordinated here?
* Is a missing service, action, query, policy, or value object trying to emerge?

Principle:

> A god object is often evidence that missing concepts have not been named yet.

---

## 8. Traits hiding complexity

Treat large traits as concealed responsibility.

Smell:

```php
class User extends Model
{
    use HasBilling;
    use HasNotifications;
    use HasPermissions;
    use HasExports;
}
```

Ask:

* Is this trait sharing small reusable behavior, or injecting a subsystem?
* Would the host class still look reasonable if all trait methods were pasted into it?
* Is the trait hiding god-object growth?
* Is the trait compensating for a missing collaborator, service, query, action, or value object?

Principle:

> A trait does not remove complexity; it only moves it out of sight.

---

## 9. Heavy branching

Look for large `switch`, `match`, or multi-branch `if/else` structures.

Smell:

```php
switch ($paymentMethod) {
    case 'credit_card':
        // ...
        break;

    case 'bank_transfer':
        // ...
        break;

    case 'paypal':
        // ...
        break;
}
```

Ask:

* Is each branch a different implementation of the same capability?
* Are new cases likely to be added over time?
* Is this branching repeated elsewhere?
* Would a strategy, handler map, enum method, polymorphic object, or pipeline make the variation explicit?

Principle:

> Branching is fine for simple decisions. It becomes architectural pressure when each branch represents a different implementation of the same capability.

Do not automatically implement a strategy pattern. First identify whether the branch represents stable variation.

---

## 10. Duplication

Treat duplication as evidence, not as an automatic command to abstract.

Ask:

* Is this accidental duplication or intentional duplication?
* Do these pieces of code change together?
* Is there a business concept appearing in multiple places?
* Is a query, action, DTO, service, strategy, value object, policy, or domain concept trying to emerge?
* Would abstraction reduce complexity, or just hide it?

Principle:

> Duplication is often a signal that a missing concept wants to be named.

Avoid premature abstractions. Similar code is not always the same concept.

---

## 11. Long parameter lists

Look for methods receiving many parameters, especially when the method performs many checks.

Smell:

```php
processPayment(
    $user,
    $amount,
    $currency,
    $method,
    $installments,
    $coupon,
    $metadata,
    $notify
);
```

Ask:

* Are these really separate values?
* Do some parameters form a meaningful concept?
* Are several parameters always passed together?
* Is the method validating too many combinations?
* Would a DTO, value object, command object, or dedicated class clarify the operation?

Principle:

> Long parameter lists often mean the code has a data shape but no data object.

---

## 12. Fluent interface opportunities

Look for operations that are naturally read as a sequence of steps.

Possible smell:

```php
$generator = new ReportGenerator($month, $user, true, false);
$generator->generate();
```

Possible direction:

```php
Report::make()
    ->forMonth($month)
    ->forUser($user)
    ->includeInactive()
    ->generate();
```

Ask:

* Is the current API hard to read on subsequent passes?
* Does the operation have a natural sequence?
* Would a fluent interface make the workflow easier to understand?
* Or would it just hide mutable state?

Principle:

> Consider fluent interfaces when they make intent read sequentially.

Do not use fluency just for style. A fluent interface should improve readability, not hide state.

---

## 13. Long inline queries

Look for verbose queries inside controllers, actions, jobs, services, or views.

Smell:

```php
User::query()
    ->where(...)
    ->whereHas(...)
    ->with(...)
    ->leftJoin(...)
    ->selectRaw(...)
    ->groupBy(...)
    ->orderBy(...)
    ->paginate();
```

Ask:

* Is this query simple enough to understand inline?
* Does it encode a business read that deserves a name?
* Is it likely to be reused by HTML, JSON, exports, jobs, reports, or APIs?
* Would extracting it reduce noise in the surrounding code?
* Would a dedicated query class make the use case easier to read?

Principle:

> Long queries are often unnamed business reads.

---

# Suggested output format

When reporting findings, use this shape:

```txt
Observed:
Concrete code facts from the inspected scope.

Where:
File, class, method, route, job, or call site where it appears.

Evidence signals:
- Signal 1
- Signal 2

Architectural pressure:
What structural decision the code may be avoiding.

Decision to make:
The design choice the team should explicitly make.

Possible directions:
- Direction 1
- Direction 2
```

Keep findings practical. Prefer a few high-signal observations over a long list of minor complaints. Include a short watchlist only for weak signals that should not be promoted to candidates.


# Present candidates as an HTML report

Before writing the report, read and follow [HTML-REPORT.md](HTML-REPORT.md) for the full HTML scaffold, Mimir/temp-path rules, diagram patterns, and styling guidance.

Each report should include a compact **Scope inspected** section before the candidates:

- inspected files/folders/routes/workflows
- entrypoints followed
- nearby files added to scope
- out-of-scope areas, when relevant

Each candidate should include:

- **Files** — which files/modules are involved, with line ranges when useful
- **Evidence** — a short code excerpt or compact call-site list that shows the problematic shape
- **Evidence signals** — at least two signals from the reporting bar
- **Observed** — code facts, not interpretation
- **Problem** — why the current architecture is causing friction, stated in developer-concrete terms
- **Architectural pressure** — what design decision the code is avoiding
- **Decision to make** — what the team should explicitly choose
- **Possible direction** — plain English description of what would change; do not overstate as the only solution
- **Benefits** — explained in terms of locality and leverage, and how tests would improve
- **Before / After diagram** — side-by-side, custom-drawn, illustrating the shallowness and the deepening
- **Before / After pseudocode** — compact call-stack or workflow pseudocode showing current shape and proposed shape
- **Recommendation strength** — one of `Strong`, `Worth exploring`, `Speculative`, rendered as a badge

End the report with:

- **Watchlist** — optional, weak signals that did not clear the reporting bar
- **Top recommendation** — which candidate you'd tackle first and why, using explicit priority logic: leverage, locality, test cost, blast radius, and migration safety

# Final instruction

Do not behave like an auto-refactor tool.

Behave like an architectural radar.

Your job is to detect where the code is asking for a design decision.

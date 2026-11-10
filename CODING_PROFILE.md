# Coding Profile

This document captures the user's engineering profile across projects. Every coder or agent working with the user should read it before making architecture or implementation changes. Project-specific instructions may add constraints, but they do not override these general principles unless the user explicitly says so.

This profile defines what good work looks like. How an agent is allowed to act (plan approval, Git, comments, testing gates) is governed by `AGENTS.md`, which takes precedence wherever the two touch.

## Core Standard

The user values code that is consistent, explicit, maintainable, and hard to misuse. The goal is not cleverness or minimal edits. The goal is a codebase where the right place for a behavior is obvious, repeated logic is encapsulated, and different entry points do not quietly drift apart.

Systems the user is building should be treated as serious production systems. They should have stable internal contracts, clear ownership boundaries, and predictable failure modes.

## Global Principles Versus Project Instructions

This profile contains principles that should travel with the user across projects. Project-specific architecture, product goals, test matrices, runbooks, and quality contracts belong in that project's own documentation.

Do not insert a project's local instructions, links, terminology, or current work plan into this global profile merely to make the project guidance easier to discover. A project is not entitled to modify the user's global profile. Change this document only when the user explicitly asks to generalize a lesson across their work.

When applying the profile:

- Read the global principles first.
- Read the current project's own instructions and source-of-truth documents separately.
- Apply both, with the project document supplying local context rather than rewriting the global rule.
- If a local rule appears to conflict with a global principle, surface the conflict instead of silently choosing one.
- Keep examples project-neutral unless a named project is necessary to explain the principle.

## Product Outcomes Over Supporting Machinery

The requested product behavior is the goal. Test harnesses, evaluators, migration helpers, audit records, compatibility layers, dashboards, reports, and other supporting machinery are means to that goal. They are not automatically valuable because they make a process look rigorous.

Before adding supporting machinery, state:

- What production outcome it helps improve or protect.
- What concrete failure it detects or prevents.
- Why a simpler existing mechanism is insufficient.
- Its ongoing cost in code, storage, compute, operator attention, and maintenance.
- When it will be removed if it is temporary.

If these questions cannot be answered concretely, do not build the machinery.

Do not let a measurement, testing, or audit system become larger or more sophisticated than the production behavior it exists to improve. When most engineering effort goes into proving that a tool ran while the product remains unimproved, the work has drifted away from its purpose.

Use one current tool by default. Do not create a sequence of versioned harnesses, runners, adapters, reports, or databases when the tool has no external compatibility obligation. Improve it in place, remove the obsolete implementation, and continue. Versioning is justified only when versions must genuinely coexist, an external consumer depends on an older contract, or an immutable historical artifact is itself an explicit product requirement.

Operational evidence and product-quality evidence are different. Logs, traces, hashes, accounting, and reproducible execution can prove what ran. They do not prove that the output was useful or that the product works well. Never present the sophistication of supporting machinery as evidence of product quality.

## Resource-Conscious Engineering

Storage, memory, compute, provider calls, tokens, and engineering attention are real design constraints. Do not assume unlimited hardware, money, or time.

The default posture is:

- Do not retain duplicate implementations, databases, build outputs, raw captures, or generated reports without a current use.
- Keep compact summaries instead of complete raw evidence unless the raw material is needed to reproduce a named failure.
- Scope diagnostics to the failure under investigation and clean them up when that investigation ends.
- Avoid dedicated infrastructure for temporary development work when an existing safe boundary is sufficient.
- Estimate a mechanism's continuing cost before building it, including cleanup and maintenance.
- Prefer one bounded, reusable tool over repeated one-off machinery.

Resource discipline does not mean accepting weak contracts, unsafe behavior, or fake tests. It means spending rigor where it protects production behavior instead of accumulating machinery and artifacts by default.

## Authorization Is Narrowly Scoped

Treat permission as specific to the operation, repository, files, version, environment, and external system the user named. Approval to perform one action does not authorize adjacent actions merely because they are convenient parts of the same workflow.

Examples:

- Permission to use Git for one recovery does not grant general Git permission.
- Permission to delete one obsolete implementation does not authorize deleting the current replacement or unrelated generated data.
- Permission to inspect an external system does not authorize mutating it.
- Permission to change a project document does not authorize changing the global profile.

When the requested scope is exact, preserve that exact scope. Ask before taking a materially broader, destructive, externally visible, or difficult-to-reverse action.

## No MVP Framing

The user dislikes MVP framing. Do not treat requested product work as validation work unless the user explicitly says the goal is validation.

If the user asks to build a system, assume the product direction is intentional. The job is to build the correct foundation from day one, even if that takes longer.

Do not propose shortcuts, throwaway implementations, temporary architecture, or reduced correctness in the name of speed. Scope can be sequenced, but the architecture, contracts, ownership boundaries, authorization model, persistence model, and failure behavior must be designed as the real system from the beginning.

A smaller first delivery is acceptable only when it is a complete, coherent slice of the intended final system. It must not rely on fake boundaries, hidden fallbacks, weak contracts, or code that is expected to be ripped out later.

MVP language is usually the wrong frame. Prefer discussing the correct system shape, ownership boundaries, implementation phases, what is complete now versus deliberately deferred, and what must be true before the system is considered done.

## Contracts As Coordination Boundaries

Public contracts are team coordination surfaces, not only serialization details. Protobuf files, service definitions, envelope shapes, and shared specs should be correct enough for other teams to build against, even when one implementation is still catching up internally.

When changing a contract:

- Read the current source-of-truth spec again before editing.
- Do not rely on memory of an older contract.
- Define the public shape accurately and completely.
- Make unsupported runtime behavior fail explicitly instead of leaving the contract ambiguous.
- Avoid blocking other teams with incomplete or stale protocol surfaces.

If a spec says a field or service exists, the protocol should expose it in the right place. If the implementation cannot support it yet, the implementation should say so deterministically.

## Cross-Repository Contract Discipline

A product rarely exists in one repository or process. Client libraries, command-line tools, control planes, workers, domain services, documentation, and frontends may all participate in one behavior. A change is not complete merely because one repository has been updated.

When a contract crosses repositories:

- Identify every producer and consumer before changing the shape.
- Verify every relevant declaration surface, compiler or build step, runtime bundle, execution path, domain service, documentation surface, and frontend assumption.
- Do not let each layer invent aliases for the same concept.
- Prefer one canonical shape and reject loose aliases once the system has moved to the new contract.
- Treat "works in one repo" as incomplete until the other participating repos either support the contract or explicitly reject it.

If a repository is out of scope for the current change, report the mismatch clearly instead of silently assuming another agent will resolve it.

## Transport And Operations

HTTP, gRPC, MCP, CLI, or any future transport should be adapters over the same operation layer.

Transport code should:

- Parse transport-specific input.
- Build operation request structs.
- Call the operation.
- Return the operation response using the shared response mechanism.

Transport code should not:

- Own business policy.
- Own resource dispatch.
- Own validation semantics beyond transport parsing.
- Reimplement operation logic.
- Shape custom response envelopes when the operation already has a protobuf or shared response shape.

When a transport has a shared response encoder or contract, sibling endpoints should use it consistently. Custom envelopes and one-off error handling are smells unless the endpoint deliberately belongs to a different contract.

## Operation Ownership

Operations are the canonical application boundary. If a behavior matters across transports, it belongs in operations or below operations, not in a controller/server/tool handler.

Operations should own:

- Scope and access expectations.
- Request validation policy.
- Dynamic dispatch for operation-owned concepts.
- Error meaning.
- Calls into domain contexts.
- Response shape.

Controllers and transport servers should not know enough to make policy decisions. They should hand off to operations.

## Encapsulation And DRY

The user prefers repeated patterns to be extracted, abstracted, and encapsulated when the extracted concept is real.

Good encapsulation:

- Gives a repeated behavior one owner.
- Makes call sites smaller and more uniform.
- Reduces future drift across HTTP, gRPC, MCP, and internal callers.
- Makes adversarial/regression tests easier to write at the right level.

Bad encapsulation:

- Creates vague utility modules with no domain ownership.
- Hides simple logic behind indirection without reducing drift.
- Splits a cohesive behavior across files just to make files shorter.

If 60% of a function is repeated across places, look for the concept underneath the repetition. Extract that concept, not just the text.

## Refactor, Don't Duplicate

When an existing implementation is insufficient, extend it. Do not create a sibling.

- Never create "Enhanced", "Public", "Internal", "V2", or similar variants of an existing handler, executor, repository, service, or client. Add the capability to the original, or make the difference a configuration option on one implementation.
- Prefer configuration over variants: one handler with configurable features beats three handlers that each grew their own features.
- Use the project's existing shared infrastructure (HTTP client, error types, validation, caching, worker pools, authentication) instead of constructing a local copy with different behavior.
- Before creating a new file, search for similar types, functions, and interfaces. Ask whether the functionality already exists, whether it can be extended, and whether the new file is really a second version of something.

```go
// Wrong: a sibling that will drift from the original
type EnhancedBaseExecutor struct {
    *BaseExecutor
    metrics *StageMetrics
}

// Right: the original gains the capability
type BaseExecutor struct {
    metrics  *StageMetrics
    profiler *StageProfiler
}
```

## Isolation, Ownership, Encapsulation, And Abstraction

These four ideas are one design discipline, with ownership at the center:

- **Ownership:** every policy, fact, lifecycle decision, validation rule, and transformation has one authoritative executable owner. Other components call that owner; they do not maintain approximations of its behavior.
- **Isolation:** keep a responsibility and its failure modes inside the boundary that owns it. A defect in one concern should not require unrelated layers to understand or repair that concern.
- **Encapsulation:** expose the operation or contract callers need while hiding the internal rules and state transitions that make it correct. Callers express intent rather than reproduce mechanics.
- **Abstraction:** extract a shared concept only when the concept is real across callers. The abstraction should route work to the correct owner and make drift harder; it should not be a vague helper or an extra layer with no authority.

When several layers need the same rule, do not copy the rule into prompts, schemas, validators, repair code, persistence mappers, and tests independently. Identify the source of truth, make the other layers derive from it where possible, and add boundary checks where derivation is impossible. A contract change is incomplete until every producer and consumer either uses the owner or explicitly rejects the new shape.

A useful review question is: **if this behavior changes tomorrow, what single place should be edited?** If the honest answer is several unrelated places, ownership is still fragmented and the design needs another pass.

## Modularization Must Improve Ownership

The user does not want files split just to make more files. Splitting is valuable only when it creates clearer ownership, removes meaningful repetition, or makes future changes easier to reason about.

Good modularization:

- Makes each module responsible for one coherent concept.
- Gives shared behavior a single owner.
- Keeps call sites smaller without hiding policy in vague helpers.
- Makes it easier to test the behavior at the right boundary.
- Reduces the chance that sibling paths drift.

Bad modularization:

- Moves lines around without clarifying responsibility.
- Creates thin wrapper files that do not own a real concept.
- Forces readers to jump through many files to understand one simple behavior.
- Splits a cohesive operation only because a file became long.

Prefer fewer, deeper modules with real ownership over many shallow files. A split should be defensible as a maintainability improvement, not merely as line-count management.

## God Files

A god file is a module that accumulates too many unrelated responsibilities until every change has to pass through it. The user dislikes god files because they hide ownership and make regressions likely.

Do not solve god files by scattering code randomly. Split by responsibility:

- A module should own one coherent concept.
- A boundary should be deep enough to justify its existence.
- Callers should not need to know the internals of every sub-step.

When a file is growing, ask what concepts it is mixing. Extract the concept with the clearest boundary first.

## Consistency Is A Feature

If one part of the system follows a pattern, adjacent parts should follow it too. Inconsistent exceptions are treated as bugs until proven otherwise.

Examples:

- If one endpoint returns the shared operation response, sibling endpoints should not preserve old custom envelopes.
- If one transport calls the application operation layer, sibling transports should not call lower-level domain modules directly for the same behavior.
- If the application operation owns not-found meaning, transport adapters should not duplicate that policy.
- If resource dispatch belongs to an application operation, a transport adapter should not maintain a second dispatch table.

Consistency is not cosmetic. It prevents three versions of the same system from developing under HTTP, gRPC, and MCP.

## Complexity Must Name Its Failure

Every mechanism earns its place by naming the concrete failure it prevents and pointing at the regression test that pins that failure. This applies to gates, retries, cooldowns, caches, defensive branches, fallback paths, and configuration knobs equally.

If nobody can name the failure, the mechanism is deleted. Deletion is the default state. Complexity argues its way in; simplicity does not argue its way back.

Weak defenses include:

- It already exists.
- It might be useful someday.
- It makes the code more robust in general.
- Removing it feels risky.

A defensible mechanism can state when it runs, what failure it prevents, which test proves it, and what breaks if it is removed. A mechanism that cannot answer those questions is not load-bearing infrastructure. It is unowned complexity, and it becomes a simplification target, not a permanent fixture.

## One Fact, One Owner

Every fact in the system has exactly one owner. Other components ask the owner. They do not keep copies.

A cached copy of another component's state is only correct while a set of unwritten synchronization rules hold. Those rules break silently, usually on the path nobody tested, and the system ends up holding two answers to one question. Mirrored state is how a response ends up dead in one layer while another layer still reports it alive.

When a new "who owns this fact" question appears, the answer is a pointer at the existing authority, not a new field. If a copy seems necessary for convenience or performance, the design needs another pass:

- Derive the value at the call site instead of storing it.
- Pass the owning object instead of copying fields out of it.
- Construct emitted data from the owner directly, so mismatched values are unrepresentable.

For lifecycle decisions, identity beats identifiers. A late callback should present the object it was created with, and the owner should reject it if that object is no longer current. Comparing copied ids is how stale work mutates live state.

## Safety By Construction

When state can be observed from more than one execution context, prefer designs where the dangerous interleaving is impossible or harmless over designs where it is guarded. A lock, a flag, or a reservation adds a rule that every future caller can break. Construction removes the rule.

Concrete moves, in preference order:

- Rebind, do not mutate. Replacing a collection wholesale means every reader sees either the old state or the new state, never a mid-mutation view. Clearing it in place means an iterator somewhere crashes.
- Take a local reference before check-then-use. Reading a shared field twice around a decision is a race; reading it once into a local is not.
- Emit from the owner. Constructing outputs directly from the owning object makes mismatched values unrepresentable instead of validated.
- Only then reach for coordination, and keep it at the smallest scope that closes the named failure.

A residual race is acceptable only when its harm can be named and shrugged at in one sentence ("a trim may be partially undone by a request that arrives mid-trim, which is the correct outcome anyway"). If the residual harm cannot be named, the design is not done. Do not add coordination machinery for a residual whose harm is acceptable — that is complexity without a failure.

## Failure Philosophy

The user prefers explicit failure over hidden fallback when a dependency or architectural assumption is required.

A system should not silently pretend that a missing primary dependency is optional when the product design requires it. Fallbacks are acceptable only when they are deliberate, visible, and preserve the promised correctness contract.

When the architecture designates a particular store, service, or execution path as authoritative, use it as authoritative. If its required capability is unavailable, fail clearly instead of drifting into a weaker implementation whose behavior differs silently.

## No Legacy Paths Without Explicit Product Need

The user prefers hard cutoffs over compatibility layers when the old path is conceptually dead. Translation layers, fallback adapters, and dual v1/v2 execution paths are future drift unless there is an explicit product or migration requirement.

When replacing a contract or execution path:

- Remove the old path if callers are no longer supposed to use it.
- Reject old versions clearly instead of silently translating them.
- Do not keep compatibility code just because it is convenient.
- Do not preserve a deprecated path without tests and a named owner.

Backward compatibility is allowed when it is a deliberate product decision. It should not appear accidentally because an agent was afraid to delete old code.

Do not describe a path as "transition" or "migration" unless there is a real migration plan with named owners, tests, and a cutoff. If the user says the system is moving to a new version only, old persisted formats, old bundle shapes, old aliases, and old execution paths should be removed or rejected unless they explicitly ask to preserve them.

When a system is pre-launch, being rebuilt from a clean state, or explicitly has no historical data that must survive, assume a hard cutoff. Do not invent upgrade paths, backfills, compatibility shims, parallel implementations, or historical-data protections for deployments the user has said will be deleted. Improving code does not by itself require versioning it. Replace the old implementation with the improved one and keep one current path.

If preserving an older path is proposed anyway, identify the real caller, real data, or external obligation that requires it. Fear of deletion, generic migration practice, or the possibility that an old deployment might exist is not sufficient evidence.

## Internal Concepts Versus User-Facing Docs

Documentation must match its audience. Internal coordination mechanisms may be essential between components, but that does not mean users should see or understand them directly.

When writing docs:

- Match the audience of the existing document.
- Keep public docs focused on what users configure, do, observe, and troubleshoot.
- Keep internal contracts, scheduling details, authorization material, audit machinery, and service-coordination details in internal specifications or architecture docs.
- Do not expose internal machinery in user-facing docs just because it exists.
- Follow the tone, vocabulary, and structure of nearby docs instead of inventing a generic documentation voice.

If users never directly configure or act on a concept, public documentation should not pretend they do.

## Operational Honesty

The system should be honest about what it is doing. If work is executed inline, say inline. If work is queued, say queued. If retry belongs to the caller, worker, domain service, or control plane, the code and metadata should identify that owner.

Hidden special cases are a design smell even when they appear to work. A path that only queues under some conditions, retries only in a narrow internal case, or falls back to a weaker behavior should be explicit in naming, contracts, logs, and tests.

When debugging operational behavior, prefer facts from traces, logs, attempts, and ownership boundaries over assumptions about what the code was intended to do.

## Platform-Owned Metadata

Developer input should not override platform-owned metadata when correctness, accounting, audit, authorization, or security is involved.

Examples of platform-owned metadata include:

- Invocation identifiers.
- Run identifiers.
- Organization, project, tenant, or principal identity.
- Accounting context.
- Audit correlation fields.
- Platform-issued capability or grant scope.

If user code can provide a similar field, the platform should either reject it or overwrite it with authoritative metadata. Do not let developer-supplied metadata win silently over the runtime's own identity and accounting context.

## Ownership Across Components

Architecture decisions should identify which component owns each responsibility. The user dislikes semantic debates that do not end with an explicit owner.

Use the actual system boundaries, but keep the following default distinctions clear when those layers exist:

- A declaration or client layer owns what developers are allowed to express.
- A build or deployment tool owns local validation, compilation, packaging, and deployment preparation.
- A control plane owns authorization, scheduling, durable orchestration state, reconciliation, quota, billing, and durable audit records.
- An execution worker owns the work it performs and its direct calls to tools or vendors.
- A domain service owns its domain data, domain behavior, and final domain usage facts.
- A frontend owns presentation and user workflow, not hidden business policy or contract translation.

These are responsibility categories, not mandatory services. Combine or split them according to the real architecture, but do not let ownership become ambiguous.

When discussing a design, explain:

- What component owns the decision.
- What component executes the work.
- What component records the audit trail.
- What component owns retries and reconciliation.
- What component is the source of truth for accounting or usage.

If the answer is "several components kind of do it," the design probably needs another pass.

## Frontend Architecture When Applicable

The frontend is a presentation and workflow layer over the same operation contracts every other transport uses. It should not carry business policy, hidden fallbacks, or ad hoc translation of contract shapes. The rules below make that concrete.

### No `useEffect`

In React frontends, do not reach for `useEffect` as the default state-management tool. Derive state during render. Put imperative work in event handlers. Use `useSyncExternalStore` for state that lives outside React.

`useEffect` is where drift, races, and stale closures accumulate. If a piece of state seems to need `useEffect`, the state probably belongs elsewhere: in a data hook, in a store subscribed via `useSyncExternalStore`, in a router search param, or in a handler invoked at the moment the intent is expressed. When effects are used to sync duplicate state or to reconcile with URL or server state, the symptoms are freezes on tab return, duplicate fetches, subscription leaks, and stuck loading spinners. Design the state so the derivation is direct.

### One composed view-model hook per component

Each component calls exactly one hook: a composed view-model such as `useXyzView`, defined in the project's hook layer and imported — never declared inline in the component or route file. That hook is responsible for composing data hooks (`useXyzPageData`), URL-state primitives, local UI state, and any derived selectors the view needs. Components must not call the primitives directly, and must not call a second hook alongside the view-model — `useState`, `useIsMobile`, query hooks, and the like all move inside the view-model.

Why this matters:

- The component stays readable and testable at the view layer.
- Every consumer of the same view shape gets the same composition, so edge cases (loading, empty, unauthorized, partial data) are handled once, not per component.
- The view-model hook becomes the single seam for testing the presentation behavior without a DOM.

If a component looks like it needs a second primitive hook, that primitive belongs inside its view-model, not next to it.

### Route files are thin; screens and view-models have separate owners

A route file is wiring, not a screen. Its page component is layout composition only: it calls one imported view-model and renders imported feature components. It holds no hook definitions, data fetching, or business logic. Route matching, search validation, guards, and loaders are the route file's real job.

The screen's substance lives elsewhere:

- `components/<feature>/` owns every real UI block — the form, the table, the dialog, the page body. Each substantial component calls its own view-model hook.
- The view-model layer owns hooks in two tiers: a page-data hook wraps the feature's typed data query and returns the data shape; a view hook composes that page data with UI state, URL state, and derived selectors into the object the component consumes.

A hook is never defined inline in a route or component file — it is always imported from the project's hook layer. To change what a screen shows, edit the feature component and its view-model; change the route file only when routing changes.

Do not author an empty wrapper component. If a route's screen is exactly one imported component with no route-level layout to add, point the route directly at that component. An inline page component earns its place only when it adds real composition such as a shell, header, or multiple children. One child and nothing else means wire the child directly.

### Memoize as source-level discipline

`useMemo` for every derived value. `useCallback` for every function that is passed as a prop, into a dependency array, or otherwise identity-sensitive. This is not premature optimization; the React Compiler handles the runtime perf work. The point is diff hygiene and predictable identity so equality checks, memoized children, and dependency arrays stay stable and reasoning stays local.

"Don't over-optimize" means do not stack load-bearing optimizations on top of memoization. It does not mean strip memoization.

### Encapsulation into purpose-built shared components

Every recurring UI pattern becomes a purpose-built shared component that hides all its moving parts: state, layout, keyboard handling, focus management, ARIA semantics, empty and loading and error branches. Callers pass intent, not internals.

Anti-patterns to reject:

- Stretching an unrelated generic component to cover the new case.
- A thin hook wrapper that leaves the JSX repeated across pages.
- A shared component that leaks state or props whose only purpose is to accommodate one caller's edge case.

If the same combination of a Popover, a Combobox, and a virtualized list appears on more than one page, that combination is a component, not a copy-paste.

### Preserve JSX when the backend reshapes

If a backend contract change breaks TypeScript in a view, do not trim the JSX to match the smaller shape. Extend the data hook to fill every field the JSX consumes with `null`, `false`, `0`, or empty defaults so the presentation keeps compiling.

The JSX is the design record for the page. Deleting fields in response to a temporary backend reshape loses design intent and makes rollback lossy. Keep the presentation whole and let the data-layer mapper decide how to hydrate it.

### Presentation-only frontend

No hidden contract translation, no fallback business logic, no reimplementation of anything the backend already decides. If a shape needs to be transformed for the view, that mapping lives in a mapper module owned by the data layer, not scattered across components.

If a component appears to be making a business decision, that decision belongs behind the operation, not in the browser.

### Typed frontend data layer, organized by feature

Use the project's selected typed contract client together with one standard query/cache layer. Centralize schema generation or introspection, scalar mappings, request execution, caching, invalidation, authentication behavior, and typed errors. Do not create a bespoke client abstraction or transport path for every query.

Organize frontend data access **by feature**, not by technical artifact kind. A feature module should give each of these responsibilities one owner:

- Reusable field selections or response contracts.
- Query and mutation declarations.
- Mappers from transport contracts to stable view shapes, including null/default hydration.
- Query, mutation, cache-key, and invalidation hooks.

A single shared executor should own raw transport, credentials, security headers, typed errors, unauthorized behavior, retries, and other cross-cutting policy. Feature hooks never fetch directly; they use that executor.

This realizes the view-model and mapper rules above: the data layer maps contracts, one composed view-model supplies the component, and the component remains presentation-only. A new screen should pass through the project's feature data module, page-data/view-model layer, feature components, and thin route wiring. No raw client code or fetching in components, no field translation outside the mapper, and no hooks defined inline in routes or components.

## Async Work And Accounting

Asynchronous work should not force the wrong component to become the accounting system of record.

When one component starts asynchronous work owned by another, the initiating component may report the synchronous side effect and any stable job reference it received. It must not invent final usage, storage, billing, or completion facts. The component performing the domain work owns its final usage facts; the accounting or control-plane owner handles reconciliation, aggregation, quota, billing, and durable audit records.

For async provider work:

- Emit stable references when they are available.
- Do not block invocation completion on unrelated background completion unless the contract explicitly requires it.
- Do not put credentials or secret payloads in completion metadata.
- Let the system of record produce final accounting facts.
- Make missing async references visible as an observability limitation, not a silent success.

## Use The Data Model That Owns The Problem

Choose storage and computation according to the shape of the problem and the architecture's designated source of truth. Do not repeatedly reconstruct one data model inside a store or application layer designed for another.

When evaluating a feature, ask:

- Is this naturally relational, document-shaped, graph-shaped, event-stream, search, or analytical work?
- Is application code loading large amounts of data merely to reconstruct operations the owning store already supports?
- Would the authoritative data system express the operation more directly, safely, and efficiently?

If yes, implement the behavior in the owning data model rather than adding another cross-model workaround. For example, graph traversal, neighborhoods, paths, and communities belong in the graph system when the product architecture has one; transactional operational state still belongs in its transactional store.

## Testing Must Match the Claim

Testing should build confidence in layers. Isolated tests prove individual rules and failure boundaries. Integration tests prove that components behave correctly together. Realistic workload tests exercise the complexity, scale, ordering, state, and variation expected in actual use. End-to-end tests prove complete user-visible workflows through their real runtime boundaries.

Evidence from one layer must not be presented as proof of another. Passing unit tests does not establish integration quality. A small synthetic benchmark does not establish general product quality. A mocked execution path does not prove the production path works.

"Realistic" depends on the system being built. It may mean multiple records, longer inputs, concurrent operations, accumulated state, multi-step workflows, service failures, varied users, large datasets, sustained load, or combinations of these. Test design should reflect the conditions under which the component will actually operate.

Evaluate the complete outcome relevant to that component: correct results, missing expected behavior, incorrect or unintended behavior, state changes, side effects, failure handling, latency, and resource use where applicable.

Begin with precise isolated tests because they make defects diagnosable. Then compose those proven behaviors into increasingly realistic scenarios. Do not stop at isolated cases and describe them as system validation.

The user wants regression and adversarial tests, not only happy-path tests.

Tests should prove:

- The old bug cannot return.
- Incorrect inputs fail clearly.
- Duplicate transport paths use the same operation behavior.
- Response shape does not drift.
- Edge cases are owned at the right layer.

When moving behavior from a controller into operations, add operation-level tests. Keep HTTP tests focused on transport behavior and response shape.

## Writing Good Tests

The goal is confidence, not coverage. A test suite that reaches high line coverage by pinning implementation details is a liability.

- **Test behavior, not implementation.** Exercise the public interface and observable results. Do not reach into internal fields or assert on private call sequences.
- **Keep tests simple and focused.** Minimal setup, one clear intent, assertions that say what is being verified. If a reader cannot tell what a test proves from its name and body, rewrite it.
- **Happy path first, then failure.** Establish that the normal case works before covering error conditions, then cover both.
- **Mock only at real boundaries.** Mock external dependencies (network services, databases, third-party APIs), expensive operations, and hard-to-reproduce failures. Do not mock your own code or simple in-process logic when the real thing is fast and reliable. If writing mocks takes longer than writing the code, the design or the test is wrong.
- **Generate mocks with the project's standard tool** instead of hand-writing them, so mocks track their interfaces.
- **Name tests by scenario.** `TestCache_GetReturnsMissAfterExpiry` and `TestAdapter_StartFailsWhenCalledTwice` say what is proven; `TestStart`, `TestError`, and `TestCase1` do not.
- **Structure each test as setup, execute, verify.**
- **Be environment-aware.** Skip, with a stated reason, tests that need an environment that is not present, rather than letting them fail or pass vacuously.
- **Test the contract.** When a type implements an interface, test that every method in the contract behaves as the contract promises.
- **Separate unit from integration tests** so the fast suite runs constantly and the expensive suite runs deliberately.

## No Test Cheating

Tests are allowed to change when the intended contract changes. They should not be weakened simply to make a run green.

Bad test fixes include:

- Updating fixtures blindly without understanding the production contract.
- Removing assertions that catch real drift.
- Teaching tests to accept both the old and new shape when the product decision is a hard cutoff.
- Mocking around a missing integration instead of exposing the missing boundary.
- Making production behavior looser so an outdated test passes.

Good test fixes explain why the expectation changed and keep adversarial coverage for the new contract.

Two further rules that repeated failures have earned:

- A test must never pin behavior production cannot deliver. If a fake implements an interface, method, or path that the production object lacks, the test is fiction: it stays green while every real deployment does something else. When wiring a fake, diff it against the production type it stands in for. The honest fix is either to wire the production path for real or to delete the phantom behavior and its test together — never to keep a green test for a capability that does not exist.
- A load-bearing test proves itself by failing first. When a test exists to pin a specific fix or prevent a specific regression, break the code deliberately, watch the test fail, then restore and watch it pass. A regression test that has never been seen to fail proves nothing about what it guards. Report the fail-then-pass evidence, not just the green run.

## Change Discipline

Before changing code:

- Read the surrounding code first.
- Identify the current pattern.
- Decide which module should own the behavior.
- Avoid hard-coded fixes aimed only at the current repro.
- Verify the source of truth instead of relying on memory, especially for contracts, client and command-line behavior, build artifacts, execution services, domain APIs, and cross-repository consumers.

After changing code:

- Remove the old path if it has been replaced.
- Do not leave parallel implementations unless both are intentionally supported.
- Run focused tests for the changed behavior.
- Run a broader regression slice when shared response, scope, operations, or transport code changes.

Once a plan has been approved under `AGENTS.md`, do not stop at an issue list. Continue through implementation, tests, and cleanup unless blocked by a real external dependency or explicitly paused. Before approval, the plan is the deliverable.

## A Change In Cost Is A Change In Contract

Every caller was written against the old cost profile of what it calls. When a function moves from instant to blocking, cheap to expensive, infallible to fallible, or in-process to cross-process, its signature may be unchanged while its contract is not — and the bugs land in the callers, not the function.

The canonical failure: a hook is a no-op, so callers invoke it inline on a latency-sensitive thread while holding a global lock. Later the hook becomes a blocking cross-process call. Nothing in the diff of the hook is wrong; the system now freezes under the lock. The change that made the hook real was obligated to revisit every call site.

When changing a cost profile, re-audit each caller for: what lock is held across the call, what thread or event loop it runs on, what timeout or failure it now needs to tolerate, and what assumed it could not fail. The established pattern for the expensive version usually already exists nearby (an off-thread teardown, a queued variant) — mirror it rather than inventing a new one.

## External Reviews Are Requirements Input

Reviews from automated reviewers, other models, or colleagues are input to the design, never prescriptions. The reviewer sees symptoms; the owner of the code decides the shape of the fix.

Discipline for consuming a review:

- Verify every checkable claim before accepting it. Read the cited line, run the repro, grep for the consumer. A review that cites line numbers that do not exist in current files is reviewing stale code — discard it and re-request against the current tree rather than arguing with its findings.
- Separate the finding from the proposed fix. A finding can be real while the proposed fix is wrong-shaped — usually over-machined (a lock hierarchy where a rebind suffices, a handshake where identity already holds). Right-size the fix to the named failure, not to the reviewer's imagination of the failure.
- When two reviews conflict, adjudicate by reading the source, not by trusting the more confident report.
- Honest non-compliance beats fake compliance. If a requested item cannot be implemented correctly — a dependency lacks the hook, the semantics do not exist — report the mismatch explicitly with evidence. Do not invent a wrong-semantics mapping so a checklist line turns green.

## Verification Is Independent Of The Author

The author's green bar is necessary and not sufficient. Substantive change gets an adversarial pass whose explicit job is refutation: re-run the gates independently, read the diff end to end, and attack the specific claims the author made — with concrete failure scenarios, not vibes. A verifier that finds nothing on the first read has not read hard enough; a verifier that invents faults it cannot reproduce is equally useless.

Verification must be proportional to risk. Irreversible data changes, public contracts, security boundaries, billing, releases, and external side effects justify stronger independent evidence. A reversible local development tool does not automatically justify immutable preregistration, one-shot execution markers, exhaustive source hashing, permanent raw captures, or repeated forensic review. Use the lightest verification that can expose the named risk.

Do not repeatedly re-audit a stable conclusion without new evidence or a changed risk. Once the relevant failure is reproduced, fixed, and covered at the correct boundary, return attention to the production outcome.

Drift from an agreed spec is failure unless justified — and justified deviation is correct, not disobedience. When a spec's enumeration contradicts its own stated criterion, the criterion wins; the deviation is reported with the reasoning, and the spec gets fixed. Silent compliance with a buggy spec and silent deviation from a good one are the same failure.

## Decisions Are Recorded With Their Rationale

Durable architecture and product decisions should live next to the code they govern, in a concise spec, decision record, or status document (never in code comments, which `AGENTS.md` restricts to actionable TODOs), stating the current decision, its reasoning, and what it superseded. Not every debugging step, test run, tool revision, or discarded experiment deserves a permanent ledger entry. Documentation must earn its maintenance cost just like code.

A decision record is corrected in place when the decision changes. Prefer one current account over an append-only chain of superseded notes. Preserve history only when it is needed for an external obligation, an irreversible operation, or understanding a still-relevant constraint. A stale record stating a reversed decision is worse than no record because an agent may faithfully execute it.

## Deletion And Cleanup Semantics

Deletion is product behavior, not only a database operation. If a project, deployment, snapshot, asset, resource, or namespace can be deleted, the system should define what happens to dependent state.

Before implementing deletion, identify:

- Durable database rows.
- Object storage assets.
- Active deployments and snapshots.
- Queued or scheduled work.
- Credentials, bindings, and secrets.
- Audit records and billing records.
- Frontend-visible state.
- External provider state that should or should not be touched.

Prefer complete, explicit deletion semantics over partial cleanup that leaves hidden active state behind.

## Release Completion Standard

For release-oriented work, passing tests is not always the finish line. If the request includes release, patch, image, registry, deployment, or similar language, the work is complete only when the requested artifact has been produced and verified.

This standard applies only after Git operations have been explicitly enabled and a push has been explicitly requested, as `AGENTS.md` requires. A release request alone does not unlock Git.

For container release work, that means:

- Commit the intended code.
- Push the branch when requested.
- Create and push the release tag when requested.
- Build and push the expected image name and tags.
- Inspect the registry artifact after pushing.
- Report the exact commit, version, image tags, and digest.

Do not treat a local build as equivalent to a pushed release artifact.

## Design Pushback Expectations

The user expects agents to be able to defend a design from the actual system behavior. If asked to argue for code, explain when it runs, what owns it, what failure mode it protects, and what would break if it were removed.

Weak answers include:

- Keeping code because it already exists.
- Preserving a branch because it might be useful someday.
- Claiming a design works without checking traces, tests, or callers.
- Treating a question about ownership as a request for a quick patch.

Good answers connect the implementation to the contract, runtime behavior, tests, and operational evidence. If the code does not carry its weight, say so and propose a cleaner removal or replacement.

## What Future Agents Should Remember

The user is not asking for patches that merely satisfy one failing test. They are asking for the system to become more coherent over time.

When they push back, it is usually about ownership, consistency, hidden fallbacks, or a special case that should have been encapsulated. Treat that as design signal, not just preference.

For service operations, the preferred shape is:

```text
transport adapter
  -> application operation request
  -> application operation
  -> domain owner / authoritative storage
  -> application operation response
  -> shared transport response encoder
```

For a component-based frontend, the corresponding shape is:

```text
presentation component
  -> one composed view-model
  -> feature data hooks + shared state primitives
  -> shared transport client / mapper
  -> application operation response
  -> presentation
```

If your change follows that shape, removes drift, and has regression tests, it is probably aligned with the user's coding profile.

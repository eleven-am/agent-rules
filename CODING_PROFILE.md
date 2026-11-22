# Coding Profile

This profile defines the engineering standard for work across the user's projects. Read it before making architecture or implementation decisions. Apply it with the current project's instructions and authoritative specifications.

`AGENTS.md` governs authority, scope, approvals, Git, code comments, and mandatory test and build gates. It takes precedence wherever these documents overlap. `MEMORY_PROTOCOL.md` governs session continuity. This profile grants no additional permission to inspect, change, delete, publish, or release anything.

Project instructions supply local architecture, contracts, and requirements. Keep those details in the project. Change this global profile only when the user explicitly asks to generalize a lesson across projects. Surface instruction conflicts as `AGENTS.md` requires; engineering judgment does not authorize silently overriding a mandatory rule.

## 1. Engineering objective

**Deliver the requested behavior correctly and completely, while minimizing the effort required to understand, trust, operate, and change the resulting system over time.**

Readability and maintainability are part of the deliverable. Apply this standard while choosing the design, writing the implementation, and verifying the result. Do not treat structural quality as optional cleanup after the feature works.

The finished code should reveal the structure of the problem: its concepts, responsibilities, rules, and failure modes. Remove the accidental complexity accumulated during exploration. A reader should not have to reconstruct the agent's investigation to understand the solution.

These are concurrent acceptance requirements. They must remain visible in every substantive decision:

| Quality | Required result |
| --- | --- |
| Correctness and completeness | The actual requirements hold through the real execution path, including relevant failure conditions and operational constraints. |
| Readability and comprehension | A competent engineer can follow intent, control flow, state changes, and failure behavior without unnecessary mental reconstruction. |
| Conceptual integrity | The implementation expresses one coherent design with clear responsibilities and respected boundaries. |
| Encapsulation, DRY, and abstraction | Shared knowledge and behavior have authoritative owners; callers express intent through meaningful contracts. |
| Minimal necessary complexity | Each mechanism earns its ongoing cost through a concrete responsibility, requirement, or failure it addresses. |
| Consistency | Related paths use the same concepts, contracts, conventions, and shared infrastructure. |
| Maintainability and change locality | A change reaches the responsible owner and its real consumers without requiring unrelated code to understand that responsibility. |
| Engineering judgment | Trade-offs are grounded in the actual system, evidence, and the user's constraints. |

The agent may choose how to satisfy this standard. It may not waive, reinterpret, or lower the standard to accept its own implementation. Functional correctness does not compensate for unreadable structure, fragmented ownership, incomplete encapsulation, duplicated policy, or unnecessary machinery. A passing test suite, a confident explanation, and an existing precedent do not excuse a structural violation.

Treat required security and data integrity, authorized scope, explicit contracts, and every applicable engineering requirement here as constraints. Choose among compliant designs by their total burden of comprehension and change over the system's lifetime. Consider the implementation, callers, tests, operations, and dependencies together.

An instruction to prefer a design is a choice rule among compliant alternatives. An applicability condition identifies the kind of work covered; it does not let the agent opt out. A permitted technique still has to meet the other requirements. An explanation must show how the design satisfies them, rather than supply a reason to disregard them.

Only the user may authorize a departure from a mandatory standard. When compliance is blocked by scope, an external contract, or a real instruction conflict, identify the exact condition and its consequence. Continue independent authorized work, but do not silently relax the condition or claim the affected outcome is complete. Ordinary implementation choices within the standard remain the agent's responsibility.

The shortest implementation, smallest diff, fewest files, and most abstractions are not independent goals. Choose them only when they serve the engineering objective. Preserve required behavior and evidence when simplifying.

## 2. Readability is a design requirement

Write for the competent engineer who will encounter the code without the conversation that produced it.

### Make the domain visible in the principal flow

Model the operation through named inputs, outputs, responsibilities, and steps. A reader following its principal flow must be able to identify what the system is doing, in what order, and where each decision belongs without first reconstructing collection plumbing, callback construction, or storage mechanics.

Give each function a coherent responsibility. Calling a function an entire feature does not establish cohesion: distinguish the operation coordinating work from the policies and transformations it coordinates. Keep validation, state transitions, consequential side effects, and failure handling visible at the level that owns them.

An application operation must express its domain stages at a consistent level of detail. Substantial query construction, ranking policy, classification, enrichment, and mapping belong in named implementations at their responsible boundary. Do not interleave their internal mechanics with the coordinating flow. A query implementation may contain detailed SQL; its application caller should rely on the query's meaningful contract.

Use existing owners and keep related implementation close. This requirement does not mandate a new file, interface, service, or layer for each step. Do not move a confusing block behind `processData` or another vague wrapper and call that encapsulation. The extracted operation must have a clear purpose, a useful contract, and an independently understandable implementation.

### Keep data flow direct

Express distinct transformations in an order that a reader can follow. Use meaningful intermediate values when a nested expression makes the reader mentally evaluate several accumulation or transformation steps to discover the domain action. Do not compress an operation into nested calls merely to reduce line count.

Represent fixed domain cases directly through named values, meaningful types, straightforward control flow, or an established declarative representation. The supported cases and their meaning must be discoverable from that representation. Do not build a local predicate, callback, or factory mechanism to manufacture a fixed set of choices that can be expressed directly. A finite set of different cases does not establish a need for configurable execution machinery.

Put independently meaningful domain transformations and policies in named functions or methods at the owning layer. Do not hide them as inner helper definitions inside the operation coordinating them. Nested functions are allowed for an existing API's callback contract or necessary lexical capture; neither allowance permits a hidden business policy or another orchestration layer inside the callback. An ordinary sort comparator or event callback can adapt arguments and call the named owner.

Higher-order composition is appropriate when required by an existing framework, library interface, or product contract that actually accepts variable behavior. Preserve that contract, but keep domain policy directly identifiable. Do not invent such a requirement to justify machinery already written.

### Carry meaning through names and representations

Use the domain's vocabulary consistently. Names must reveal the actual responsibility in context. General words such as data, items, process, and handle do not identify a domain responsibility by themselves. Short loop variables are acceptable where their referent is immediate; ambiguous naming across a larger operation is not.

When an operation produces several distinct domain results with interchangeable representations, return them through named fields or established semantic types. Do not encode their identities solely through return order, parallel arrays, or undocumented positions. A caller must be able to identify the result it is using without consulting a separate ordering convention. Ordinary language conventions such as Go's `(value, error)` and `(value, ok)` remain appropriate; they already identify different roles. Preserve a required external signature at its boundary and express domain meaning clearly inside it.

Make interfaces state what callers provide, what they receive, and what can fail. Creation, persistence, refresh, deletion, and other consequential state changes must be apparent from the operation's contract and the flow invoking it. Do not hide them behind a name promising only lookup, inspection, or calculation. Renaming a tangled operation does not repair unclear ownership; make the responsibility coherent as well.

Use established language idioms and the project's mandatory conventions. Do not introduce clever compression, unexplained special cases, unnecessary type machinery, or representation choices justified only by a general preference. Choose representations according to their actual identity, ownership, lifetime, and mutation requirements.

Judge local readability and system coherence together. A readable caller should be able to rely on an understandable contract; it should not need a private copy of the implementation to remain understandable.

Common operations must be straightforward to call and invalid use difficult. A reader must not need a hidden sequence of calls, the agent's explanation, or a chain of forwarding wrappers to recover the responsibility being performed. Required domain complexity may need documentation; prose cannot compensate for avoidable confusion in the implementation. Follow `AGENTS.md` for comments and docstrings; durable rationale belongs in project documentation.

## 3. Ownership, encapsulation, and abstraction

### Give each responsibility an authoritative owner

Every business rule, fact, validation policy, transformation, and lifecycle decision needs a clear owner. Other components call that owner or derive from its contract. They do not maintain independent approximations of its behavior.

Encapsulation means the owner contains the rules and state transitions that make its responsibility correct. Expose the operation callers need. Keep responsibility-specific failure handling inside that boundary, so unrelated layers do not need to understand or repair it.

Encapsulation is complete only when callers can express intent without reproducing the owner's decision sequence. A helper that leaves callers responsible for the same selection rules, sequencing, exceptions, or invariants has extracted mechanics without fully owning the behavior. Move those decisions into their real owner; do not require each caller to assemble the responsibility from fragments.

When a behavior changes, identify the authoritative place where its meaning changes. Consumers may require coordinated updates; one owner does not imply that a cross-system change touches only one file.

### Keep shared knowledge and behavior DRY

Look for existing owners and implementations before adding code. Extend the existing owner and route equivalent entry points through it. Do not leave repeated business rules, parallel validation policy, or sibling implementations that can drift.

Consolidate the complete shared policy, including its ordering, tie-breaks, special cases, and failure behavior. Extracting one calculation or predicate does not satisfy DRY when callers still duplicate the surrounding rule. Inspect equivalent callers and confirm that they delegate the complete decision to its owner. Different input types, data sources, or transports do not justify copying a policy with the same meaning and change obligations.

Extract the real shared concept behind repetition. A useful abstraction makes callers more uniform, contains the shared rules, and reduces the combined complexity of the implementation and its callers. Similar text is evidence to investigate; it does not by itself establish that independent domain responsibilities belong together.

Sharing a low-level API does not establish a shared policy. A readiness probe and a management command may share authentication or encoding while retaining different lifecycle and failure semantics. Consolidate the common responsibility completely without combining distinct operations into an artificial general-purpose interface.

Follow the abstraction timing required by `AGENTS.md`: use working, tested concrete behavior or observed repetition as evidence; begin with an abstract shared contract only when it is established end to end. Neither a hypothetical future caller nor familiarity with a pattern justifies speculative machinery.

If genuine shared behavior remains duplicated, find the right boundary. If an extraction merely moves lines into a vague helper or introduces flags for unrelated cases, revisit the concept being extracted. Do not exchange duplication for an abstraction that obscures the domain.

### Require an abstraction to pass at both ends

Accept an abstraction only when all of the following hold:

- Its name and contract identify a real responsibility or invariant.
- Its implementation contains the decisions belonging to that responsibility.
- Its callers communicate domain intent without reconstructing its mechanics or supplying repeated policy fragments.
- The current behavior or demonstrated repetition establishes its need under `AGENTS.md`.
- It reduces the combined burden of understanding and changing the owner and its callers, without introducing speculative variation.

Reduced repetition alone is insufficient. A generic callback interface, configurable pipeline, or strategy factory fails this standard when it hides a small fixed operation behind more concepts than the operation requires. Do not add a framework to avoid writing a focused domain operation.

Do not satisfy the call-site requirement by hiding complexity in an unreadable helper. Review the owner and representative callers together. Likewise, do not make a caller locally convenient by duplicating its owner's policy.

### Split by responsibility

Prefer modules with meaningful, complete responsibilities and small, useful interfaces. Split an overloaded module along the concepts it mixes. Do not split a cohesive operation merely to reduce file length, or keep unrelated responsibilities together merely to reduce file count.

A wrapper earns its place by owning a real boundary, invariant, translation, or responsibility. A module that only forwards arguments without adding such a boundary usually adds navigation rather than understanding.

Reuse the existing implementation instead of creating an `Enhanced`, `New`, `V2`, `Public`, or `Internal` sibling of the same behavior. Express supported variation through the existing owner when the responsibility is the same; configuration must not become a collection of unrelated modes. Legitimately distinct responsibilities need explicit boundaries rather than disguised copies.

## 4. Contracts, consistency, and change

### Preserve meaning across boundaries

Read the current authoritative specification before changing a contract. Keep public declarations accurate and complete enough for consumers to build against. An implementation that cannot yet support declared behavior must reject it explicitly; do not hide the capability, invent its semantics, or return fabricated success.

Treat response shapes, error meaning, authorization, lifecycle, and operational cost as parts of the contract. A function becoming blocking, expensive, fallible, or remote changes its callers' obligations even when its signature stays the same. Revisit relevant callers for lock scope, thread or event-loop use, deadlines, cancellation, failure handling, and assumptions about completion.

For contracts spanning repositories or components, identify every relevant producer and consumer: declarations, generated clients, compilation, runtime bundles, execution paths, domain services, documentation, and frontend assumptions. Use one canonical shape and vocabulary. Each participant must implement the new contract or explicitly reject it. Report out-of-scope mismatches and their effect on completion.

### Evolve the existing system coherently

Use established shared clients, encoders, errors, validation, authentication, caching, and execution infrastructure. Related entry points should follow the same ownership and response conventions. Treat an unexplained exception as a design problem to resolve.

Consistency does not require reproducing a demonstrated defect. Correct it at the responsible boundary within the approved scope, update the affected consumers, and explain the reason. Do not introduce a parallel architecture or expand a task into an unrelated rewrite.

When requirements or instructions conflict, expose the conflict and the proposed interpretation. A specification's purpose can reveal an incorrect enumeration, but it does not grant permission to silently change the specification or override an approval boundary.

### Keep one current path unless coexistence is required

When an authorized change replaces a conceptually dead path, remove or explicitly reject that path and its obsolete aliases, formats, and implementations. Do not preserve it through hidden translation or fallback.

Compatibility needs a real consumer, retained data, or external obligation. Any temporary coexistence needs an owner, tests, a migration plan, and a cutoff. A deliberately ongoing compatibility obligation needs an explicit supported contract.

For a clean rebuild, a pre-launch system without retained data obligations, or a deployment the user has explicitly said will be discarded, use a hard cutoff. Do not invent backfills, upgrade paths, or historical-data protections. Improving an implementation does not itself require versioning it.

## 5. Complexity, reuse, and resources

Build the requested product with production foundations: real ownership, contracts, authorization, persistence, and failure semantics. Do not reframe product work as an MVP or validation exercise unless requested. Sequence scope through complete, coherent slices without introducing throwaway correctness or speculative future features.

Every significant mechanism must have a concrete reason to exist. For defensive mechanisms, name the failure prevented and the test that demonstrates protection. For an abstraction, name the responsibility it encapsulates or the understanding and change burden it removes.

Identify the source of each configuration decision. When one setting must agree across configuration files, command arguments, runtime objects, and other representations, resolve its value once and derive those representations from the owner. Repeating the same literal at each output does not establish a shared source. Extract values because they represent the same decision and must change together, rather than because their spelling happens to match.

Before introducing a dependency, layer, retry, cache, gate, configuration option, or supporting tool, establish:

- The current outcome, obligation, or failure it addresses.
- Why an existing or simpler mechanism is insufficient.
- Its continuing cost in complexity, maintenance, latency, storage, compute, money, and attention.
- Its removal condition if it is temporary.

Use this reasoning to choose the design. Record consequential decisions concisely; do not create a separate justification report for every function. Unclear purpose calls for investigation. Remove confirmed unnecessary machinery within the authorized scope rather than retaining it through speculation.

Reuse the project's existing implementation first. For common or standardized capabilities, follow `AGENTS.md`'s preference for maintained, stable, compatible packages over reinvention. Evaluate the actual contract, license, integration, and maintenance cost; a familiar package name is not sufficient evidence of suitability.

Supporting machinery must remain proportionate to the production outcome it protects. Test harnesses, evaluators, dashboards, migration helpers, audit records, and reports do not create value merely by looking rigorous. Keep one current tool where versions have no coexistence obligation. Improve it in place and remove obsolete machinery within scope.

Bound diagnostic work and retained artifacts to a current use. Prefer compact summaries, retain raw evidence when needed to reproduce a named failure, and clean temporary infrastructure up when its purpose ends. Spend resources on confidence in the product rather than repeated demonstrations that tools ran.

## 6. State, failure, and lifecycle integrity

### Keep authoritative state authoritative

Read or derive facts from their owner. Construct emitted data from that owner instead of maintaining fields that must stay manually synchronized. Do not mirror authoritative state for convenience.

Every retained field must have a current behavioral consumer, persistence obligation, or explicit external contract. Assigning a field, copying it into another internal record, or asserting its existence in a test does not establish a production use. Check readers as well as writers. Do not store a derived or remote-looking fact merely because it might be useful later. Within the changed responsibility, remove unused state and the record conversions or update machinery that exist only to maintain it.

A required cache, snapshot, or replica must remain explicitly derived, with defined freshness, invalidation, and reconciliation semantics. It must not become a competing authority. Prefer direct derivation when those additional obligations are unnecessary.

Each state owner must use one explicit update and publication model for a given piece of state. Apply that model across creation, update, removal, cleanup, and callbacks. Do not replace a collection on some paths while silently mutating the published collection on others. Building or modifying an unpublished working copy before publishing it is compatible with replacement; mutating an already published snapshot is not. Review in-place mutations as well as assignments when checking consistency.

Reject stale lifecycle work at the owner. In an in-process object lifecycle, retain and validate the identity of the instance that scheduled the work. Across processes, use the system's authoritative version or lifecycle token. Do not let a copied identifier stand in for proof that the work still belongs to the current lifecycle.

### Make invalid states and interleavings difficult to create

Prefer ownership and representation that make dangerous transitions impossible or harmless. Use immutable snapshots, replacement instead of shared mutation, and stable local references where they actually simplify the lifecycle.

These techniques do not by themselves establish concurrency safety. Publication, mutation, and access must satisfy the language and runtime's memory model. Use synchronization when required, at the smallest boundary that protects the actual invariant. Do not remove a necessary lock or atomic operation in the name of simplicity.

Describe any accepted residual interleaving and its observable consequence precisely. It is acceptable only if it preserves the required contract; apparent rarity or difficulty reproducing it is not a justification.

### Make failure and operational state truthful

A required dependency remains required. If its capability is unavailable, fail clearly. A fallback is acceptable only when deliberately part of the product contract, visible, and correct for the promise being made.

Distinguish unknown, unavailable, unsupported, empty, and successful results. Do not substitute `0`, `false`, an empty collection, or a success-shaped response merely to make a consumer compile or a test pass.

Represent execution honestly: inline, queued, running, completed, or failed. Identify who owns retries and reconciliation. When debugging, use the actual path, attempts, traces, and state transitions rather than assuming the implementation matches its intended design.

Platform identity, tenant scope, grants, invocation identifiers, audit correlation, and accounting context remain platform-owned. Reject or authoritatively replace conflicting caller-supplied fields. Developer input must not silently override trusted metadata.

### Finish the whole lifecycle

The component starting asynchronous work may report actual synchronous effects and a stable job reference. The component doing the work owns its final domain results and usage facts; the accounting owner reconciles and aggregates them. Do not invent final completion, storage, usage, or billing facts at the initiating boundary.

Do not block a synchronous response on unrelated background completion unless required by the contract. Expose missing job references as an observability limitation. Keep credentials and secret payloads out of completion metadata.

Deletion must define the fate of durable rows, stored assets, active deployments, queued work, credentials, retained audit or billing records, frontend state, and external provider resources. Stop obsolete work from recreating deleted state where applicable. Preserve required retention obligations and authorized scope; partial cleanup must not be reported as complete deletion.

## 7. Architecture where applicable

Apply these boundaries to the responsibilities that actually exist. They do not mandate a separate service, process, or framework for every row.

### Services, transports, and data

HTTP, gRPC, MCP, CLI, and other transports adapt to the same application operations. A transport parses its input, constructs the operation request, invokes the operation, and uses the shared response mechanism. It does not independently own business policy, resource dispatch, validation meaning, error semantics, or custom response envelopes for an existing shared contract.

Application operations own the application boundary: scope and access expectations, request validation policy, operation-level dispatch, error meaning, domain calls, and response shape. Domain owners retain their domain behavior and authoritative data.

| Responsibility | Owner when that layer exists |
| --- | --- |
| What developers may declare | Declaration or client layer |
| Local validation, compilation, packaging, deployment preparation | Build or deployment tooling |
| Authorization, scheduling, durable orchestration, reconciliation, quota, billing, durable audit | Control plane |
| Executed work and direct tool or vendor calls | Execution worker |
| Domain data, behavior, and final domain usage facts | Domain service |
| Presentation and user workflow | Frontend |

Be explicit about who decides, executes, retries, reconciles, audits, and accounts. Transport parsing and local checks do not replace authoritative enforcement.

Use the designated data model and store for the problem. Prefer operations native to the authoritative store over loading data into application code to reconstruct another data model. Graph operations belong in the graph system when the architecture has one; transactional state remains with its transactional owner. Do not introduce another store merely because a data-model label sounds suitable.

### React frontend conventions

React work follows `FRONTEND_REACT.md`. Read it before any React change; its conventions are mandatory there and are not relaxed by general appeals to simplicity.

## 8. Verification must support the claim

Functional correctness, architectural quality, and readability require appropriate evidence. A green test run does not establish all three. Review the design and read the changed code as a maintainer as well as exercising its behavior.

### Review structural quality explicitly

Apply the structural requirements while choosing boundaries and implementing each responsibility. Before declaring completion, read the finished change again without relying on the explanation or exploration that produced it. Inspect the owner and its real callers together. Do not defer the first meaningful readability review to the user.

The implementation must answer these questions through its structure:

| Check | Required evidence in the code |
| --- | --- |
| Domain flow | The principal operation exposes its meaningful stages, and each stage's implementation can be located directly. |
| Abstraction | Named responsibilities replace repeated mechanics; fixed choices do not require readers to reconstruct factories or callback chains. |
| Complete ownership | Equivalent consumers delegate the entire shared policy, including sequencing, ranking, exceptions, and failure meaning. |
| Semantic interfaces | Names, types, and result fields preserve the identities of inputs, outputs, and consequential side effects. |
| State discipline | Retained state has current consumers or explicit obligations, and every write follows the owner's update model. |
| Consistency | Related paths use the same contracts, configuration decisions, conventions, and applicable architectural boundaries. |
| Change locality | A concrete change to one responsibility reaches its owner and necessary consumers without repeating the decision in unrelated code. |
| Necessary complexity | Each added mechanism has a demonstrated role, and simpler existing mechanisms have not been bypassed without evidence. |

Reject the implementation while a required condition is unmet. Neither renaming a helper, moving a block to another file, adding an explanatory paragraph, nor reporting passing tests resolves a structural violation by itself. Inspect the resulting code to establish that the responsibility and comprehension burden actually changed.

Use the existing formatter, linter, or architectural checks for conventions they can enforce. Verify explicit restrictions that tooling does not cover as part of this same review. A rule being present in the instructions is not evidence that the output follows it. Do not invent numerical readability scores, arbitrary size quotas, source-shape tests, or a new review framework to substitute for understanding the implementation.

Keep this review bounded. Correct identified violations within scope and check the affected owner and callers again. A clean, evidence-supported result ends the pass; do not manufacture findings or repeat it without a changed implementation or unresolved issue. Record only consequential findings and their resolution in the ordinary completion report.

### Test the real behavior at the right boundary

- Test observable contracts through public interfaces. Avoid coupling tests to private fields, incidental call sequences, or the implementation being replaced.
- Keep setup small, each test's intent clear, and assertions meaningful. Use scenario names that state what is established. Establish the normal case and then its failure conditions.
- Test a rule at its owning boundary. Operation tests cover operation behavior; transport tests cover parsing and response translation. Exercise all promised interface methods where applicable.
- Mock real external or expensive boundaries and hard-to-reproduce failures. Use fast in-process implementations directly. Generate mocks with the project's standard tooling, and verify that fakes expose the same capabilities and semantics as production.
- Separate fast isolated tests from integration and end-to-end tests. Add realistic workloads where the claim involves scale, concurrency, accumulated state, long inputs, failure recovery, or multiple users and steps.

Do not present evidence from one layer as proof of another. A mock does not establish that the production integration works. Execution logs and hashes establish what ran; useful results and real behavior establish product quality. Include relevant side effects, missing behavior, failure handling, latency, and resource use in the evaluation.

### Demonstrate regression protection honestly

Run the focused tests and required builds specified by `AGENTS.md`. Run an appropriate regression slice when shared behavior changes. For a regression test guarding a specific defect, observe it fail for that defect and pass after the correction. Use the original failing implementation or a controlled local reintroduction, without overwriting unrelated work or affecting live systems.

Change expectations when the intended contract changes and explain why. Do not loosen assertions, accept obsolete shapes after a hard cutoff, fake unavailable production capabilities, or weaken production semantics to make tests green.

If an environment-dependent check cannot run, report it as missing evidence and identify what remains unverified. A skip is not a pass. Follow `AGENTS.md` for required gates; an unavailable mandatory gate remains a completion blocker.

### Review independently and proportionately

Substantive changes receive an independent adversarial pass aimed at refuting their concrete claims. Re-run relevant gates, read the whole change, and examine plausible failure scenarios. That pass must also evaluate the structural requirements above; a correctness-only review does not establish design quality. A supported clean review is valid; inventing defects is not rigor. If required independent verification is unavailable, report verification as blocked rather than describing self-review as independent.

Validate findings against current source, contracts, and reproducible behavior. Separate a real finding from the reviewer's proposed fix. Resolve disagreement with evidence, and choose the repair that fits the owning responsibility. Reject fictitious compatibility mappings and unsupported semantics even when they would satisfy a review checklist.

Match review depth to risk. Security boundaries, billing, public contracts, irreversible data changes, and releases need stronger evidence. Use the lightest process capable of exposing the named failure. Do not repeat a stable review without changed code, new evidence, or a concrete unresolved risk.

## 9. Working discipline and durable knowledge

Before designing a change, establish the requested outcome, current implementation, authoritative contracts, responsible owners, real consumers, and relevant failure modes. Read the surrounding code and project memory as required. Verify remembered paths, versions, and system state against current evidence.

Choose the coherent system shape before accumulating patches. Address the class of failure at its owner. If a repair requires a second owner, a special-case alias, a hidden fallback, or a weaker test, reconsider the design before building more machinery around it.

Before committing to a substantial shape, identify the domain steps, the owner of each policy, the concepts represented by its inputs and outputs, and the existing consumers it must serve. Use the required implementation plan for this reasoning; do not add a separate planning ceremony. Revise a proposed shape that already requires callers to reproduce policy, obscure fixed cases, or keep redundant state synchronized.

Use engineering judgment to choose among implementations that satisfy the standard. Explain consequential trade-offs through actual effects on comprehension, ownership, drift, and continuing cost. Do not use a trade-off explanation to waive a mandatory requirement. Resolve ordinary implementation choices from evidence; escalate genuine instruction conflicts or scope changes under `AGENTS.md`.

Once implementation is approved, carry it through the authorized implementation, verification, and cleanup. Continue until the intended outcome is complete or a concrete external dependency, missing permission, or user pause blocks it. Report the exact blocker and completed work without presenting partial progress as completion.

Keep the full objective and accepted constraints active throughout the task. A new correction refines that standard; it does not replace earlier requirements with whichever concern was mentioned most recently.

When the user challenges a design, investigate the underlying concern about ownership, readability, consistency, hidden failure, or unnecessary complexity. Answer at the level requested: a structural review must examine how the code communicates and organizes its responsibilities, rather than drift into an unrelated bug audit or test report. Explain what the mechanism owns, when it runs, why it is needed, and how its structure meets the standard. Do not defend it merely because it exists, and do not turn a design question into an unsolicited patch.

Record consequential architecture and product decisions in concise project documentation with the decision, rationale, and what it supersedes. Keep the current account corrected in place. Follow `MEMORY_PROTOCOL.md` for daily history, user decisions, handoffs, and outstanding work; current architecture records and session history serve different purposes. Do not preserve every command, discarded experiment, or tool revision as a permanent ledger.

Match documentation to its audience. Public documentation explains what users configure, do, observe, and troubleshoot. Internal specifications explain internal contracts, coordination, and operational mechanisms. Use the existing document's vocabulary and style, and keep global guidance free of project-specific plans or links.

## 10. Completion means the whole engineering outcome

Declare completion only when:

- The requested behavior is implemented through its real boundaries, with required failure and lifecycle semantics.
- The changed code satisfies the explicit structural acceptance checks: readable domain flow, complete ownership, meaningful abstractions, semantic interfaces, consistent state and conventions, localized change, and necessary complexity.
- Every producer and consumer needed for the requested outcome satisfies the agreed contract.
- Required tests, builds, and reviews are complete and support the stated claims.
- Replaced paths and temporary machinery are addressed within scope, and necessary project records reflect the current result.
- Any requested release or deployment artifact has actually been produced and inspected under the required permissions.

A structural violation, missing required capability, consumer update, or verification gate leaves the affected outcome incomplete or blocked, including when the affected repository is outside the authorized scope. Reporting the problem does not satisfy the requirement. The agent cannot relabel a mandatory quality failure as a nonblocking limitation. Any departure requires the user's explicit authorization under the authority rules above; other limitations may accompany a completed result only when the promised outcome and required gates are satisfied.

For authorized release work, verify the requested remote artifact and report the relevant commit, version, image tags, and digest. A local build is not a published release. Git, pushes, tags, deployments, and external writes remain subject to `AGENTS.md` and the user's specific authorization.

Report what changed, why the structure is appropriate, how it was verified, and any material remaining limitation. Keep the report proportional to the work. The implementation itself must carry the clarity and coherence promised here.

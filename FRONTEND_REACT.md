# React Frontend Conventions

Read this file before React work, together with `CODING_PROFILE.md`, which it extends. `AGENTS.md` governs authority and gates; this file grants no permission.

These are explicit engineering conventions for applicable React work. General appeals to simplicity do not authorize silently discarding them. The frontend owns presentation and user workflow; authoritative business policy remains behind application operations.

**State and composition.** Derive state from its source during render, put imperative work in event handlers, and use `useSyncExternalStore` for external state. Do not use `useEffect` as the default state-management mechanism or to synchronize duplicate state. Keep data fetching and external state coordination in their owning data or state layer.

Each component calls exactly one imported composed view-model hook. That hook composes data hooks, URL state, local UI state, and selectors into the view's contract. Primitive hooks, query hooks, and additional hooks belong inside that composition, not alongside it in the component. Define hooks in the project's hook layer, never inline in a component or route file.

**Routes and screens.** Route files own route matching, search validation, guards, and loaders. A route's page component is layout composition using imported feature components and its imported view-model. It contains no data fetching, hook definitions, or business logic. Feature components own the screen's substantial UI. Page-data hooks wrap typed feature queries; view-model hooks combine that data with presentation state and selectors.

Do not create an empty route wrapper around one child. Point the route directly at the imported component unless a page component contributes actual layout or composition.

**Memoization.** Use `useMemo` for derived values and `useCallback` for functions passed as props, used in dependency arrays, or otherwise sensitive to identity. Keep this source convention; do not strip it under a general instruction to avoid optimization. Maintain correct dependencies and pure calculations. Memoization must not be necessary for correctness: React may discard cached values, and compiler behavior is not a substitute for sound state ownership.

**Complete shared components.** Encapsulate recurring UI patterns as purpose-built components owning their state, layout, keyboard behavior, focus, accessibility semantics, and loading, empty, error, and unauthorized states. Callers supply intent. Do not leave repeated JSX behind a thin hook wrapper, distort an unrelated generic component, or expose internals solely to accommodate an isolated caller.

**Typed data by feature.** Use the project's selected typed contract client and one standard query/cache layer. Organize field selections, query and mutation declarations, mappers, cache keys, invalidation, and data hooks by feature. Centralize schema generation or introspection and scalar mappings. One shared executor owns transport, credentials, security headers, typed errors, unauthorized behavior, retries, and other cross-cutting policy. Feature hooks use it; components do not fetch directly.

**Stable, honest view shapes.** Map transport contracts to view shapes in the feature data layer. Preserve the intended screen during backend changes; do not delete JSX simply to silence type errors. Hydrate optional fields with semantically valid defaults or explicit unavailable states. Unknown amounts are not zero; unsupported capabilities are not successful empty results. If the contract cannot support the intended screen, expose that mismatch and resolve it at its owner. Keep field translation and fallback business logic out of components.

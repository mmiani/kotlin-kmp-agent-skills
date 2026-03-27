---
name: kotlin-project-feature-implementation
description: Use when implementing or extending a KMP feature. Provides a pre-coding checklist, layer-by-layer rules, state pipeline design, and source-set discipline. Forward-looking only — not a review skill.
license: Apache-2.0
metadata:
  author: Mariano Miani
  version: "2.0.0"
---

# Kotlin Multiplatform Feature Implementation

Use this skill when **implementing** a new feature or extending an existing flow in a Kotlin Multiplatform project.

This skill is forward-looking only. It is not a review skill. It does not produce verdicts or issue severity ratings. For post-implementation or PR review, use `kotlin-project-architecture-review` instead.

## What this skill does

- Provides a pre-coding inspection checklist to read the codebase before writing anything
- States the architectural defaults to implement against
- Gives layer-by-layer implementation rules
- Defines the expected output format for an implementation plan

## Default architecture assumptions

Unless the project clearly does otherwise, implement features using:

- UI driven from immutable state
- user events flowing into a state holder
- repositories owning data access and coordination
- domain layer only when business logic is complex or reused across multiple state holders
- shared logic in `commonMain` only when valid for all declared targets
- platform code at the edges
- narrow module APIs and cohesive feature ownership
- tests alongside new logic, not deferred

---

## Step 0: Read before writing

Inspect the relevant existing feature before writing any code. Identify:

1. **Module boundaries** — which modules own the feature area, what their public APIs are
2. **Source-set placement** — what is in `commonMain` vs platform source sets; where new code belongs
3. **Route and navigation ownership** — where routes are defined, how new screens would integrate
4. **State-holder pattern in use** — ViewModel, presenter, state machine; what the existing contract looks like
5. **Repository and data-source abstractions** — existing interfaces, implementations, error model
6. **Domain layer presence and rationale** — does it exist, does it add value, should new logic go there
7. **Error-model conventions** — exceptions, result wrappers, sealed error types; what the project uses consistently
8. **Existing tests** — test locations, test doubles in use, test patterns established

Do not invent new structure if the codebase already has a valid one. Match conventions unless there is a clear reason not to, and state the reason in the implementation plan.

---

## Layer-by-layer implementation rules

### UI layer

- Render from immutable `UiState` — no ad hoc dependency access in composables
- No repository or data-source calls from UI code
- No business rules in composables
- No DTOs or persistence models in `UiState`
- No platform-specific APIs in shared composables
- Handle all reachable states explicitly: loading, success, empty, error, partial-data, retry
- Handle responsive layout needs at this layer, not in lower layers

### State holder

- Expose exactly one immutable observable state stream (e.g., `StateFlow<UiState>`, or equivalent observable in the project's chosen pattern)
- Separate one-time effects (navigation, snackbars, permission requests) from persistent `UiState` — do not conflate them
- Consume user actions/events as inputs; do not let UI coordinate work directly
- Coordinate lower layers (domain, data) — do not own data-layer logic inline
- Do not become a god object: if the state holder is growing large, check whether domain extraction or screen splitting is appropriate
- Keep lifecycle/platform wiring out of shared state-holder logic unless the project explicitly puts it there

**Pattern note:** On Android, ViewModel is the standard state holder and integrates with the lifecycle natively. On KMP targets without Android ViewModel (iOS standalone, desktop, web), use the equivalent presenter or state-machine pattern the project has established. The shape must remain the same — one observable state output, separate effects — regardless of the underlying mechanism. See `kotlin-project-state-management` for a full treatment of state-holder pattern options across KMP targets.

### Domain layer

Add a domain layer only when at least one of these is true:

- the business rule is reused by more than one state holder or flow
- the business rule is non-trivial and benefits from isolated testing
- extracting it makes the state holder meaningfully smaller and clearer

Do not add pass-through use cases to satisfy an architecture diagram. A use case that does nothing but forward a repository call is net-negative: it adds indirection with no value.

### Data layer

- Repositories expose domain-facing interfaces — not DTOs, not persistence schemas, not HTTP response shapes
- Repositories coordinate local and remote sources internally; callers do not see this coordination
- DTOs and persistence models live below the repository boundary and do not cross it upward
- Preserve the project's established error model (exceptions, `Result<T>`, sealed class) consistently across new repositories
- New data sources have a single, narrow responsibility

### Source sets

Before placing any new file in `commonMain`, confirm it is valid for all declared targets.

Decision order:
1. Does it compile and behave correctly on all targets with no platform-specific API? → `commonMain`
2. Is it valid for a platform family (e.g., all Apple targets)? → intermediate source set (e.g., `appleMain`, `iosMain`)
3. Does it genuinely differ per target? → platform source set with a shared `expect` declaration in `commonMain` if needed

Do not default to `expect`/`actual` before checking whether an interface + injected implementation would be simpler. See `kotlin-platform-kmp-bridges` for full bridge-choice guidance.

### Module boundaries

- Keep feature changes local to the owning module wherever possible
- Do not bypass module APIs for implementation convenience
- If a change requires touching many modules, treat that as a signal that module boundaries may need review — flag it in the plan
- New public module APIs should be as narrow as needed and no wider

### Navigation

- Follow the route model and entry-point patterns already established in the project
- Do not re-architect navigation as part of a feature implementation unless explicitly scoped
- Preserve back/up behavior that matches user expectations
- Model deep-link entry realistically: the back stack after entry should be coherent, not empty

### Tests

Write tests alongside implementation, not after. At minimum, test:

- state-holder transitions for the new flow (loading → success, loading → error, retry)
- business rules in new domain use cases, if any
- repository coordination logic, mappers, and error-handling paths
- navigation decision logic for any conditional routing

Use `kotlin.test` in shared test source sets. Use JUnit or platform-specific test infrastructure only in platform-specific test source sets.

---

## Recommended feature structure

When the project has no established feature structure, this shape is a reasonable default:

```
feature-<name>/
  presentation/
    <FeatureName>Screen.kt         # composable, stateless rendering
    <FeatureName>ViewModel.kt      # or presenter/state holder equivalent
    <FeatureName>UiState.kt
    <FeatureName>UiAction.kt       # user events
    <FeatureName>UiEffect.kt       # one-time effects (optional, if separate)
    <FeatureName>Route.kt          # route definition / nav entry point
  domain/                          # omit if no real domain logic
    <FeatureName>UseCase.kt
    <FeatureName>Model.kt
  data/
    <FeatureName>Repository.kt     # interface (contract)
    <FeatureName>RepositoryImpl.kt
    <FeatureName>RemoteSource.kt
    <FeatureName>LocalSource.kt
    <FeatureName>Dto.kt
    <FeatureName>Mapper.kt
  di/                              # omit if project does not use a DI framework
    <FeatureName>Module.kt
```

The folder shape is not the goal. Cohesive ownership and predictable placement are. Match existing feature structure in the codebase unless it is clearly broken.

---

## Required output format

When using this skill to guide implementation, produce:

1. **Pre-coding inspection summary**
   - modules and source sets affected
   - state-holder pattern in use
   - repository/data-source conventions observed
   - domain layer presence and rationale
   - error model in use
   - existing test patterns

2. **Data ownership decisions**
   - source of truth for each new data type
   - who owns writes, who owns reads
   - offline-first considerations if relevant

3. **State pipeline design**
   - `UiState` shape (fields, sealed sub-states if needed)
   - user actions/events
   - one-time effects
   - state transitions: loading → success, loading → error, retry, empty

4. **Layer plan**
   - which files to create or modify, in which source set, in which module
   - what stays in shared code vs platform code
   - domain layer: yes/no and why

5. **Implementation** — code changes

6. **Tests to add**
   - state-holder transition tests
   - domain use case tests (if applicable)
   - repository and mapper tests
   - which test source set each test lives in

7. **Risks and edge cases**
   - source-set correctness risks
   - navigation edge cases (back stack, deep-link entry)
   - error paths not covered by happy-path implementation
   - configuration or lifecycle edge cases

8. **Assumptions**
   - any architectural assumption made that is not directly verified in the codebase

---

## Anti-patterns to prevent

- Starting to write code before reading the existing structure
- Multiple writable sources of truth for the same data type
- Direct repository or data-source access from composables
- Pass-through use cases that add ceremony without isolation benefit
- DTOs or persistence models flowing into `UiState`
- Platform-specific APIs in `commonMain` source sets
- Feature logic embedded in `expect`/`actual` bridge implementations
- Treating loading, error, and empty states as afterthoughts
- Inconsistent error modeling relative to the rest of the codebase
- New module APIs wider than the feature needs

---

## References

- Android architecture recommendations — https://developer.android.com/topic/architecture/recommendations
- Android UI layer — https://developer.android.com/topic/architecture/ui-layer
- Android domain layer — https://developer.android.com/topic/architecture/domain-layer
- Android data layer — https://developer.android.com/topic/architecture/data-layer
- Android modularization — https://developer.android.com/topic/modularization
- Android navigation principles — https://developer.android.com/guide/navigation/principles
- Android configuration changes — https://developer.android.com/guide/topics/resources/runtime-changes
- Kotlin Multiplatform project structure — https://kotlinlang.org/docs/multiplatform/multiplatform-discover-project.html
- Compose Multiplatform — https://kotlinlang.org/docs/multiplatform/compose-multiplatform.html
- Navigation in Compose Multiplatform (Navigation 2, stable) — https://kotlinlang.org/docs/multiplatform/compose-navigation.html
- Navigation 3 in Compose Multiplatform (alpha as of mid-2025 — verify before adopting) — https://kotlinlang.org/docs/multiplatform/compose-navigation-3.html

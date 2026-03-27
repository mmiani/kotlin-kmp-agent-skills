---
name: kotlin-project-architecture-review
description: Use when reviewing KMP architecture, PRs, layer boundaries, state-holder design, Android entry-point discipline, shared-vs-platform placement, or long-term maintainability in Compose Multiplatform projects.
license: Apache-2.0
metadata:
  author: Mariano Miani
  version: "1.4.0"
---

# Kotlin Multiplatform Architecture Review

Use this skill to review architecture decisions, feature plans, pull requests, migrations, and refactors in a Kotlin Multiplatform project.

This skill is **review-only**. It produces a verdict, issues by dimension, severity ratings, and concrete recommendations. It does not produce implementation plans or step-by-step coding guidance. For forward-looking implementation guidance, use `kotlin-project-feature-implementation` instead. For state-holder pattern selection across KMP targets, see `kotlin-project-state-management`.

This skill is intentionally strict. Its purpose is to protect maintainability, correctness of shared code placement, clean boundaries between layers, realistic ownership of state and data, Android entry-point discipline, and long-term scalability.

## Primary review goals

The review should validate whether the proposal:

- preserves a clear single source of truth for important data
- follows unidirectional data flow
- uses a proper UI state production pipeline
- keeps Android components as platform entry points rather than business-logic containers
- places business logic in the right layer
- uses the domain layer only when it adds real value
- keeps repositories and data sources responsible for data ownership concerns
- uses source sets correctly in KMP
- preserves modular boundaries and avoids accidental coupling
- keeps platform-specific behavior at the edges
- remains testable and understandable as the codebase grows

Do not optimize for theoretical purity alone.
Optimize for maintainability, correctness, consistency, and architectural clarity.

---

## Official architecture defaults to review against

Unless the project has a strong, deliberate reason not to, prefer these defaults:

- Single source of truth for each important data type
- Unidirectional data flow
- UI driven from data models
- State holders for UI complexity
- ViewModels or equivalent state holders (presenters, state machines) exposing UI state and receiving user actions
- Coroutines and Flow for async work and observable state
- Clear separation of UI, domain, data, and platform integration
- Domain layer only when business logic is complex or reused
- Repositories as the main boundary for exposing and coordinating app data
- Android components treated as lifecycle-bound entry points, not as general-purpose business-logic containers

---

## Review dimensions

### 1. Single source of truth

Check whether every important piece of data has one clear owner.

Questions:
- What is the source of truth for this data?
- Is there more than one writable owner?
- Is UI holding durable app truth that should live lower?
- Is repository ownership explicit?
- In offline-first flows, is local persistence treated as source of truth when appropriate?

Flag as a concern when:
- multiple layers mutate the same data independently
- UI owns business-critical state beyond screen-local concerns
- network payloads are treated as truth where resilient local ownership is needed
- ownership is ambiguous

### 2. Unidirectional data flow

Review whether state and events move in one direction.

Expected shape:
- user events move upward into a state holder
- state is produced by state holders from data/domain inputs
- rendered UI consumes state
- updates come back as new state instead of ad hoc mutation

Check whether:
- UI renders from state instead of pulling dependencies directly
- state is not mutated from several unrelated places
- one-off events are not confused with long-lived UI state
- data changes propagate through a coherent pipeline

Flag as a concern when:
- composables call repositories directly
- business logic runs in rendering code
- state is changed from multiple layers without a clear owner
- navigation, snackbar, toast, or permission effects are mixed into persistent state without clear modeling

### 3. Android component entry-point discipline

Android components are entry points with distinct lifecycles, and Activities / Fragments primarily host UI.

Check whether:
- Activities and Fragments are treated as UI hosts and platform lifecycle boundaries
- Services are used only for real background/service responsibilities
- BroadcastReceivers are thin entry points that delegate quickly
- ContentProviders are used intentionally rather than as general architecture shortcuts
- platform entry points delegate to state holders, repositories, or other appropriate layers

Flag as a concern when:
- Activities or Fragments own business rules, repository coordination, or transport parsing
- Services are used as architecture dumping grounds
- BroadcastReceivers contain meaningful feature orchestration inline
- platform entry points become the effective source of truth

### 4. UI layer responsibilities

The UI layer should consume app data, render it, handle user interactions, and reflect event effects in UI state.

Check whether:
- UI is focused on rendering and interaction
- state holders sit between UI and lower layers
- UI models are shaped for rendering rather than mirroring transport models
- screen states include loading, success, empty, error, partial-data, and retry when needed

Flag as a concern when:
- UI owns repository/data-source orchestration
- DTOs reach the UI directly
- UI performs large business transformations inline
- UI state is incoherent or under-modeled

### 5. State-holder quality

Review whether state holders are doing the right work. The specific mechanism (ViewModel on Android, presenter on iOS standalone, state machine in shared KMP code) is less important than whether the contract is satisfied: one immutable observable state output, separate one-time effects, user actions as inputs, no business logic in the UI layer. See `kotlin-project-state-management` for a full treatment of the tradeoffs between state-holder approaches across KMP targets.

Check whether:
- there is a clear state-holder boundary such as ViewModel, presenter, or equivalent
- immutable state is exposed — not mutable state flows accessible from outside
- the state holder consumes user actions and produces UI state without being a two-way bridge
- business logic and UI rendering logic are not mixed inline
- state production is based on clear, traceable inputs and outputs
- one-time effects (navigation, snackbars, permission triggers) are modeled separately from persistent `UiState`

Flag as a concern when:
- no state holder exists despite meaningful screen complexity
- the state holder is a god object: rendering, data parsing, network calls, analytics, and navigation logic all inline
- mutable state is leaked broadly — e.g., `MutableStateFlow` exposed as public API
- UI state and one-time effects are conflated into the same stream
- state-holder pattern is inconsistent across similar screens without a stated reason

### 6. Domain layer usage

The domain layer is optional. It should exist when it reduces duplication or isolates meaningful business logic.

Check whether:
- domain use cases encapsulate complex or reusable business logic
- domain models stay independent of UI and transport concerns
- the domain layer adds clarity rather than indirection

Flag as a concern when:
- a domain layer exists but only forwards calls
- trivial pass-through use cases add ceremony with no isolation benefit
- domain code depends on framework, UI, or transport details
- reusable business logic is duplicated across state holders instead of extracted

### 7. Data layer responsibilities

Repositories should expose data, centralize changes, resolve conflicts across sources, abstract sources, and own data-related business rules.

Check whether:
- repositories expose app/domain-facing outputs
- repositories centralize writes and coordination
- multiple data sources are resolved in one place
- data sources remain implementation details where appropriate
- the rest of the app is insulated from transport and persistence specifics

Flag as a concern when:
- repositories merely mirror raw endpoints
- UI or state-holder coordinates local and remote data directly
- repository ownership is bypassed
- persistence and network details leak upward

### 8. Error handling architecture

Check whether:
- the project has a consistent failure model
- failures are surfaced deliberately
- repository/data errors are normalized when needed
- user-facing messages are derived at the presentation edge
- retryable and non-retryable failures are distinguishable when relevant

Flag as a concern when:
- strings are the effective error model
- each layer invents its own failure contract
- failures are swallowed silently
- transport messages are shown directly to users by default

### 9. Layering and separation of concerns

Expected layers:
- presentation / UI
- orchestration / state-holder
- optional domain
- data / repositories / services / persistence
- platform integration

Check whether:
- UI contains business rules
- domain depends on transport, UI, or platform details
- DTOs leak into domain or presentation
- repositories own data coordination
- platform concerns remain outside business logic

Flag as a concern when:
- one file mixes UI, networking, mapping, and business rules
- repository implementations live inside ViewModels
- domain models are actually transport models
- platform SDK types appear in shared business code

### 10. Source-set correctness in KMP

Check whether code in `commonMain` is truly valid for all declared targets.

Review:
- whether `commonMain` references platform APIs
- whether target-specific behavior is isolated
- whether source-set placement follows compilation reality rather than convenience
- whether the design would still work if more targets were added

Flag as a concern when:
- platform-specific APIs appear in shared code
- shared code assumes one platform’s lifecycle, resources, filesystem, or navigation model
- platform-only dependencies leak into common code

### 11. Shared vs platform-specific boundary quality

Check whether:
- the proposal shares the right things
- native concerns stay at the edges
- expect/actual is justified and small
- abstraction boundaries are minimal and clear

Flag as a concern when:
- platform-specific code leaks into business logic
- large expect/actual surfaces own feature logic
- native or vendor types spread through shared modules

### 12. Module boundaries and modularization quality

Check whether:
- each module has a clear purpose
- dependencies are intentional and minimal
- public APIs are narrow
- shared modules are truly shared and not dumping grounds
- feature ownership remains cohesive

Flag as a concern when:
- unrelated features depend on each other directly
- common/shared/core modules accumulate unrelated code
- visibility is broad for convenience
- granularity is either too coarse or too fragmented

### 13. Navigation and Android component interaction

Check whether:
- navigation ownership is clear
- route definitions are coherent
- start-destination behavior is understandable
- back behavior is realistic
- intent-driven entry points fit the navigation model
- Activities launched via explicit or implicit intents still delegate into the same architecture instead of bypassing it

Flag as a concern when:
- deep links or intent entries create architecture bypasses
- routes are brittle and stringly typed without structure
- navigation behavior depends on hidden assumptions

### 14. Manifest and exported-surface review

Android components must be visible to the system through the manifest, and manifest declarations define part of the app’s architectural surface.

Check whether:
- Activities, Services, and ContentProviders that should run are declared appropriately
- BroadcastReceivers are declared or dynamically registered intentionally
- intent filters are added only where they represent real external entry points
- manifest exposure matches the intended architecture surface

Flag as a concern when:
- components rely on accidental manifest exposure
- architectural entry points are unclear from declarations
- too many components are externally reachable without a clear reason
- manifest declarations and actual ownership boundaries drift apart

### 15. Responsiveness and configuration resilience

Check whether:
- UI state production is resilient to configuration changes
- layout adaptation is structured rather than bolted on
- state holders remain valid across lifecycle/configuration changes where appropriate
- layout assumptions are not hard-coded to one form factor

Flag as a concern when:
- configuration change handling is fragile
- adaptive layouts require rewriting feature logic
- state is tied too tightly to one screen shape

### 16. Resources and presentation boundaries

Check whether:
- localization and resources stay in presentation/platform concerns where appropriate
- shared business logic does not hard-code values that belong in resources
- locale-sensitive formatting (dates, numbers, currency) happens at the presentation edge, not in repositories or use cases
- the feature can evolve to alternative resources, configurations, or locales without major refactoring

Flag as a concern when:
- user-facing strings are assembled or formatted in repositories or use cases
- locale-sensitive logic runs inside business rules rather than at the presentation edge
- presentation constants or hard-coded display values are buried in shared data or domain layers
- the design assumes a single language, locale, density, or configuration
- a resource change (new locale, new theme, new form factor) would require modifying non-presentation code

### 17. Testability as an architectural property

Check whether:
- business rules are isolated for unit tests
- mappers are pure and testable
- repositories can be tested with fakes/mocks
- state-holder logic can be tested without rendering UI
- key failure paths are testable
- platform entry points are thin enough that most logic is testable outside them

Flag as a concern when:
- critical logic is trapped in Activities, Services, Receivers, or other platform entry points
- key flows can only be tested end to end
- architecture relies on hidden global state

---

## Severity framework

### High severity
Likely to cause architectural drift or correctness problems.

Examples:
- no single source of truth
- business logic embedded in Activities or Fragments
- repositories bypassed by UI/state-holder code
- platform APIs in commonMain
- major module-boundary violations
- manifest/exported entry points that bypass intended architecture

### Medium severity
Workable, but likely to create maintenance cost.

Examples:
- weak domain-layer justification
- oversized state holder
- DTO leakage into presentation
- unclear module ownership
- inconsistent failure modeling
- entry-point delegation that is only partially consistent

### Low severity
Structurally acceptable but worth improving.

Examples:
- naming obscures ownership
- package split could be clearer
- tests miss important transitions
- route modeling could be more explicit

---

## Required output format

When performing the review, respond with:

1. Verdict
   - good fit
   - acceptable with revisions
   - poor fit

2. Architecture summary
   - what the proposal is doing
   - which layers, modules, source sets, and Android entry points it affects

3. What is structurally sound
   - concrete strengths only

4. Issues by review dimension
   - SSOT
   - UDF
   - Android component boundaries
   - UI layer
   - state-holder quality
   - domain-layer usage
   - data-layer design
   - source sets
   - modularization
   - navigation / intents / manifest surface
   - responsiveness/resources
   - testability
   - other relevant sections

5. Severity for each issue
   - high / medium / low

6. Concrete recommendations
   - exact structural changes
   - better layer placement
   - better component delegation
   - better module/source-set placement
   - better domain/data ownership where needed

7. Suggested target structure
   - proposed module/package/source-set / entry-point layout if useful

8. Open risks
   - migration cost
   - rollout concerns
   - backward-compatibility concerns

---

## Tone

Be direct and practical.
Do not give vague praise.
If the proposal is weak, say so clearly and explain why.

---

## Anti-patterns to flag aggressively

- no clear single source of truth
- bidirectional or ad hoc state mutation
- business logic in composables, Activities, or Fragments
- DTO-driven UI
- state-holder-free complex screens
- meaningless pass-through domain layer
- repositories bypassed by upper layers
- transport details leaking upward
- platform-specific APIs in commonMain
- modules with unclear purpose
- manifest or intent-filter surface that does not match the intended architecture
- hidden failure handling
- architecture that is only testable through large integration paths

---

## References

- Android app architecture: https://developer.android.com/topic/architecture
- Android architecture recommendations: https://developer.android.com/topic/architecture/recommendations
- Android UI layer: https://developer.android.com/topic/architecture/ui-layer
- Android domain layer: https://developer.android.com/topic/architecture/domain-layer
- Android data layer: https://developer.android.com/topic/architecture/data-layer
- Android application fundamentals: https://developer.android.com/guide/components/fundamentals
- Kotlin Multiplatform project structure: https://kotlinlang.org/docs/multiplatform/multiplatform-discover-project.html
- Kotlin Multiplatform hierarchy: https://kotlinlang.org/docs/multiplatform/multiplatform-hierarchy.html
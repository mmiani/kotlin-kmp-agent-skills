<div align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/7/74/Kotlin_Icon.png" alt="Kotlin" width="80" />

# Kotlin KMP Agent Skills

**A public catalog of AI agent skills for Kotlin Multiplatform projects.**

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/skills-12-brightgreen.svg)](#skills)
[![Kotlin Multiplatform](https://img.shields.io/badge/Kotlin-Multiplatform-7F52FF?logo=kotlin&logoColor=white)](https://kotlinlang.org/docs/multiplatform/multiplatform-discover-project.html)
[![Compose Multiplatform](https://img.shields.io/badge/Compose-Multiplatform-4285F4?logo=jetpackcompose&logoColor=white)](https://kotlinlang.org/docs/multiplatform/compose-multiplatform.html)

These skills are intentionally opinionated and grounded in official Android, Kotlin Multiplatform, and Gradle guidance. Built for reusable public use — not for one private codebase.

</div>

---

## Skills

### 🏗️ Architecture & Implementation

| Skill | What it does |
|---|---|
| [`kotlin-project-architecture-review`](skills/kotlin-project-architecture-review/SKILL.md) | Reviews KMP architecture, PRs, and layer boundaries. Produces a verdict, issue list, and concrete recommendations. |
| [`kotlin-project-feature-implementation`](skills/kotlin-project-feature-implementation/SKILL.md) | Guides feature implementation with a pre-coding checklist, layer-by-layer rules, and state pipeline design. Forward-looking only — not a review skill. |
| [`kotlin-project-modularization`](skills/kotlin-project-modularization/SKILL.md) | Reviews or designs module boundaries, dependency direction, visibility control, and granularity. |
| [`kotlin-project-state-management`](skills/kotlin-project-state-management/SKILL.md) | Covers state-holder pattern selection across KMP targets — ViewModel, shared presenter, MVI — including effect handling and `UiState` modeling. |

### 🎨 UI & Navigation

| Skill | What it does |
|---|---|
| [`kotlin-ui-compose-multiplatform`](skills/kotlin-ui-compose-multiplatform/SKILL.md) | Reviews shared Compose UI — state-driven architecture, composable decomposition, layout/modifier discipline, and previewability. |
| [`kotlin-ui-adaptive-resources`](skills/kotlin-ui-adaptive-resources/SKILL.md) | Reviews adaptive UI strategy — window-size classes, canonical layouts, navigation adaptation, and multi-window support. |
| [`kotlin-navigation-compose-multiplatform`](skills/kotlin-navigation-compose-multiplatform/SKILL.md) | Reviews Compose Multiplatform navigation — route modeling, back stack ownership, NavOptions, deep links, and browser URL binding. |

### 🔌 Platform Boundaries

| Skill | What it does |
|---|---|
| [`kotlin-platform-kmp-bridges`](skills/kotlin-platform-kmp-bridges/SKILL.md) | Reviews platform-specific integrations — source-set placement, hierarchical sharing, `expect`/`actual` usage, and entry-point wiring. |
| [`kotlin-platform-app-links-and-deep-links`](skills/kotlin-platform-app-links-and-deep-links/SKILL.md) | Reviews Android App Links and deep links — intent-filter design, host verification, `assetlinks.json`, and manifest scope. |

### 🗄️ Data, Testing & Build

| Skill | What it does |
|---|---|
| [`kotlin-data-kmp-data-layer`](skills/kotlin-data-kmp-data-layer/SKILL.md) | Reviews KMP data layers — repositories, data sources, source-of-truth design, conflict resolution, and error handling. |
| [`kotlin-testing-kmp`](skills/kotlin-testing-kmp/SKILL.md) | Reviews test strategy — `kotlin.test`, unit tests, instrumented tests, Compose UI tests, test doubles, and screenshot testing. |
| [`kotlin-build-kmp-gradle-governance`](skills/kotlin-build-kmp-gradle-governance/SKILL.md) | Reviews Gradle build structure — shared build logic, convention plugins, version catalogs, and source-set configuration. |

---

## Install

Copy one or more skill folders into your project's agent skills directory.

```bash
# Single skill
cp -r skills/kotlin-project-architecture-review .claude/skills/

# All skills at once
cp -r skills/* .claude/skills/
```

> The path `.claude/skills/` is an example. The exact directory depends on the agent framework you're using.

---

## What these skills address

KMP projects tend to drift in predictable ways. This catalog focuses on where that drift happens most:

| Area | Common drift |
|---|---|
| **Source sets** | Platform code in `commonMain`; shared code that assumes one platform |
| **State ownership** | Multiple writable sources of truth; effects modeled as persistent state |
| **Data layer** | Repositories that only mirror endpoints; DTOs leaking into UI |
| **Navigation** | `NavController` passed deep into composables; stringly-typed routes |
| **Platform bridges** | `expect`/`actual` overused where interfaces would be simpler |
| **Deep links** | Unverified App Links; `assetlinks.json` misconfigured on one of several hosts |
| **Adaptive UI** | Phone layout stretched to tablet; no window-size-class strategy |
| **Testing** | Business logic only tested through UI; `kotlin.test` absent from shared source sets |
| **Modularization** | `common`/`core` modules as dumping grounds; cyclic dependencies |
| **Build** | Repeated Gradle config; no convention plugins; platform deps in common source sets |

---

## Design principles

These skills assume the following defaults unless a codebase has a strong reason to differ:

- **Layered architecture** — UI, domain (optional), data, platform
- **Single source of truth** — one clear owner per data type
- **Unidirectional data flow** — events up, state down
- **Immutable UI state** — exposed from state holders, rendered by UI
- **Effect separation** — one-time effects distinct from persistent `UiState`
- **Repository boundaries** — data sources hidden behind repositories
- **Source-set discipline** — shared code as high as valid, platform code at the edges
- **Minimal `expect`/`actual`** — used narrowly; interfaces preferred for complex abstractions
- **Test pyramid** — most confidence from lower-level tests, not UI/instrumented tests
- **Centralized build governance** — convention plugins, version catalogs, explicit repositories

---

## Naming convention

All skills follow:

```
kotlin-<category>-<functional-name>
```

Available categories:

| Category | Purpose |
|---|---|
| `project` | Cross-cutting KMP project-level concerns (architecture, modularization, state management) |
| `ui` | Compose UI, layout, adaptive behavior |
| `navigation` | Navigation structure, routes, deep links |
| `data` | Repositories, data sources, source-of-truth |
| `platform` | Platform bridges, `expect`/`actual`, Android-specific platform concerns |
| `testing` | Test strategy, test layers, test tooling |
| `build` | Gradle structure, convention plugins, version catalogs |
| `architecture` | Standalone architecture-pattern skills (rare) |

---

## Scope

This repository is for **public, reusable** skills only.

**Not included:**
- Secrets or private infrastructure details
- Internal hostnames or proprietary workflows
- Project-specific assumptions presented as universal KMP defaults

**Not assumed as universal baseline** (can be layered on top in project-specific variants):
- A specific DI framework
- A specific database library
- A specific CI provider
- A fixed AGP / Gradle / Kotlin compatibility matrix
- Stack-specific ProGuard / R8 rules

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## References

<details>
<summary>Android architecture</summary>

- [Android Architecture](https://developer.android.com/topic/architecture)
- [Architecture Recommendations](https://developer.android.com/topic/architecture/recommendations)
- [UI Layer](https://developer.android.com/topic/architecture/ui-layer)
- [Domain Layer](https://developer.android.com/topic/architecture/domain-layer)
- [Data Layer](https://developer.android.com/topic/architecture/data-layer)
- [Application Fundamentals](https://developer.android.com/guide/components/fundamentals)

</details>

<details>
<summary>Modularization and build</summary>

- [Guide to Android app modularization](https://developer.android.com/topic/modularization)
- [Common modularization patterns](https://developer.android.com/topic/modularization/patterns)
- [Gradle Convention Plugins](https://docs.gradle.org/current/userguide/implementing_gradle_plugins_convention.html)
- [Gradle Version Catalogs](https://docs.gradle.org/current/userguide/version_catalogs.html)
- [Android Kotlin Multiplatform Plugin](https://developer.android.com/kotlin/multiplatform/plugin)

</details>

<details>
<summary>Navigation and deep links</summary>

- [Navigation Principles](https://developer.android.com/guide/navigation/principles)
- [Navigate to a destination](https://developer.android.com/guide/navigation/use-graph/navigate)
- [Navigate with options](https://developer.android.com/guide/navigation/use-graph/navoptions)
- [Pass data between destinations](https://developer.android.com/guide/navigation/use-graph/pass-data)
- [Animate transitions](https://developer.android.com/guide/navigation/use-graph/animate-transitions)
- [Conditional navigation](https://developer.android.com/guide/navigation/use-graph/conditional)
- [Back stack](https://developer.android.com/guide/navigation/backstack)
- [About App Links](https://developer.android.com/training/app-links/about)
- [Create deep links](https://developer.android.com/training/app-links/create-deeplinks)
- [Add App Links](https://developer.android.com/training/app-links/add-applinks)
- [Configure website associations](https://developer.android.com/training/app-links/configure-assetlinks)

</details>

<details>
<summary>Adaptive UI and layout</summary>

- [Adaptive layouts in Compose](https://developer.android.com/develop/ui/compose/layouts/adaptive)
- [Support different display sizes](https://developer.android.com/develop/ui/compose/layouts/adaptive/support-different-display-sizes)
- [Window size classes](https://developer.android.com/develop/ui/compose/layouts/adaptive/use-window-size-classes)
- [Multi-window mode](https://developer.android.com/develop/ui/compose/layouts/adaptive/support-multi-window-mode)
- [Build adaptive navigation](https://developer.android.com/develop/ui/compose/layouts/adaptive/build-adaptive-navigation)
- [Canonical layouts](https://developer.android.com/develop/ui/compose/layouts/adaptive/canonical-layouts)
- [List-detail layout](https://developer.android.com/develop/ui/compose/layouts/adaptive/list-detail)
- [Supporting pane layout](https://developer.android.com/develop/ui/compose/layouts/adaptive/build-a-supporting-pane-layout)

</details>

<details>
<summary>Testing</summary>

- [Testing fundamentals](https://developer.android.com/training/testing/fundamentals)
- [What to test](https://developer.android.com/training/testing/fundamentals/what-to-test)
- [Test doubles](https://developer.android.com/training/testing/fundamentals/test-doubles)
- [Testing strategies](https://developer.android.com/training/testing/fundamentals/strategies)
- [Local unit tests](https://developer.android.com/training/testing/local-tests#location)
- [Robolectric](https://developer.android.com/training/testing/local-tests/robolectric)
- [Instrumented tests](https://developer.android.com/training/testing/instrumented-tests)
- [Screenshot testing](https://developer.android.com/training/testing/ui-tests/screenshot)

</details>

<details>
<summary>Kotlin Multiplatform and Compose Multiplatform</summary>

- [KMP project structure](https://kotlinlang.org/docs/multiplatform/multiplatform-discover-project.html)
- [KMP hierarchy](https://kotlinlang.org/docs/multiplatform/multiplatform-hierarchy.html)
- [Share code on similar platforms](https://kotlinlang.org/docs/multiplatform/multiplatform-share-on-platforms.html)
- [Expected and actual declarations](https://kotlinlang.org/docs/multiplatform/multiplatform-expect-actual.html)
- [Use platform-specific APIs](https://kotlinlang.org/docs/multiplatform/multiplatform-connect-to-apis.html)
- [Compose Multiplatform](https://kotlinlang.org/docs/multiplatform/compose-multiplatform.html)
- [Compose adaptive layouts](https://kotlinlang.org/docs/multiplatform/compose-adaptive-layouts.html)
- [Compose previews](https://kotlinlang.org/docs/multiplatform/compose-previews.html)
- [Navigation in Compose Multiplatform](https://kotlinlang.org/docs/multiplatform/compose-navigation.html)
- [Deep links in Compose navigation](https://kotlinlang.org/docs/multiplatform/compose-navigation-deep-links.html)
- [Navigation 3 (alpha)](https://kotlinlang.org/docs/multiplatform/compose-navigation-3.html)
- [Compose Multiplatform testing](https://kotlinlang.org/docs/multiplatform/compose-test.html)
- [KMP DSL reference](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-dsl-reference.html)

</details>

---

<div align="center">

Apache-2.0 · Built for [Claude](https://claude.ai) and compatible AI coding agents

</div>

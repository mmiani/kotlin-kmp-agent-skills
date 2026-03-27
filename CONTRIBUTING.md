# Contributing

Thanks for contributing.

## What belongs here

This repository is for public, reusable AI agent skills related to Kotlin Multiplatform.

A good submission should be:

- focused on a clear use case
- reusable across multiple KMP codebases
- specific enough to change implementation quality
- grounded in sound architecture and platform boundaries
- safe to publish publicly

## Required structure

Each skill must live in its own folder under `skills/` and include `SKILL.md`.

Example:

```text
skills/
  kotlin-project-architecture-review/
    SKILL.md
```

## Required frontmatter

Each `SKILL.md` must begin with YAML frontmatter and include at least:

- `name` — must match the folder name exactly
- `description` — one sentence stating when the skill should be used; this is the trigger text read by agents
- `license` — must be `Apache-2.0` for public submissions
- `metadata.author`
- `metadata.version` — use semver, e.g. `"1.0.0"`

## Naming

All skills must follow:

`kotlin-<category>-<functional-name>`

Use a category listed in `CATEGORIES`. If your skill does not fit any existing category, propose a new category entry in the same PR and explain why the existing ones do not cover it.

**Category guidance:**
- `project` — cross-cutting KMP project-level concerns (architecture, modularization, state management, feature implementation patterns)
- `ui` — Compose UI, layout, adaptive behavior
- `navigation` — navigation structure, routes, deep links in shared nav code
- `data` — repositories, data sources, source-of-truth design
- `platform` — platform bridges, expect/actual, Android-specific platform concerns
- `testing` — test strategy, test layers, test tooling
- `build` — Gradle structure, convention plugins, version catalogs
- `architecture` — reserved for standalone architecture-pattern skills not fitting `project` (rare)

## Writing guidance

A strong skill should:

- state clearly when it should be used (the `description` frontmatter field is the primary trigger text, not the body)
- define the architectural defaults it assumes
- give explicit decision criteria for what to flag and why
- separate high-severity from lower-severity issues with concrete examples
- include anti-patterns with enough specificity to be actionable
- include references when official guidance materially informs the skill

A skill should **not**:

- present one opinionated stack choice (a DI framework, a database library) as a universal KMP default
- embed fast-changing version numbers as timeless structural rules
- include anything that assumes a specific private codebase or internal infrastructure
- be so generic that it gives no better guidance than reading the official docs directly

## Public safety rules

Do not include:

- secrets
- private hostnames
- internal endpoints
- production identifiers
- customer data
- private logs
- unpublished proprietary architecture details

## Pull requests

Please explain:

- what the skill is for
- how it differs from existing skills
- what category it belongs to
- which official or primary references informed it, if relevant


## Build guidance scope

Build-related skills must distinguish between:

- official structural guidance from Android/Gradle/Kotlin docs
- stack-specific examples that are optional
- time-sensitive version compatibility advice that should not be treated as timeless defaults

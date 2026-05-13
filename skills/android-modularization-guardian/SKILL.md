---
name: android-modularization-guardian
description: Use when reviewing Android project modularization — Gradle module graph, feature/core/app module responsibilities, dependency direction violations (feature depending on feature impl, core depending on feature), granularity decisions, and per-module documentation requirements.
---

# Android Modularization Guardian

## Role

Specialist in Android project modularization. Reviews enforce the dependency graph, module responsibilities, and granularity decisions based on Google's Now in Android modularization strategy.

This skill covers **module structure and inter-module dependencies only**. For code architecture inside modules (ViewModel, UseCase, layer boundaries), use `android-architecture-guardian`.

---

## Enforced Dependency Direction

```
:app  →  :feature:x:impl  →  :feature:x:api
                          →  :core:*
         :feature:x:api   →  (nothing; leaf)
:core:*  →  :core:*  (only lower-level cores)
```

No direction reversals. No cycles. Violations are critical.

---

## Module Types

### `:app`

**Owns:**
- `MainActivity` and the single `NavHost`.
- `Application` subclass.
- App-level dependency injection setup (e.g., Hilt `@HiltAndroidApp`).
- Top-level navigation scaffolding.

**Rules:**
- May depend on any `:feature:*:impl` and any `:core:*`.
- Must not contain feature business logic, screens, or ViewModels beyond routing glue.
- Must not be depended upon by any other module.

---

### `:feature:x:api`

**Owns:**
- Navigation contracts: route strings, `NavKeys`, deep-link patterns.
- Public interfaces that other features need to discover this feature (e.g., `FeatureEntryPoint`).

**Rules:**
- Must not depend on other `:feature:*` modules (api or impl).
- Must not contain screens, ViewModels, or business logic.
- Kept deliberately thin — a single `routes.kt` and a navigation entry class is the norm.

### `:feature:x:impl`

**Owns:**
- All Composable screens for feature `x`.
- All ViewModels for feature `x`.
- Feature-specific DI modules.
- Feature-level navigation graph wiring.

**Rules:**
- May depend on its own `:feature:x:api`.
- May depend on other features' **api** modules (for navigation cross-links). Never on their **impl**.
- May depend on any `:core:*` module.
- Must not be depended upon by `:core:*` or other `:feature:*:impl` modules.

### Flag these violations

| Violation | Severity |
|---|---|
| `:feature:a:impl` depending on `:feature:b:impl` | Critical |
| `:feature:x:api` depending on any `:feature:*` | High |
| `:feature:x:impl` containing a `NavHost` that belongs in `:app` | Medium |
| Business logic (UseCase, Repository) inside `:feature:*:api` | High |

---

### `:core:*` Modules

Standard core modules and their responsibilities:

| Module | Contains |
|---|---|
| `:core:model` | Domain models shared across features |
| `:core:data` | Repository implementations, data sources |
| `:core:network` | Retrofit setup, interceptors, network models |
| `:core:database` | Room database, DAOs, entity classes |
| `:core:ui` | Shared Composables, extensions, Compose utilities |
| `:core:designsystem` | Theme, typography, color tokens, icons |
| `:core:testing` | Test utilities, fakes, base test classes |

**Rules:**
- May depend on other `:core:*` modules, respecting a lower-level → higher-level direction (e.g., `:core:database` may not depend on `:core:data`).
- Must not depend on any `:feature:*` module.
- Must not depend on `:app`.

### Flag these violations

| Violation | Severity |
|---|---|
| `:core:*` depending on `:feature:*` | Critical |
| `:core:*` depending on `:app` | Critical |
| `:core:database` importing from `:core:data` or vice-versa in a direction that creates a cycle | High |
| Shared utility code duplicated across two or more feature modules instead of extracted to `:core:*` | High |

---

## Dependency Validation Checklist

When reviewing a PR that touches `build.gradle.kts` files:

1. **No upward dependencies** — lower modules must not reference higher modules.
2. **No sibling feature impl dependencies** — `:feature:a:impl` must never reference `:feature:b:impl`.
3. **No feature dependencies in core** — `:core:*` modules are framework, not feature-aware.
4. **API/impl split respected** — if a feature has both `:api` and `:impl`, cross-feature links go through `:api` only.
5. **No cycles** — run `./gradlew :app:dependencies` output or a dependency graph tool; flag any cycle.
6. **Shared code in `:core:`** — if the same class or utility appears in two features, recommend extraction.

---

## Granularity Guidance

### Extract a new module when

- **Ownership** is ambiguous — two teams edit the same module and step on each other.
- **Reuse** is real — the same code is duplicated in two or more features.
- **Build time** is measurably impacted — incremental builds degrade because a shared module is too large.
- **Isolation** is needed — a feature must be compiled and tested independently (e.g., a library feature shipped as an AAR).

### Do not extract a module when

- The extraction is speculative — "we might reuse this someday."
- The module would contain a single file or fewer than ~5 classes.
- The result would be a `:core:util` dumping ground with no clear responsibility.
- The team is small (≤3 engineers) and module overhead outweighs isolation benefits.

Over-modularization is a real cost: more `build.gradle.kts` files, more sync time, more cognitive overhead. Recommend against it explicitly when the trigger conditions are not met.

---

## Documentation Requirements

Every module must contain:

### `README.md`

Required sections:
1. **Purpose** — one paragraph: what this module owns and why it exists.
2. **Public API** — what other modules are expected to use from this one.
3. **Dependencies** — bullet list of direct Gradle dependencies, both project and library.
4. **Owner** — team or person responsible (optional for solo projects).

Flag any module without a `README.md` as a **Medium** severity finding.

### Dependency Graph

Each module's `README.md` should include or link to a dependency graph diagram. Acceptable formats:
- Mermaid `graph TD` block embedded in the README.
- A generated image from a tool like `gradle-dependency-graph-generator`.

Flag missing dependency graphs as **Low** severity.

---

## Output Format

For each violation found:

1. **Location** — `build.gradle.kts` file and the offending dependency line.
2. **Violation** — which rule is broken.
3. **Severity** — Critical / High / Medium / Low.
4. **Fix** — concrete change. Show the corrected `build.gradle.kts` snippet when applicable.

Group findings by module type: app → feature → core → documentation.

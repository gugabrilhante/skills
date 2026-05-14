---
name: android-create-new-feature
description: "Use when scaffolding a new Android feature from scratch. Detects project structure, asks clarifying questions only when ambiguous, then creates the feature following guardian-android-architecture, guardian-android-testability, guardian-android-testing, guardian-package-architecture, and guardian-android-modularization rules."
---

# Android Create New Feature

## Role

Android feature architect. Mission: scaffold a complete, production-ready new feature — with clean architecture, testable code, and tests — tailored to the project's existing structure.

---

## Phase 1 — Project Detection

Before writing any code, gather these facts:

| # | What to check | How |
|---|---|---|
| 1 | Module structure | Count dirs with a `build.gradle(.kts)` — single-module if only `:app`, multi-module otherwise |
| 2 | DI framework | `grep -r "hilt\|Hilt\|koin\|Koin" --include="*.gradle*" -l` |
| 3 | Package organization | Read existing feature packages — layer-first, feature-first, or hybrid |
| 4 | Build DSL | `.gradle` = Groovy, `.gradle.kts` = Kotlin |
| 5 | Navigation | Check for Jetpack Navigation or Compose Navigation |
| 6 | Existing conventions | Read one existing feature end-to-end (ViewModel, UseCase, Repository, Screen) |

### When to ask the user

Ask ONLY when detection is ambiguous:

- **Single-module project:** Ask whether the new feature should live inside the existing module (feature-first packages) or if this is the right moment to extract it into a new Gradle module.
- **Package organization is inconsistent:** Describe what was found and ask which style to follow for the new feature.

Do NOT ask if the project structure is unambiguous.

---

## Phase 2 — Placement

Apply the rules from `guardian-package-architecture` and (if multi-module) `guardian-android-modularization`.

### Single-module — feature-first packages

```
com.example.app/
  feature/
    <feature-name>/
      data/
      domain/
      presentation/
      di/
```

### Multi-module — new Gradle modules

Create:
- `:feature:<name>:api` — navigation contracts and public entry points only. No screens, ViewModels, or business logic.
- `:feature:<name>:impl` — all screens, ViewModels, DI modules, and feature-level navigation.

Wire `:feature:<name>:impl` as a dependency in `:app`.  
Wire `:feature:<name>:api` wherever cross-feature navigation is needed.  
Create a `README.md` in each new module (sections: Purpose, Public API, Dependencies).

---

## Phase 3 — Feature Scaffold

Apply the rules from `guardian-android-architecture` and `guardian-android-testability`.

### Domain

- `<FeatureName>Repository` interface — pure Kotlin, zero platform imports.
- One `UseCase` per distinct operation (fetch, create, delete, etc.). Each has a single `invoke`.
- Domain model — pure Kotlin data class, no Room or serialization annotations.

### Data

- `<FeatureName>RepositoryImpl` implementing the domain interface.
- Data source (local Room DAO, remote Retrofit service, or both — as needed).
- Explicit mapper functions: DTO/Entity → domain model. No implicit casting.
- DI binding of `RepositoryImpl` to `Repository`.

### Presentation

- `<FeatureName>UiState` — single immutable `data class`.
- `<FeatureName>UiEvent` — `sealed interface`.
- `<FeatureName>ViewModel`:
  - Receives `UiEvent`, translates to domain operations via `UseCase`.
  - Exposes `StateFlow<UiState>` — immutable to the View.
  - Must not hold `Context`, call DAOs directly, or expose `MutableStateFlow`.
  - Inject `CoroutineDispatcher` or a `DispatcherProvider` — no hardcoded `Dispatchers.*`.

### UI

- `<FeatureName>Screen` (stateful) — wires `ViewModel`, collects state via `collectAsStateWithLifecycle`.
- `<FeatureName>Content` (stateless) — receives state and callbacks only. Fully previewable.
- Navigation route registered in `:feature:<name>:api` (multi-module) or the app navigation graph (single-module).
- Every interactive element and key assertion node gets a `.testTag("tag")`.

### Testability rules (enforced from the start)

- All external dependencies injected via constructor — no `object` access inside logic.
- No hardcoded `Dispatchers` in business logic.
- No direct `System.*`, `Build.*`, `java.io.*`, or clock access in domain or presentation — wrap in injectable interfaces.
- No `object` singleton access inside UseCases or ViewModels.

---

## Phase 4 — Tests

Apply the rules from `guardian-android-testing`.

Write tests for every class created in Phase 3.

### Unit tests

- Each `UseCase`: happy path, error path, edge cases.
- `ViewModel`: each `UiEvent` type, loading / success / error state transitions, initial state.
- Mappers: each field mapped correctly, null handling.

### Integration tests (if Room is involved)

- DAO insert, query, update, delete using an in-memory Room database. No mocks.

### UI tests

- Happy path navigation to the feature screen.
- Primary user action (create / submit / confirm).
- Error state rendered correctly.
- Use `onNodeWithTag("tag")` — never `onNodeWithText()` as primary matcher.
- No `Thread.sleep()` — use `TestDispatcher`, `waitUntil {}`, or `IdlingResource`.

---

## Output

### Files Created
Full list with paths.

### DI Wiring
Where bindings were added (module file and method).

### Navigation
How the feature is reachable (route, NavGraph entry point).

### Tests Created
List of test files and what they cover.

### Manual Steps Required
Anything that cannot be automated (e.g., adding a bottom nav item, connecting to a real backend endpoint, adding a Codecov token).

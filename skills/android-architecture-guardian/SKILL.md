---
name: android-architecture-guardian
description: Use when reviewing Android code for architectural correctness: MVVM/MVI patterns, Unidirectional Data Flow, UI layer passivity, ViewModel responsibilities, UseCase design, Domain layer purity, and data model leakage between layers.
---

# Android Architecture Guardian

## Role

Specialist in application and code architecture for Android projects. Reviews enforce modern Android architecture principles as defined by Google's Architecture Guidelines and production experience.

This skill covers **code architecture only**. For module graph and dependency boundaries between Gradle modules, use `android-modularization-guardian`.

---

## Architectural Patterns

Accepted patterns:

- **MVVM** — ViewModel exposes state; View observes and renders.
- **MVI-like** — single state object, events flow up, state flows down.
- **Unidirectional Data Flow (UDF)** — state descends, events ascend, no cycles.
- **Reactive/passive UI** — View reacts to state; it never drives state.

Flag any design that inverts or bypasses these flows.

---

## UI Layer

### What the View must do

- Observe immutable UI state (`StateFlow`, `collectAsStateWithLifecycle`).
- Emit user events or actions to the ViewModel.
- Render state. Nothing else.

### What the View must not do

- Contain business rules.
- Sort, filter, validate, or transform data.
- Mutate shared state directly.
- Hold references to repositories, use cases, or data sources.

### Flag these violations

| Violation | Severity |
|---|---|
| Business logic inside a `@Composable` function | Critical |
| `if/when` branches driven by raw data instead of UI state | High |
| Sorting or filtering a list inside a Composable | High |
| Validation logic inside a screen | High |
| Calling a repository or use case from a Composable | Critical |
| Direct state mutation from UI (e.g., mutating a `MutableStateFlow` held by the View) | Critical |
| `ViewModel` created with `remember` or scoped inside a Composable | High |

---

## ViewModel Layer

### Responsibilities

- Hold and coordinate UI state via a single `StateFlow<UiState>`.
- Receive UI events and translate them into domain operations.
- Call UseCases; never call repositories or data sources directly.
- Expose immutable state; never expose mutable state to the View.
- Manage loading, error, and success states.

### What the ViewModel must not do

- Hold a `Context`, `Activity`, or `View` reference.
- Access Room DAOs directly.
- Make Retrofit or network calls directly.
- Perform data mapping between DTOs and domain models.
- Contain more than one unrelated responsibility.
- Exceed ~300 lines without a clear justification; flag for extraction.

### Flag these violations

| Violation | Severity |
|---|---|
| `context` or `activity` stored as a field | Critical |
| `@Dao` or `RoomDatabase` injected into ViewModel | Critical |
| Retrofit `Service` or `OkHttpClient` injected into ViewModel | Critical |
| Repository returning Room `Entity` consumed directly in ViewModel | High |
| ViewModel handling two unrelated features | High |
| ViewModel over 300 lines | Medium |
| Mutable state exposed as `MutableStateFlow` to the View | High |
| Side effects triggered from `init` without lifecycle awareness | Medium |

---

## Domain Layer

### Rules

- Pure Kotlin. Zero Android imports (`android.*`, `androidx.*`).
- Each UseCase does exactly **one** thing. Single public operator `fun invoke`.
- Repository contracts (interfaces) live in the domain layer; implementations live in data.
- Domain models are independent of Room entities, DTOs, and Retrofit responses.
- No framework annotations (`@Entity`, `@SerializedName`, etc.) on domain models.

### Flag these violations

| Violation | Severity |
|---|---|
| `android.*` or `androidx.*` import inside a domain class | Critical |
| UseCase with multiple `invoke` overloads doing different things | High |
| UseCase delegating to another UseCase without a clear orchestration reason | Medium |
| Repository implementation living in the domain layer | High |
| Domain model annotated with Room or Gson/Moshi annotations | High |
| Domain model containing UI formatting logic (e.g., `toDisplayString()` that calls Android resources) | Medium |

---

## Data Layer

### Rules

- Repository implementations translate DTOs/entities to domain models before returning.
- DTOs carry network/storage annotations (`@SerializedName`, `@ColumnInfo`). They must not leave the data layer.
- Room `@Entity` classes must not appear in the domain or UI layers.
- Mappers are explicit functions or classes; no implicit casting between layers.

### Flag these violations

| Violation | Severity |
|---|---|
| Room `@Entity` passed to a ViewModel or Composable | Critical |
| DTO (`@SerializedName`) used as a domain model | Critical |
| Repository method returning a DTO instead of a domain model | Critical |
| Mapper logic inlined inside a DAO query or Retrofit callback | Medium |
| Data layer class importing from a feature or UI module | High |

---

## Refactor Principles

- Prefer creating a new UseCase over adding a method to an existing one.
- If a ViewModel grows past ~300 lines, extract a second ViewModel or delegate to a helper class.
- If a Repository accumulates unrelated methods, split it by domain entity.
- Favour small, surgical refactors over full rewrites in a single PR.
- Do not use architecture patterns as an excuse to over-engineer simple screens; one small screen with a thin ViewModel is fine.

---

## Output Format

For each violation found:

1. **Location** — file and line range.
2. **Violation** — what rule is broken.
3. **Severity** — Critical / High / Medium.
4. **Fix** — concrete, minimal change. Show code when the fix is non-obvious.

Group findings by layer: UI → ViewModel → Domain → Data.

---
name: android-refactor-for-testability
description: "Use when improving code quality in an existing Android codebase: fixes architecture violations, eliminates testability blockers, and adds missing tests. Applies guardian-android-architecture, guardian-android-testability, and guardian-android-testing rules in sequence, with permission to refactor production code."
---

# Android Refactor for Testability

## Role

Code quality engineer for Android. You have permission to refactor production code. Mission: improve the health of existing code in three ordered phases — fix architecture violations, eliminate testability blockers, then write missing tests.

Run all three phases on the scope provided by the user. If no scope is given, target all files changed since the last commit (`git diff --name-only HEAD~1`).

---

## Phase 1 — Architecture Review and Fix

Apply the rules from `guardian-android-architecture`.

### What to check

- **UI layer:** Views must not contain business logic, sorting, filtering, or validation.
- **ViewModel:** Must not hold `Context`, call DAOs directly, make network calls, or expose `MutableStateFlow` to the View.
- **Domain layer:** Zero `android.*`, `androidx.*`, or `java.io.*` imports. Each `UseCase` has exactly one `invoke`. Repository contracts (interfaces) live here; implementations do not.
- **Data layer:** DTOs and Room `@Entity` classes must not cross into domain or UI. Mappers are explicit functions — no implicit casting between layers.

### What to do

Fix each violation found. Prefer surgical changes — do not rewrite code that is architecturally sound.

---

## Phase 2 — Testability Review and Refactor

Apply the rules from `guardian-android-testability`.

### What to check

- **Global state:** `object` singletons, `companion object` used as service locator, `System.*`, `Build.*`, `Locale.*`.
- **Non-determinism:** Direct access to current time, random, or UUID — not behind an interface.
- **Side effects:** `Log.*`, `Toast.*`, file I/O (`java.io.File`, `FileWriter`, `OutputStream`, etc.), network calls, analytics — anything not behind an injectable interface.
- **Threading:** Hardcoded `Dispatchers.IO` / `Dispatchers.Default`; `GlobalScope` usage.
- **Environment coupling:** `Context`, `SharedPreferences`, `PackageManager` inside domain or presentation logic.
- **Constructor injection:** Every external dependency must be injected via constructor — no service locator, no direct `object` access inside logic.

### What to do

For each blocker:
1. Search for an existing abstraction (`Logger`, `ClockProvider`, `DispatcherProvider`, `FileProvider`, etc.) — prefer reuse over novelty.
2. If no abstraction exists, create the minimal interface.
3. Inject it via constructor / DI.
4. Update DI module bindings (Hilt `@Provides`/`@Binds` or Koin `single { }`/`bind`).

---

## Phase 3 — Test Coverage

Apply the rules from `guardian-android-testing`.

After Phase 1 and 2, production code is architecturally sound and testable. Write missing tests.

### Scope

- **Domain:** Each UseCase — happy path, error path, edge cases.
- **Data:** Repositories, mappers, Flow emissions.
- **Presentation:** ViewModels — state transitions, each event type, loading / error / success.
- **Integration:** Room DAOs using an in-memory database. No mocks.
- **UI:** Key user journeys using Compose test APIs or Espresso. Use `.testTag()` on every interactive element. No `Thread.sleep()` — use `TestDispatcher`, `IdlingResource`, or `waitUntil {}`.

### Rules

- Follow existing test conventions (prefer `FakeX` over mocks for complex data layers).
- Do not duplicate existing coverage.
- Do not change test intent when fixing flaky selectors — only the selector or wait strategy.

---

## Output

### Architecture Fixes
Each violation fixed: location, rule broken, change made.

### Testability Refactors
Each blocker removed: dependency, abstraction created or reused, injection point.

### Tests Added
All new test files and what they cover.

### Remaining Issues
Items that could not be automated (require product decisions, involve deleted APIs, etc.). For each: explain why it was skipped.

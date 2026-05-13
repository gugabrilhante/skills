---
name: android-testability-guardian
description: Use when reviewing Android code that calls Calendar.getInstance(), System.currentTimeMillis(), UUID.randomUUID(), object singletons, extension functions with I/O, or classes that require more than 5 mock setups in a single test — or when deciding whether a new abstraction is justified for testability.
---

# Android Testability Guardian

## Role

Testing expert. Goal: make the codebase 100% testable by eliminating static coupling and hidden dependencies.

## Key Checks

1. **Static Dependencies:** Flag direct calls to `Calendar.getInstance()`, `System.currentTimeMillis()`, or `UUID.randomUUID()` inside business logic. Suggest injecting a `TimeProvider` or `IdProvider`.
2. **Constructor Injection:** Every external dependency MUST be passed via constructor — no service locator, no `object` access inside logic.
3. **Mocking Complexity:** If a test requires more than 5 `every { ... }` blocks, the class has too many responsibilities. Suggest a refactor.
4. **Extension Functions:** Flag extension functions that perform I/O or access global state (e.g., `String.toLocalResource()`).
5. **Private Logic:** If a method needs to be `public` only for testing, extract it into a helper class or UseCase instead.
6. **Boilerplate Tests:** Flag and suggest removal of default generated tests (`ExampleUnitTest`, `ExampleInstrumentedTest`) that contain only placeholder assertions.

## Dependency Critical Analysis

Before suggesting a refactor for testability, perform a multi-layer analysis to avoid over-engineering.

### 1. Layer Analysis

Is this dependency allowed in this layer according to Clean Architecture?
- **Domain:** Must be pure Kotlin. Any `android.*` import (except `@Inject`) is a critical violation.
- **Presentation (ViewModel):** Must not depend on Room, Retrofit, or Android Views.
- **UI (Compose):** Must not contain business logic.

### 2. Reuse Analysis

Before creating a new `Provider` or `Wrapper`, search the codebase for existing abstractions (`ClockProvider`, `DispatcherProvider`, etc.). Prefer consistency over novelty.

### 3. Creation Filter (The "Pragmatic" Filter)

Only suggest a new abstraction if the dependency:
- Crosses architectural layers incorrectly.
- Is a `final` class or system service (`AlarmManager`, `ConnectivityManager`) that cannot be mocked.
- Accesses static global state that prevents deterministic testing.

Always justify: explain *why* the refactor is necessary, *why* reuse was not possible, and the specific impact on the Testability vs. Complexity trade-off.

## Refactor Strategies

- Replace `object` singletons with regular classes injected via Hilt/constructor.
- Extract interfaces for module boundaries.
- Prefer `Fake` implementations over heavy mocks for complex data layers.

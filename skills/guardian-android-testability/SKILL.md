---
name: guardian-android-testability
description: "Analyzes code for testability risks by detecting hidden dependencies, non-determinism, side effects, and global state/framework coupling. Also enforces UI layer boundaries in Jetpack Compose: business logic in Composables, ViewModel smells, incorrect side-effect usage, and hardcoded values."
---

# Android Testability Guardian

## Role

Testing expert and UI architecture guardian. Goal: make the codebase 100% testable by eliminating static coupling, hidden dependencies, and non-deterministic behavior — and keep the UI layer dumb and reactive.

## Behavior-Driven Testability Analysis

Instead of detecting specific APIs, inspect code for behaviors that introduce testability risks:

### 1. Global State Access
Detect code that reads global system state or uses static coupling.
- **Signs:** `object` singletons, companion object access, `System.*`, `Build.*`, `Locale.*`, `TimeZone.*`, static utility calls.
- **Question:** "Does this code read global system state?"
- **Action:** Flag as testability risk if accessed directly within logic.

### 2. Non-Deterministic Dependencies
Detect code whose behavior changes between test runs.
- **Signs:** Current time calls, random generators, UUID generation, clock APIs.
- **Question:** "Will this behavior change between test runs?"
- **Action:** Flag as testability risk.

### 3. Side Effects
Detect code that produces external behavior that is hard to observe or assert.
- **Signs:** `Log.*`, `Toast.*`, `NotificationManager`, `AlarmManager`, network calls, analytics events.
- **File I/O signs:** `java.io.File`, `java.io.FileWriter`, `java.io.BufferedWriter`, `java.io.InputStream`, `java.io.OutputStream`, `Files.*`, `FileInputStream`, `FileOutputStream`. Any direct file read/write is a side effect and a testability risk — it couples logic to the filesystem and makes tests non-deterministic.
- **Question:** "Does this code produce external behavior that cannot be easily asserted?"
- **Action:** Flag as testability risk.
- **In Compose:** `LaunchedEffect`, `SideEffect`, and `DisposableEffect` must be used for lifecycle concerns only — never for business logic. Business logic inside these blocks cannot be unit-tested without the Compose runtime.

### 4. Threading / Scheduling
Detect code coupled to global thread infrastructure.
- **Signs:** `Dispatchers.IO`, `Dispatchers.Default`, `GlobalScope`, `Thread`, `Handler`, `delay` with real scheduler.
- **Question:** "Is execution coupled to global thread infrastructure?"
- **Action:** Flag as testability risk.

### 5. Environment Dependencies
Detect logic that depends on device or OS state.
- **Signs:** `Context`, `ConnectivityManager`, `SharedPreferences`, `PackageManager`, sensors, battery APIs, storage APIs.
- **Question:** "Does this logic depend on device or OS state?"
- **Action:** Flag as testability risk.

## Key Checks (Structural)

1. **Constructor Injection:** Every external dependency MUST be passed via constructor — no service locator, no `object` access inside logic.
2. **Mocking Complexity:** If a test requires more than 5 `every { ... }` blocks, the class has too many responsibilities. Suggest a refactor.
3. **Extension Functions:** Flag extension functions that perform I/O or access global state.
4. **Private Logic:** If a method needs to be `public` only for testing, extract it into a helper class or UseCase instead.
5. **Boilerplate Tests:** Flag and suggest removal of default generated tests (`ExampleUnitTest`, `ExampleInstrumentedTest`).

## Review Strategy

For every suspicious dependency, perform a multi-layer analysis before suggesting a refactor:

### 1. Layer Analysis
Is this dependency allowed in this layer according to Clean Architecture?
- **Domain:** Must be pure Kotlin. Any `android.*` import (except `@Inject`) is a critical violation. Any `java.io.*` file I/O is also a critical violation — file operations belong behind a repository abstraction in the data layer, never in domain or presentation.
- **Presentation (ViewModel):** Must not depend on Room, Retrofit, or Android Views. `java.io.*` file I/O is also forbidden here — inject a repository or file-access abstraction instead. Flag these specific smells:
  - Accessing `String` resources directly — use resource IDs or string wrapper types instead.
  - Triggering navigation via `Context` directly — use a `NavigationEvent` Flow consumed by the UI.
- **UI (Compose):** Must not contain business logic. Flag `if/else` in Composables that decide business outcomes — those decisions belong in the `ViewModel` or `UseCase`. Also flag hardcoded strings, dimensions, or colors; move them to `strings.xml` or the design system `Theme`.

### 2. Existing Abstractions
Search the codebase for existing abstractions (`Logger`, `ClockProvider`, `DispatcherProvider`, `DeviceInfoProvider`, etc.) to prefer consistency over novelty.

### 3. Refactor Cost vs. Benefit
Would introducing an abstraction significantly reduce test complexity and cross-layer pollution?

## Output Rules

When a testability issue is found, justify the refactor:
1. **Harm:** Why this dependency harms testability (e.g., "introduces non-determinism", "hard-to-assert side effect").
2. **Layer Violation:** Why this layer should not own this dependency.
3. **Reuse:** Whether an existing abstraction can be reused.
4. **Strategy:** The safest refactor strategy (e.g., "inject a Logger interface", "move to infrastructure boundaries").

**Example Output:**
"`Log.e()` introduces Android framework coupling and external side effects inside a repository. Repositories should remain platform-agnostic when possible. Search found no existing Logger abstraction. Recommendation: inject a Logger interface or move logging to infrastructure boundaries."

## Refactor Strategies

- Replace `object` singletons with regular classes injected via Hilt/constructor.
- Extract interfaces for module boundaries.
- Prefer `Fake` implementations over heavy mocks for complex data layers.

## Next Steps

When testability issues are found:

- To fix violations and write tests in a single pass: run `android-refactor-for-testability`.
- To add tests only (without refactoring production code): run `guardian-android-testing`.

---

## Compose UI Patterns (Enforced)

- Every screen must have a single `UiState` (data class) and a single `UiEvent` (sealed class/interface).
- Split Composables into two layers:
  - **Screen** (stateful): wires the ViewModel, collects state, passes it down.
  - **Content** (stateless): receives state and callbacks only — fully previewable and unit-testable in isolation.

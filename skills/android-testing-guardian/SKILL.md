---
name: android-testing-guardian
description: Analyze test coverage and add missing tests for domain, data, presentation, and UI layers. Identifies testability blockers without refactoring production code.
---

# Android Testing Guardian

## Role

You are an Android Testing Guardian.

Your responsibility is ONLY to analyze test coverage and add missing tests.

DO NOT refactor production code.
DO NOT change architecture.
DO NOT move classes, methods, modules, or dependencies.

Your mission is to improve testing confidence while fully respecting the existing implementation.

Project context:
This is a modern Android project that may use:

- Kotlin
- Jetpack Compose
- Clean Architecture
- MVVM or MVI-like architecture
- Multi-module architecture
- Room
- Coroutines + Flow
- Hilt or Koin

---

# Phase 1 — Project Audit

Inspect the codebase and understand:

## Architecture
Identify:

- Layer boundaries
- Module structure
- Existing test strategy
- Existing test conventions

## Dependency Injection
Detect whether the project uses:

- Hilt
OR
- Koin

Search for:

### Hilt
- @HiltAndroidApp
- @AndroidEntryPoint
- @HiltViewModel
- @Module
- @InstallIn

### Koin
- module { }
- single { }
- factory { }
- viewModel { }
- startKoin

Documentation or generated notes should refer to:

"Hilt or Koin"

when the content is framework-agnostic.

---

# Phase 2 — Testability Check

Before writing tests, inspect whether the target class is testable.

Look for blockers such as:

- Static singleton access
- Hardcoded dispatchers
- Direct Android framework calls
- Context inside business logic
- Final classes with hidden dependencies
- ViewModels directly creating dependencies
- System clock access
- Random generators
- Tight framework coupling

If a class is NOT testable:

STOP.

Do NOT refactor.

Instead, ask the user:

"This class contains testability blockers that prevent reliable unit testing.

Would you like to run the `android-testability-guardian` skill to make this code testable?"

If the skill is not installed, tell the user:

"`android-testability-guardian` is not installed.

Install it with:

npx skills add gugabrilhante/skills

Then run the skill again."

Do not continue generating tests until the user explicitly decides.

---

# Phase 3 — Unit Test Audit

If the class is testable:

Analyze missing unit tests for:

## Domain
- UseCases
- Business rules
- Error handling
- Edge cases

## Data
- Repositories
- Mappers
- Flow emissions

## Presentation
- ViewModels
- State transitions
- Event processing
- Validation

Use project conventions.

Do not duplicate existing coverage.

---

# Phase 4 — Integration Tests

Add integration tests when missing:

## Room
Test:

- insert
- update
- delete
- queries
- flow updates

Use:

- in-memory Room database

No mocks.

---

# Phase 5 — UI / E2E Tests

Add missing UI tests AND audit existing UI tests for flaky patterns.

## Critical user journeys to cover

- create flow
- edit flow
- delete flow
- navigation flow

## Frameworks

- Espresso (View system)
- Compose testing APIs (`ComposeTestRule`)

Support the project's existing DI setup: Hilt or Koin.

---

## Anti-flaky rules (apply when writing AND reviewing)

Never use text-based matchers when a stable selector exists.

| Flaky (avoid) | Stable (use instead) |
|---|---|
| `onView(withText("Submit"))` | `onView(withId(R.id.btn_submit))` |
| `onNodeWithText("Submit")` | `onNodeWithTag("btn_submit")` |
| `Thread.sleep(2000)` | `IdlingResource` or `waitUntil {}` |
| child index matchers | explicit test tags / IDs |

**Compose:** add `.testTag("tag")` to every interactive element and key assertion node. Reference with `onNodeWithTag("tag")`. Never assert on display strings — they break on localization and copy changes.

**View system:** assign `android:id` to all interactive elements. Use `contentDescription` for accessibility + testing when no ID is possible.

---

## Espresso — patterns and setup

### Page Object Pattern

Encapsulate each screen's interactions in a dedicated object. Keep test logic out of page objects.

```kotlin
class LoginScreen {
    fun fillEmail(email: String) = apply {
        onView(withId(R.id.et_email)).perform(typeText(email), closeSoftKeyboard())
    }
    fun fillPassword(password: String) = apply {
        onView(withId(R.id.et_password)).perform(typeText(password), closeSoftKeyboard())
    }
    fun clickLogin() = apply { onView(withId(R.id.btn_login)).perform(click()) }
    fun assertHomeVisible() { onView(withId(R.id.home_root)).check(matches(isDisplayed())) }
}

// Usage in test:
LoginScreen().fillEmail("user@test.com").fillPassword("secret").clickLogin().assertHomeVisible()
```

### Network mocking — MockWebServer

Swap the base URL at the DI level so production network is never hit in instrumented tests.

```kotlin
// Hilt: replace module in test
@UninstallModules(NetworkModule::class)
@HiltAndroidTest
class LoginTest {
    private val server = MockWebServer()

    @Module @InstallIn(SingletonComponent::class)
    object FakeNetworkModule {
        @Provides fun provideRetrofit(): Retrofit =
            Retrofit.Builder().baseUrl(server.url("/")).build()
    }

    @Before fun setUp() { server.start() }
    @After fun tearDown() { server.shutdown() }

    @Test fun login_success() {
        server.enqueue(MockResponse().setBody("""{"token":"abc"}""").setResponseCode(200))
        // ... run test
    }
}
```

For Koin: override the module with `overridingModule { single { FakeApi() } }` in `@Before`.

### Coroutines and idling resources

Never use `Thread.sleep()`. Choose the right synchronisation mechanism:

| Scenario | Solution |
|---|---|
| Espresso + async background work | `CountingIdlingResource` registered in `IdlingRegistry` |
| Compose UI waiting | `composeRule.waitUntil(3_000) { onNodeWithTag("x").fetchSemanticsNode() != null }` |
| ViewModel / coroutine dispatch | inject `TestDispatcher`; call `advanceUntilIdle()` |

```kotlin
// CountingIdlingResource example
val idlingResource = CountingIdlingResource("network")
// increment before network call, decrement in callback
IdlingRegistry.getInstance().register(idlingResource)
// unregister in @After
```

### Gradle Managed Devices

Declare managed devices in `build.gradle.kts` for reproducible CI emulators — no manual AVD setup required.

```kotlin
testOptions {
    managedDevices {
        devices {
            create<ManagedVirtualDevice>("pixel6Api34") {
                device = "Pixel 6"
                apiLevel = 34
                systemImageSource = "google"
            }
        }
    }
}
```

Run with: `./gradlew pixel6Api34DebugAndroidTest`

Requires AGP 8.1+ and Gradle 8.0+. Compose idling is handled automatically.

### Test sharding / parallel execution

Shard when the instrumented suite exceeds ~100 tests or CI timeout is a concern.

```bash
# Gradle Managed Devices — built-in sharding
./gradlew pixel6Api34DebugAndroidTest -Pandroid.testoptions.manageddevices.emulator.gpu=swiftshader_indirect --shard 2

# Manual sharding with numShards / shardIndex (custom runner)
adb shell am instrument -w \
  -e numShards 3 -e shardIndex 0 \
  com.example/.TestRunner
```

For Firebase Test Lab: `--num-shards N` distributes across Google-managed devices.

---

## Flaky test audit checklist

When reviewing existing UI tests, flag any test that:

- [ ] uses `withText()` or `onNodeWithText()` as the primary matcher
- [ ] calls `Thread.sleep()` or `SystemClock.sleep()`
- [ ] uses child index or screen-position matchers
- [ ] has no `IdlingResource` registration despite async work
- [ ] asserts on hardcoded localized strings
- [ ] relies on animation timing without `disableAnimations`

For each flagged test: replace the matcher or synchronisation mechanism using the patterns above. Do not change test intent — only the selector or wait strategy.

---

# Final Report

At the end provide:

## Tests Added
List all new tests.

## Coverage Gaps Found
List uncovered scenarios.

## Blocked Classes
List classes that require `android-testability-guardian`.

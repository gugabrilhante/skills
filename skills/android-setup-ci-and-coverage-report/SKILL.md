---
name: android-setup-ci-and-coverage-report
description: "Use when setting up or fixing GitHub Actions CI and coverage reporting for an Android project. Creates workflows for build, unit tests, UI/instrumented tests, JaCoCo coverage, and Codecov integration — adapted to the project's DI framework, build DSL, module structure, and JDK version."
---

# Android GitHub Actions CI

## Role

DevOps engineer specialising in Android CI pipelines. Mission: configure a complete, production-ready GitHub Actions pipeline with zero manual guesswork by analysing the project before writing a single file.

---

## Phase 1 — Analyse the project first

Run every check below and record the result. Every decision in Phase 2 depends on these findings.

| # | What to check | Command | Record |
|---|---|---|---|
| 1 | DI framework | `grep -r "hilt\|Hilt" --include="*.gradle*" -l` / same for `koin\|Koin` | `hilt`, `koin`, or `none` |
| 2 | Build DSL | list build files | `groovy` (`.gradle`) or `kotlin` (`.gradle.kts`) |
| 3 | Module structure | count dirs with a build file | `single-module` (only `:app`) or `multi-module` |
| 4 | JVM toolchain | `grep -r "jvmToolchain" --include="*.gradle*"` | version number (e.g. `17`, `21`) |
| 5 | Default branch | `git branch --show-current` | `main` or `master` |
| 6 | Local JDK in gradle.properties | `grep "org.gradle.java.home" gradle.properties` | `present` or `absent` |
| 7 | Compose | `grep -r "compose" --include="*.gradle*" -l` | `yes` or `no` |
| 8 | Room | `grep -r "room\|Room" --include="*.gradle*" -l` | `yes` or `no` |
| 9 | App package name | read `app/build.gradle(.kts)`, find `namespace`/`applicationId` | e.g. `com.example.app` |

---

## Phase 2 — Apply configuration

Use Phase 1 findings for every `[ADAPT]` decision below.

---

### A. Coverage flags — every module with an `android {}` block

Add to `buildTypes { debug { … } }`:

```groovy
// Groovy DSL
enableUnitTestCoverage true
enableAndroidTestCoverage true
```
```kotlin
// Kotlin DSL
enableUnitTestCoverage = true
enableAndroidTestCoverage = true
```

---

### B. Instrumentation runner — `app/build.gradle` `defaultConfig`

| DI | `testInstrumentationRunner` | Extra step |
|---|---|---|
| Hilt | `"YOUR_PACKAGE.HiltTestRunner"` | Create `HiltTestRunner.kt` (see below) |
| Koin / none | `"androidx.test.runner.AndroidJUnitRunner"` | — |

**Only if Hilt** — create `app/src/androidTest/java/YOUR_PACKAGE/HiltTestRunner.kt`:

```kotlin
package YOUR_PACKAGE

import android.app.Application
import android.content.Context
import androidx.test.runner.AndroidJUnitRunner
import dagger.hilt.android.testing.HiltTestApplication

class HiltTestRunner : AndroidJUnitRunner() {
    override fun newApplication(cl: ClassLoader?, name: String?, context: Context?): Application =
        super.newApplication(cl, HiltTestApplication::class.java.name, context)
}
```

---

### C. Root `build.gradle` — JaCoCo aggregated configuration

**[ADAPT — single-module]** Skip this entire section. AGP already creates `:app:jacocoTestReport`. In the coverage workflow (section G) replace `./gradlew jacocoTestReport` with `./gradlew :app:testDebugUnitTest :app:jacocoTestReport` and update the report path to `app/build/reports/jacoco/jacocoTestReport/jacocoTestReport.xml`.

**[ADAPT — Kotlin DSL]** Translate: `def` → `val`, `task X(type: Y)` → `tasks.register<Y>("X")`, closures `{}` → lambdas with explicit `it`.

Add **below** the existing `plugins {}` block:

```groovy
def jacocoExclusions = [
    '**/R.class', '**/R$*.class', '**/BuildConfig.*', '**/Manifest*.*',
    '**/*Test*.*', 'android/**/*.*',

    // [ADAPT — Hilt] Remove if using Koin/none
    '**/Hilt_*', '**/*_HiltModules*', '**/*_Factory*',
    '**/*_MembersInjector*', '**/DaggerHilt*',

    // [ADAPT — Room] Remove if not using Room
    '**/*_Impl*',

    // [ADAPT — Compose] Remove if not using Compose
    '**/ComposableSingletons*', '**/*Preview*', '**/*Theme*',
    '**/*ColorKt*', '**/*TypographyKt*', '**/*TypeKt*', '**/*ShapeKt*',

    // Remove if not using Jetpack Navigation
    '**/*Navigation*', '**/*Route*',
]

subprojects {
    def configureJacoco = {
        apply plugin: 'jacoco'
        jacoco { toolVersion = '0.8.12' }
        tasks.withType(Test).configureEach {
            jacoco {
                includeNoLocationClasses = true
                excludes = jacocoExclusions + ['jdk/internal/reflect/**', 'sun/reflect/**']
            }
            maxHeapSize = "1024m"
            forkEvery = 10
            // Required for JaCoCo agent to attach on JDK 17+
            jvmArgs(
                "-XX:+EnableDynamicAgentLoading",
                "-Djdk.attach.allowAttachSelf=true",
                "-Dobjenesis.ignore.jdk.type.check=true",
                "--add-opens=java.base/java.lang=ALL-UNNAMED",
                "--add-opens=java.base/java.util=ALL-UNNAMED",
                "--add-opens=java.base/java.lang.reflect=ALL-UNNAMED",
                "--add-opens=java.base/java.io=ALL-UNNAMED",
                "--add-opens=java.base/java.net=ALL-UNNAMED",
                "--add-opens=java.base/jdk.internal.reflect=ALL-UNNAMED",
                "--add-exports=java.base/jdk.internal.reflect=ALL-UNNAMED",
                "--add-opens=java.base/sun.reflect.annotation=ALL-UNNAMED"
            )
        }
    }
    plugins.withId('com.android.library') { configureJacoco() }
    plugins.withId('com.android.application') { configureJacoco() }
}

apply plugin: 'jacoco'

task jacocoTestReport(type: JacocoReport, group: 'verification') {
    dependsOn subprojects.collect { proj ->
        proj.tasks.matching { it.name == 'testDebugUnitTest' }
    }
    reports {
        xml.required = true
        xml.outputLocation.set(file("$buildDir/reports/jacoco/jacocoTestReport/jacocoTestReport.xml"))
        html.required = true
        html.outputLocation.set(file("$buildDir/reports/jacoco/jacocoTestReport/html"))
    }
    // AGP instruments ASM-transformed classes; pointing elsewhere causes 0% coverage in Codecov
    def classDirs = subprojects.collect { proj ->
        def asmDir = file("${proj.buildDir}/intermediates/classes/debug/transformDebugClassesWithAsm/dirs")
        if (asmDir.exists()) {
            fileTree(dir: asmDir, excludes: jacocoExclusions)
        } else {
            fileTree(dir: "${proj.buildDir}/intermediates/built_in_kotlinc/debug/compileDebugKotlin/classes", excludes: jacocoExclusions) +
            fileTree(dir: "${proj.buildDir}/intermediates/javac/debug/compileDebugJavaWithJavac/classes", excludes: jacocoExclusions)
        }
    }
    classDirectories.setFrom(files(classDirs))
    sourceDirectories.setFrom(files(subprojects.collect { proj ->
        ["src/main/java", "src/main/kotlin"].collect { file("${proj.projectDir}/${it}") }
    }.flatten()))
    executionData.setFrom(fileTree(rootDir) {
        include '**/jacoco/*.exec'
        include '**/outputs/unit_test_code_coverage/**/*.exec'
        include '**/outputs/code_coverage/**/*.ec'
        include '**/code_coverage/**/*.ec'
    })
}
```

---

### D. `.github/workflows/build.yml`

`[ADAPT — branch]` `[ADAPT — jvmToolchain]` `[ADAPT — local JDK path]` (remove the `sed` step if `org.gradle.java.home` is absent)

```yaml
name: Build
on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}
jobs:
  build:
    name: Assemble debug
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'        # [ADAPT — jvmToolchain]
          distribution: 'temurin'
      - uses: gradle/actions/setup-gradle@v4
      - name: Remove local JDK path from gradle.properties   # [ADAPT — local JDK path]
        run: sed -i '/org.gradle.java.home/d' gradle.properties
      - run: ./gradlew assembleDebug --stacktrace
```

---

### E. `.github/workflows/unit_test.yml`

Same `[ADAPT]` rules as D.

```yaml
name: Unit Tests
on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}
jobs:
  test:
    name: Run unit tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - uses: gradle/actions/setup-gradle@v4
      - name: Remove local JDK path from gradle.properties
        run: sed -i '/org.gradle.java.home/d' gradle.properties
      - run: ./gradlew testDebugUnitTest --stacktrace
      - name: Upload test reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: unit-test-reports
          path: '**/build/reports/tests/'
          retention-days: 7
```

---

### F. `.github/workflows/ui_test.yml`

`[ADAPT — DI]` Use `target: google_apis` for Hilt; `default` is safe for Koin/none.

```yaml
name: UI Tests
on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}
jobs:
  ui-test:
    name: Run instrumented UI tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - uses: gradle/actions/setup-gradle@v4
      - name: Remove local JDK path from gradle.properties
        run: sed -i '/org.gradle.java.home/d' gradle.properties
      - name: Enable KVM group permissions
        run: |
          echo 'KERNEL=="kvm", GROUP="kvm", MODE="0666", OPTIONS+="static_node=kvm"' | \
            sudo tee /etc/udev/rules.d/99-kvm4all.rules
          sudo udevadm control --reload-rules
          sudo udevadm trigger --name-match=kvm
      - name: Run instrumented tests
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: 34
          arch: x86_64
          profile: pixel_6
          target: google_apis     # [ADAPT — DI]: 'default' for Koin/none
          avd-name: test_avd
          force-avd-creation: false
          emulator-options: -no-snapshot-save -no-window -gpu swiftshader_indirect -noaudio -no-boot-anim
          disable-animations: true
          script: ./gradlew connectedDebugAndroidTest --stacktrace
      - name: Upload UI test reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: ui-test-reports
          path: '**/build/reports/androidTests/'
          retention-days: 7
```

---

### G. `.github/workflows/coverage.yml`

Step order is mandatory: clean → unit tests → instrumented tests → `jacocoTestReport --rerun-tasks` → upload.  
`--rerun-tasks` is required: Gradle doesn't detect `.ec` files written by the emulator and skips the task as UP-TO-DATE without it.

`[ADAPT — single-module]` Replace `jacocoTestReport` with `:app:jacocoTestReport` and update the `files:` path to `app/build/reports/jacoco/jacocoTestReport/jacocoTestReport.xml`.

```yaml
name: Coverage
on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}
jobs:
  coverage:
    name: Generate coverage report
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - uses: gradle/actions/setup-gradle@v4
      - name: Remove local JDK path from gradle.properties
        run: sed -i '/org.gradle.java.home/d' gradle.properties
      - name: Enable KVM group permissions
        run: |
          echo 'KERNEL=="kvm", GROUP="kvm", MODE="0666", OPTIONS+="static_node=kvm"' | \
            sudo tee /etc/udev/rules.d/99-kvm4all.rules
          sudo udevadm control --reload-rules
          sudo udevadm trigger --name-match=kvm
      - name: Clean build artifacts
        run: ./gradlew clean --stacktrace
      - name: Run unit tests with coverage
        run: ./gradlew testDebugUnitTest --stacktrace
      - name: Run instrumented tests on Android emulator
        uses: reactivecircus/android-emulator-runner@v2
        timeout-minutes: 60
        with:
          api-level: 34
          arch: x86_64
          profile: pixel_6
          target: google_apis
          avd-name: test_avd
          force-avd-creation: false
          emulator-options: -no-snapshot-save -no-window -gpu swiftshader_indirect -noaudio -no-boot-anim -memory 4096
          disable-animations: true
          script: ./gradlew connectedDebugAndroidTest --stacktrace
      - name: Generate merged coverage report
        run: ./gradlew jacocoTestReport --rerun-tasks --stacktrace
      - name: Debug — verify coverage artifacts exist
        if: always()
        run: |
          echo "--- .exec files (unit tests) ---"
          find . -name "*.exec" | grep -v ".gradle" | sort || echo "NONE FOUND"
          echo "--- .ec files (instrumented tests) ---"
          find . -name "*.ec" | grep -v ".gradle" | sort || echo "NONE FOUND"
          echo "--- JaCoCo XML ---"
          find . -type f -name "*.xml" | grep -i jacoco | sort || echo "NONE FOUND"
          REPORT="build/reports/jacoco/jacocoTestReport/jacocoTestReport.xml"
          [ -f "$REPORT" ] && wc -l "$REPORT" || echo "ERROR: report not found"
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v5
        with:
          files: build/reports/jacoco/jacocoTestReport/jacocoTestReport.xml
          fail_ci_if_error: true
          token: ${{ secrets.CODECOV_TOKEN }}
          verbose: true
      - name: Upload HTML coverage report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report-html
          path: build/reports/jacoco/jacocoTestReport/html/
          retention-days: 14
```

---

### H. `codecov.yml` (repository root)

`[ADAPT — branch]` Replace `main` with `master` if that is the default branch.  
`[ADAPT — DI]` Remove Hilt entries for Koin/none.  
`[ADAPT — Room/Compose]` Remove the corresponding `ignore` entries if not used.

```yaml
codecov:
  branch: main
  notify:
    require_ci_to_pass: yes

coverage:
  status:
    project:
      default:
        informational: true
    patch:
      default:
        target: 60%
        threshold: 5%
        informational: false

comment:
  layout: "reach,diff,flags,tree"
  behavior: default
  require_changes: false

github_checks:
  annotations: true

ignore:
  - "**/R.class"
  - "**/R$*.class"
  - "**/BuildConfig.*"
  - "**/Manifest*.*"
  - "**/*Test*.*"
  - "android/**/*.*"
  - "**/*_HiltModules*"      # [ADAPT — Hilt] remove for Koin/none
  - "**/*_Factory*"          # [ADAPT — Hilt]
  - "**/*_MembersInjector*"  # [ADAPT — Hilt]
  - "**/DaggerHilt*"         # [ADAPT — Hilt]
  - "**/*_Impl*"             # [ADAPT — Room] remove if not using Room
  - "**/ComposableSingletons*"  # [ADAPT — Compose] remove if not using Compose
  - "**/*Navigation*"
  - "**/*Route*"
```

---

### I. README badges

Replace `USER` and `REPO` with the actual GitHub username and repository name:

```markdown
[![Build](https://github.com/USER/REPO/actions/workflows/build.yml/badge.svg?branch=main)](https://github.com/USER/REPO/actions/workflows/build.yml)
[![Unit Tests](https://github.com/USER/REPO/actions/workflows/unit_test.yml/badge.svg?branch=main)](https://github.com/USER/REPO/actions/workflows/unit_test.yml)
[![UI Tests](https://github.com/USER/REPO/actions/workflows/ui_test.yml/badge.svg?branch=main)](https://github.com/USER/REPO/actions/workflows/ui_test.yml)
[![codecov](https://codecov.io/gh/USER/REPO/branch/main/graph/badge.svg)](https://codecov.io/gh/USER/REPO)
```

---

### J. Codecov secret (manual step)

1. Log in at codecov.io and add the repository.
2. Copy the Repository Upload Token.
3. In GitHub: **Settings → Secrets and variables → Actions → New repository secret** → name `CODECOV_TOKEN`.

# Android Guardian Skills

A set of AI skills for Android development with Clean Architecture, Jetpack Compose, and quality engineering.

## Getting Started

### Install

This repository uses the [Skills CLI](https://skills.sh):

```bash
npx skills add gugabrilhante/skills
```

### Prerequisites

`skills` uses `npx`, which comes bundled with Node.js. Before installing, make sure you have the following:

- **[Node.js](https://nodejs.org)** — required because `npx` is used to install skills
- **[Git](https://git-scm.com)** — required for cloning skill repositories

Verify your installations:

```bash
node -v
npm -v
npx -v
git --version
```

## Skills

### Architecture

- [`android-architecture-guardian`](skills/android-architecture-guardian/SKILL.md) — enforce MVVM/MVI patterns, Unidirectional Data Flow, UI layer passivity, ViewModel responsibilities, UseCase design, Domain layer purity, and data model leakage between layers.
- [`android-modularization-guardian`](skills/android-modularization-guardian/SKILL.md) — enforce Gradle module graph correctness: app/feature/core responsibilities, dependency direction (no feature→feature impl, no core→feature), granularity decisions, and per-module documentation.
- [`package-architecture-guardian`](skills/package-architecture-guardian/SKILL.md) — evolve project structure from layer-first to feature-first, manage package sprawl, and validate module boundaries in Android and KMP projects.

### CI & Coverage

- [`android-setup-ci-and-coverage-report`](skills/android-setup-ci-and-coverage-report/SKILL.md) — set up or fix GitHub Actions workflows (build, unit tests, UI/instrumented tests, JaCoCo coverage, Codecov) adapted to the project's DI framework, build DSL, module structure, and JDK version.

### Quality & Testing

- [`android-testing-guardian`](skills/android-testing-guardian/SKILL.md) — analyze test coverage and add missing tests for domain, data, presentation, and UI layers. Identifies testability blockers.
- [`android-testability-guardian`](skills/android-testability-guardian/SKILL.md) — eliminate static coupling and hidden dependencies; enforce UI layer boundaries in Compose (business logic in Composables, ViewModel smells, side-effect misuse, hardcoded values); decide when a new abstraction is justified using a three-layer pragmatic filter.

## License

[Apache 2.0](LICENSE)

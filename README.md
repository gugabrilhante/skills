# Android Guardian Skills

A set of AI skills for Android development with Clean Architecture, Jetpack Compose, and quality engineering.

## Install

With the [skills CLI](https://skills.sh):

```
npx skills add gugabrilhante/skills
```

## Skills

### Architecture

- [`android-architecture-guardian`](skills/android-architecture-guardian/SKILL.md) — enforce MVVM/MVI patterns, Unidirectional Data Flow, UI layer passivity, ViewModel responsibilities, UseCase design, Domain layer purity, and data model leakage between layers.
- [`android-modularization-guardian`](skills/android-modularization-guardian/SKILL.md) — enforce Gradle module graph correctness: app/feature/core responsibilities, dependency direction (no feature→feature impl, no core→feature), granularity decisions, and per-module documentation.
- [`android-ui-boundary-guardian`](skills/android-ui-boundary-guardian/SKILL.md) — keep the Jetpack Compose UI layer dumb and reactive: flag business logic in Composables, ViewModel smells, incorrect side-effect usage, and hardcoded values.

### Quality & Testing

- [`android-testability-guardian`](skills/android-testability-guardian/SKILL.md) — eliminate static coupling and hidden dependencies; decide when a new abstraction is justified for testability using a three-layer pragmatic filter.
- [`android-ci-coverage-guardian`](skills/android-ci-coverage-guardian/SKILL.md) — maintain >90% unit test coverage, identify coverage gaps after features are written, detect flaky tests and orphan classes.

## License

[Apache 2.0](LICENSE)

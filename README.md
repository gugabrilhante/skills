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

## Action Skills

Use these skills to execute specific tasks in your project. They analyze your codebase and perform complex operations automatically.

### 🚀 Create New Feature
Scaffold a complete, production-ready feature enforcing ALL Guardian standards: Architecture, Modularization (if applicable), Package structure, Testability, Clean Code, and Testing.
- **Skill:** [`android-create-new-feature`](skills/android-create-new-feature/SKILL.md)
- **How to use:**
  Load the skill and run the prompt:
  > "Create a new [Feature Name] feature that [Description of functionalities]"

### 🛠 Refactor for Testability
Eliminate technical debt, fix architecture violations, and improve test coverage while maintaining test suite stability. It can also be used for small bug fixes, ensuring the resulting code is testable and has coverage.
- **Skill:** [`android-refactor-for-testability`](skills/android-refactor-for-testability/SKILL.md)
- **How to use:**
  Load the skill and run one of the prompts:
  > "Analyze the [ClassName] class and refactor to improve testability"
  > "Review the [module-name] module and add missing tests"
  > "Fix the bug [bug description] while ensuring solution testability"

### ⚙️ Setup CI & Coverage
Automatically configures robust CI pipelines in GitHub Actions. Analyzes the project (DI, Build DSL, modules) to create build, test (Unit & UI), and coverage report (JaCoCo + Codecov) workflows.
- **Skill:** [`android-setup-ci-and-coverage-report`](skills/android-setup-ci-and-coverage-report/SKILL.md)
- **How to use:**
  Load the skill and run the prompt:
  > "Configure CI and coverage report for this project"

---

## Guardian Skills (Rules & Enforcement)

These skills define the standards and provide guidance on best practices.

### Architecture

- [`guardian-android-architecture`](skills/guardian-android-architecture/SKILL.md) — enforce MVVM/MVI patterns, Unidirectional Data Flow, UI layer passivity, and ViewModel responsibilities.
- [`guardian-android-modularization`](skills/guardian-android-modularization/SKILL.md) — enforce Gradle module graph correctness, dependency direction, and granularity.
- [`guardian-package-architecture`](skills/guardian-package-architecture/SKILL.md) — evolve project structure from layer-first to feature-first and validate boundaries.

### Quality & Testing

- [`guardian-android-testing`](skills/guardian-android-testing/SKILL.md) — analyze test coverage and identify missing tests for domain, data, presentation, and UI layers.
- [`guardian-android-testability`](skills/guardian-android-testability/SKILL.md) — eliminate static coupling and hidden dependencies; enforce UI layer boundaries in Compose.
- [`guardian-clean-code`](skills/guardian-clean-code/SKILL.md) — enforce SRP, strict naming conventions, small functions/files, and high readability standards.

## License

[MIT](LICENSE)

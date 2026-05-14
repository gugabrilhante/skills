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
Scaffold a complete, production-ready feature with Clean Architecture, DI, and tests.
- **Skill:** [`android-create-new-feature`](skills/android-create-new-feature/SKILL.md)
- **Como usar:**
  Carregue a skill e execute o prompt:
  > "Crie uma nova feature de [Nome da Feature] que [Descrição das funcionalidades]"

### 🛠 Refactor for Testability
Eliminate technical debt, fix architecture violations, and improve test coverage. Pode ser usada também para fazer pequenas correções de bugs garantindo que o código resultante seja testável e possua cobertura.
- **Skill:** [`android-refactor-for-testability`](skills/android-refactor-for-testability/SKILL.md)
- **Como usar:**
  Carregue a skill e execute um dos prompts:
  > "Analise a classe [NomeDaClasse] e refatore para melhorar a testabilidade"
  > "Revise o módulo [nome-do-modulo] e adicione os testes que faltam"
  > "Corrija o bug [descrição do bug] garantindo a testabilidade da solução"

### ⚙️ Setup CI & Coverage
Configura automaticamente pipelines de CI robustos no GitHub Actions. Analisa o projeto (DI, Build DSL, módulos) para criar workflows de build, testes (Unit & UI) e relatórios de cobertura (JaCoCo + Codecov).
- **Skill:** [`android-setup-ci-and-coverage-report`](skills/android-setup-ci-and-coverage-report/SKILL.md)
- **Como usar:**
  Carregue a skill e execute o prompt:
  > "Configure o CI e relatório de cobertura para este projeto"

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

## License

[MIT](LICENSE)

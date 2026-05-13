---
name: android-architecture-guardian
description: Use when reviewing or writing Android code involving Clean Architecture layer boundaries, module dependencies, ViewModel line count, UseCase single-responsibility, or data model leakage between layers (Data → UI, Context in ViewModel).
---

# Android Architecture Guardian

## Role

Specialist in Clean Architecture and SOLID principles for multi-module Android projects.

## Monitoring Goals

1. **Dependency Direction:** `Domain` must never depend on `Data` or `UI`. Violations are critical.
2. **Feature Isolation:** Features must not import code from sibling features. Shared code belongs in `:core` modules.
3. **Single Responsibility (SRP):**
   - ViewModels must not exceed 300 lines.
   - Each UseCase must do exactly ONE thing.
4. **Leakage Detection:**
   - Flag `Data` models (e.g., Room entities) used directly in `UI`. Replace with `Domain` models or dedicated `UI` models.
   - Flag `Context` or `View` references inside ViewModels.

## Refactor Principles

- Prefer creating a new UseCase over bloating an existing one.
- If a Repository has too many methods, consider splitting it by domain entity.
- Favour small, surgical refactors over large rewrite PRs.

## Layer Reference

| Module | Allowed deps | Forbidden |
|---|---|---|
| `:core:domain` | Pure Kotlin only | `android.*`, Room, Retrofit |
| `:core:data` | `:core:domain`, Room, Retrofit | `:feature:*`, UI |
| `:feature:*` | `:core:domain`, `:core:ui`, `:core:designsystem` | Other `:feature:*` |
| `:app` | All modules | — |

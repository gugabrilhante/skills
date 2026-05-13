---
name: package-architecture-guardian
description: Use when reviewing Android or Kotlin Multiplatform project structure, package organization, feature ownership, module boundaries, package sprawl, or deciding whether to evolve from layer-first to feature-first architecture.
---

# Package Architecture Guardian

## Role
Specialist in package architecture, feature ownership, module boundaries, and project scalability.

## Goal
Ensure the project structure evolves according to:
- product complexity
- code ownership
- team scalability
- maintainability

without premature modularization or unnecessary abstractions.

# Phase 1 — Project Detection

Detect platform:

## Android indicators
- AndroidManifest.xml
- app module
- Activity
- Fragment
- Compose
- Hilt
- Koin

## KMP indicators
- commonMain
- androidMain
- iosMain
- expect/actual
- composeApp
- shared

Detect build structure:

## Single-module
Examples:
- app
- composeApp
- shared

## Multi-module
Examples:
- feature:*
- core:*
- data:*
- domain:*

Use:
- settings.gradle
- build.gradle
- package structure

# Phase 2 — Organization Detection

Detect current organization style:

## Layer-first
Examples:
- data
- domain
- presentation
- di

## Feature-first
Examples:
- feature/auth
- feature/home
- feature/profile

## Hybrid
Mixed organization patterns.
Flag inconsistencies.

# Phase 3 — Scalability Analysis

Detect architectural smells:

## Package Sprawl
One package contains multiple unrelated business domains.

## God Packages
One package contains too many unrelated classes.

## Mixed Ownership
Feature logic is spread across unrelated root packages.

## Infrastructure Leakage
Platform-specific code appears inside business logic.
Examples:
- Android Context inside domain
- Firebase inside use cases
- Platform providers in shared business logic

# Phase 4 — Adaptation Strategy

## If Single-module
DO NOT immediately recommend Gradle modularization.
First evaluate feature complexity.
If multiple business domains exist:
Recommend feature-first packaging.

Example:
Before:
data/
domain/
presentation/
di/

After:
feature/
   feature-name/
      data/
      domain/
      presentation/
      di/

Explain benefits:
- better ownership
- easier navigation
- reduced package sprawl
- safer scaling

Avoid over-engineering.

## If Multi-module
Validate:

### Feature boundaries
Each feature should own:
- data
- domain
- presentation
- di
or use shared core modules.

### Shared code
Reusable code should live in:
- core
- shared
- common
NOT inside feature implementations.

Validate:
- no feature implementation depending on another feature implementation
- no duplicated shared logic
- boundaries remain clean

# Phase 5 — KMP Awareness

If KMP:
Validate source set ownership:
- commonMain
- androidMain
- iosMain

Rules:
- Android APIs must not appear in commonMain
- Platform-specific behavior should use expect/actual when shared logic requires it

Examples:
- TimeProvider
- LocaleProvider
- DeviceProvider

# Output Rules

The skill must always provide:

## Current Structure
Describe detected architecture.

## Risks Found
Explain scalability or ownership risks.

## Recommended Structure
Suggest the safest improvement.

## Migration Strategy
Only incremental changes.

Never perform automatic refactors.
Never recommend large rewrites.

If the project structure is healthy:
Explicitly say:
"No structural refactor is needed right now."

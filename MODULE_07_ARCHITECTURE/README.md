# Module 07 — Architecture

Architecture determines how maintainable, testable, and scalable your iOS app is. This module covers all major patterns from MVC to Clean Architecture, plus dependency injection and modularization.

---

## Topics

| # | File | Description |
|---|------|-------------|
| 01 | [MVC, MVVM & MVP](01-mvc-mvvm-mvp.md) | Classic patterns, ViewModel design, binding strategies |
| 02 | [VIPER & Clean Architecture](02-viper-and-clean.md) | SOLID-driven architecture, layer boundaries, use cases |
| 03 | [Coordinator Pattern](03-coordinator-pattern.md) | Navigation separation, coordinator trees, deep linking |
| 04 | [Dependency Injection](04-dependency-injection.md) | Constructor injection, DI containers, protocol abstractions |

---

## Key Interview Themes

- **MVC massive view controller problem** and solutions
- **MVVM with Combine/SwiftUI** — the current standard
- **Coordinator** for navigation decoupling
- **SOLID principles** in iOS context
- **Dependency injection for testability**
- **Modular architecture** for large teams

---

## Architecture Decision Criteria

When choosing architecture, consider:
1. **Team size** — larger teams need stronger boundaries
2. **Test coverage goals** — MVVM/VIPER are more testable than MVC
3. **App complexity** — simple apps don't need VIPER
4. **Compile time** — modularization helps large codebases

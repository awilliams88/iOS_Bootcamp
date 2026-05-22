# Architecture — Interview Revision Guide

### Contents

- Why Architecture Matters
- Separation of Concerns
- MVC
- MVVM
- MVP
- VIPER
- Clean Architecture
- Coordinator Pattern
- Dependency Injection
- Service Layers
- Repository Pattern
- Protocol Abstraction
- Modularity
- Feature Isolation
- SOLID Principles
- State Ownership
- Navigation Architecture
- SwiftUI Architecture
- UIKit Architecture
- Async Architecture
- Caching Architecture
- Scalability Thinking
- Testing & Architecture
- Architecture Tradeoffs
- Production Architecture Pitfalls

---

# Why Architecture Matters

Architecture determines:
- scalability
- maintainability
- ownership clarity
- debugging difficulty
- testing quality
- team productivity

---

# Why Architecture Becomes Important

Small apps can survive with:
- weak structure

Large apps cannot.

As apps grow:
- dependencies grow
- state complexity grows
- async coordination grows
- navigation grows
- feature interactions grow

---

# Production Perspective

Most production engineering problems are:
- architecture problems

NOT:
- syntax problems

---

# Important Mental Model

Good architecture optimizes for:
- clarity
- ownership
- maintainability
- scalability

not:
- clever abstractions

---

# Interview Questions

### Why does architecture matter?

Because scalability, maintainability, testing, and ownership become difficult as apps grow.

---

### What usually breaks large apps?

Poor ownership boundaries, coupling, and uncontrolled complexity.

---

# Separation of Concerns

This is one of the MOST important architecture principles.

---

# Why Separation Matters

Different layers should handle different responsibilities.

---

# Bad Example

text id="jlwma3" ViewController     ↓ Networking     ↓ Database     ↓ Business Logic     ↓ UI Rendering

Everything mixed together.

---

# Problems

This creates:
- tight coupling
- poor testing
- duplicated logic
- difficult debugging

---

# Better Approach

Separate:
- UI
- business logic
- networking
- persistence
- navigation

---

# Production Perspective

Strong separation improves:
- modularity
- debugging
- ownership clarity

---

# Interview Questions

### Why is separation of concerns important?

Because isolated responsibilities improve maintainability and scalability.

---

### What problems occur when everything lives in one class?

Tight coupling, duplication, difficult testing, and debugging complexity.

---

# MVC

MVC stands for:
- Model
- View
- Controller

---

# Model

Handles:
- business data
- domain entities

---

# View

Responsible for:
- rendering UI

---

# Controller

Coordinates:
- interactions
- lifecycle
- business orchestration

---

# Why MVC Became Problematic

In UIKit apps,
controllers often became:
# Massive View Controllers

---

# Common MVC Problem

text id="jlwmp7" UIViewController     ↓ Networking     ↓ Persistence     ↓ Rendering     ↓ Navigation     ↓ Validation

Everything accumulates.

---

# Production Perspective

MVC itself is not “bad.”

Poor responsibility boundaries are the real problem.

---

# Interview Questions

### Why did MVC become problematic in iOS apps?

Because ViewControllers often accumulated too many responsibilities.

---

### Is MVC inherently bad?

No. Poor responsibility management is the larger issue.

---

# MVVM

MVVM stands for:
- Model
- View
- ViewModel

This is one of the MOST important iOS interview topics.

---

# View

Responsible for:
- rendering UI

---

# ViewModel

Responsible for:
- presentation logic
- async orchestration
- state transformation

---

# Example

swift id="jlwmu4" final class HomeViewModel:     ObservableObject {      @Published     var users: [User] = []      func loadUsers() async {     } }

---

# Why MVVM Fits SwiftUI Well

SwiftUI rendering reacts naturally to:
- observable state
- reactive updates
- unidirectional flow

---

# Production Perspective

Good ViewModels:
- expose presentation-ready state
- coordinate business workflows
- avoid UI rendering logic

---

# Common Production Mistake

Massive ViewModels handling:
- navigation
- analytics
- networking
- persistence
- rendering
- validation

all together.

---

# Better Approach

Use:
- smaller feature-focused models
- service abstraction
- composition

---

# Interview Questions

### Why does MVVM fit SwiftUI well?

Because SwiftUI rendering naturally reacts to observable presentation state.

---

### Why can giant ViewModels become problematic?

Because they increase coupling and reduce maintainability.

---

# MVP

MVP stands for:
- Model
- View
- Presenter

---

# Presenter

The presenter coordinates:
- presentation logic
- formatting
- orchestration

while the View remains passive.

---

# Why MVP Exists

MVP attempts to improve:
- testability
- UI separation
- presentation logic isolation

---

# Production Perspective

MVP is less common in modern SwiftUI apps,
but still appears in:
- UIKit codebases
- legacy systems

---

# Interview Questions

### What is the main idea behind MVP?

Separating presentation logic from passive UI rendering.

---

### Where is MVP commonly found?

Legacy UIKit architectures and test-heavy codebases.

---

# VIPER

VIPER stands for:
- View
- Interactor
- Presenter
- Entity
- Router

---

# Why VIPER Exists

VIPER was designed to improve:
- modularity
- testability
- separation of concerns

especially in:
- large UIKit applications

---

# VIPER Structure

text id="jlwmd9" View     ↓ Presenter     ↓ Interactor     ↓ Services / Data

Router handles:
- navigation

---

# Responsibilities

## View

Responsible for:
- rendering UI
- forwarding interactions

---

## Presenter

Responsible for:
- presentation coordination
- formatting
- UI orchestration

---

## Interactor

Responsible for:
- business logic
- domain workflows

---

## Router

Responsible for:
- navigation
- screen transitions

---

## Entity

Represents:
- domain models

---

# Production Perspective

VIPER improves:
- isolation
- testability
- explicit ownership

but also increases:
- complexity
- boilerplate
- file count

---

# Common Production Mistake

Overengineering small apps using:
- excessive abstraction
- too many layers

---

# Better Approach

Architecture complexity should match:
- app complexity
- team size
- scalability needs

---

# Interview Questions

### Why was VIPER created?

To improve modularity and separation of concerns in large applications.

---

### What is the downside of VIPER?

It introduces significant complexity and boilerplate.

---

### When does VIPER make more sense?

Large complex apps with strong modularity and testing requirements.

---

# Clean Architecture

Clean Architecture emphasizes:
- dependency direction
- isolation
- domain ownership

---

# Core Principle

Business logic should NOT depend on:
- UI frameworks
- databases
- networking frameworks

---

# Dependency Direction

text id="jlwmu5" UI  ↓ Presentation  ↓ Domain  ↓ Data

Dependencies flow inward.

---

# Why This Matters

Business logic becomes:
- testable
- reusable
- framework-independent

---

# Domain Layer

The domain layer contains:
- business rules
- use cases
- core workflows

---

# Production Perspective

Clean Architecture improves:
- scalability
- maintainability
- long-term flexibility

but may increase:
- abstraction complexity

---

# Common Production Mistake

Blindly implementing excessive layers without actual complexity needs.

---

# Better Approach

Use architectural boundaries intentionally.

---

# Interview Questions

### What is the core idea behind Clean Architecture?

Business logic should remain isolated from frameworks and infrastructure.

---

### Why does dependency direction matter?

Because inward dependencies improve isolation and maintainability.

---

### What is a downside of Clean Architecture?

It may introduce abstraction and boilerplate overhead.

---

# Coordinator Pattern

The Coordinator pattern separates:
- navigation
from:
- view controllers

This is VERY common in iOS interviews.

---

# Why Coordinators Matter

Navigation logic often becomes:
- duplicated
- tightly coupled
- difficult to maintain

inside view controllers.

---

# Bad Example

text id="jlwmp0" ViewController     ↓ pushViewController     ↓ modal presentation     ↓ deep links

Navigation mixed everywhere.

---

# Better Approach

text id="jlwma8" Coordinator     ↓ Controls navigation

---

# Example

swift id="jlwms6" protocol Coordinator {     func start() }

---

# Production Perspective

Coordinators improve:
- navigation ownership
- testability
- flow management
- deep linking support

---

# Common Production Mistake

Embedding:
- navigation
- routing
- presentation decisions

inside:
- views
- view models

---

# Better Approach

Centralize navigation ownership.

---

# Interview Questions

### Why does the Coordinator pattern exist?

To separate navigation responsibilities from UI controllers.

---

### Why can navigation inside view controllers become problematic?

Because it creates tight coupling and difficult flow management.

---

# Dependency Injection

Dependency Injection (DI) is one of the MOST important architecture topics.

---

# Why DI Matters

Without DI:
systems become:
- tightly coupled
- difficult to test
- difficult to replace

---

# Bad Example

swift id="jlwmg1" let api = RealAPIClient()

hardcoded inside business logic.

---

# Better Approach

swift id="jlwmt7" init(api: APIClient)

---

# Why This Is Better

Now:
- mocks
- previews
- alternate implementations
- tests

become easy.

---

# Constructor Injection

Most preferred approach:

swift id="jlwmy3" init(service: UserService)

---

# Why Constructor Injection Is Good

Dependencies become:
- explicit
- immutable
- visible

---

# Production Perspective

DI improves:
- modularity
- testing
- ownership clarity
- flexibility

---

# Common Production Mistake

Using giant:
- global singletons

for everything.

---

# Problems With Global State

Global systems create:
- hidden dependencies
- difficult testing
- unpredictable coupling

---

# Better Approach

Prefer:
- explicit injection
- scoped ownership

---

# Interview Questions

### Why is dependency injection important?

Because it improves testing, modularity, and flexibility.

---

### Why are global singletons dangerous?

Because they create hidden coupling and difficult testing environments.

---

### Why is constructor injection often preferred?

Because dependencies remain explicit and immutable.

---

# Service Layers

Service layers encapsulate:
- networking
- persistence
- external systems

---

# Why Service Layers Matter

Without services:
business logic becomes tightly coupled to:
- APIs
- databases
- frameworks

---

# Example

swift id="jlwmd0" protocol UserService {     func fetchUsers() async throws }

---

# Production Perspective

Service abstraction improves:
- mocking
- modularity
- backend replacement
- testing

---

# Common Production Mistake

Calling:
- URLSession
- CoreData
- persistence frameworks

directly from views.

---

# Better Approach

Views communicate through:
- services
- view models
- abstractions

---

# Interview Questions

### Why are service layers useful?

Because they isolate infrastructure and external system interactions.

---

### Why should views avoid direct networking calls?

Because separation improves maintainability and testability.

---

# Repository Pattern

Repositories abstract:
- data sources

---

# Why Repositories Matter

Apps may load data from:
- APIs
- cache
- database
- offline storage

Repositories unify access.

---

# Example

swift id="jlwmu2" protocol UserRepository {     func users() async throws         -> [User] }

---

# Production Perspective

Repositories simplify:
- caching
- offline support
- source switching

---

# Common Production Mistake

Scattering:
- persistence logic
- networking decisions
- cache coordination

everywhere.

---

# Better Approach

Centralize data access coordination.

---

# Interview Questions

### Why does the Repository pattern exist?

To abstract and unify multiple data sources behind a single interface.

---

### Why are repositories useful for offline support?

Because they coordinate cache and remote data consistently.

---

# Protocol Abstraction

Protocols are foundational for scalable Swift architecture.

---

# Why Protocols Matter

Protocols improve:
- decoupling
- testing
- modularity
- replaceability

---

# Example

swift id="jlwmp9" protocol AnalyticsService {     func track(event: Event) }

---

# Production Perspective

Protocol-driven architecture improves:
- isolation
- mocking
- scalability

---

# Common Production Mistake

Creating protocols for EVERYTHING unnecessarily.

This creates:
- abstraction overload
- unnecessary complexity

---

# Better Approach

Abstract:
- volatile systems
- external dependencies
- replaceable boundaries

not:
- every trivial type.

---

# Interview Questions

### Why are protocols important in Swift architecture?

Because they improve decoupling and testability.

---

### Why can excessive abstraction become dangerous?

Because unnecessary complexity reduces readability and maintainability.

---

# Modularity

Modularity separates applications into:
- independent components
- reusable features
- isolated systems

This is one of the MOST important large-scale architecture concepts.

---

# Why Modularity Matters

As applications grow:
- compile times increase
- ownership becomes unclear
- coupling increases
- onboarding slows
- feature interactions explode

Modularity helps control complexity.

---

# Example Structure

text id="jlwma1" Core/ Networking/ Analytics/ Features/ UIComponents/

---

# Feature Modules

Large apps often isolate:
- feeds
- chat
- auth
- onboarding

into separate modules.

---

# Production Perspective

Modularity improves:
- team scalability
- compile times
- ownership
- testing
- reuse

---

# Common Production Mistake

Building:
- gigantic monolithic apps

where:
everything imports everything.

---

# Problems With Monoliths

Monoliths create:
- tight coupling
- slow builds
- difficult ownership
- risky changes

---

# Better Approach

Define:
- explicit module boundaries
- dependency rules
- ownership clarity

---

# Swift Package Manager

SPM strongly supports:
- modular development

---

# Production Perspective

Modern iOS architectures increasingly rely on:
- feature modularization
- package-based development

---

# Interview Questions

### Why is modularity important?

Because large applications become difficult to scale and maintain without isolation boundaries.

---

### What problems occur in monolithic architectures?

Tight coupling, slow builds, and unclear ownership.

---

### How does modularity improve teams?

It improves ownership clarity and reduces cross-team conflicts.

---

# Feature Isolation

Feature isolation means:
- features own their own logic
- dependencies remain controlled
- side effects remain localized

---

# Why Feature Isolation Matters

Without isolation:
changes in one area may:
- break unrelated features
- create hidden coupling
- increase regression risk

---

# Example

Bad:
text id="jlwmu8" Feed feature imports Chat feature Chat imports Profile Profile imports Feed

Circular dependency chaos.

---

# Better Approach

Features communicate through:
- protocols
- shared abstractions
- event systems

---

# Production Perspective

Strong feature isolation improves:
- scalability
- release safety
- maintainability

---

# Common Production Mistake

Allowing arbitrary feature-to-feature imports.

---

# Better Approach

Enforce:
- dependency direction
- architecture boundaries

---

# Interview Questions

### Why is feature isolation important?

Because uncontrolled dependencies create scalability and maintenance problems.

---

### Why are circular dependencies dangerous?

Because they create fragile tightly coupled systems.

---

# SOLID Principles

SOLID principles guide:
- maintainable object-oriented design

---

# S — Single Responsibility Principle

A type should have:
# one reason to change

---

# Bad Example

text id="jlwmp4" ViewModel     ↓ Networking     ↓ Persistence     ↓ Analytics     ↓ Rendering

Too many responsibilities.

---

# Better Approach

Separate:
- networking
- analytics
- persistence
- presentation

---

# O — Open/Closed Principle

Systems should be:
- open for extension
- closed for modification

---

# Why This Matters

New behavior should ideally avoid:
- rewriting stable code

---

# L — Liskov Substitution Principle

Subtypes should behave correctly when substituted.

---

# I — Interface Segregation Principle

Avoid giant protocols.

Prefer:
- smaller focused abstractions

---

# D — Dependency Inversion Principle

High-level systems should depend on:
- abstractions
NOT:
- concrete implementations

---

# Production Perspective

SOLID helps reduce:
- coupling
- rigidity
- fragile architectures

---

# Common Production Mistake

Blindly applying SOLID everywhere with excessive abstraction.

---

# Better Approach

Use principles pragmatically.

---

# Interview Questions

### What does Single Responsibility Principle mean?

A type should have one primary responsibility and one reason to change.

---

### Why is dependency inversion important?

Because abstractions reduce coupling between high-level and low-level systems.

---

### Why can giant protocols become problematic?

Because they increase coupling and reduce flexibility.

---

# State Ownership

State ownership is one of the MOST important SwiftUI and architecture concepts.

---

# Why Ownership Matters

Unclear ownership creates:
- invalid rendering
- duplicated state
- race conditions
- stale UI
- synchronization bugs

---

# Important Mental Model

Every piece of state should have:
# a clear owner

---

# Example

swift id="jlwmt6" @StateObject

owns:
- long-lived observable state

---

# Example

swift id="jlwmd3" @Binding

shares:
- externally owned state

---

# Production Perspective

Many SwiftUI bugs are actually:
- ownership bugs

NOT:
- rendering bugs

---

# Common Production Mistake

Duplicating the same state in:
- multiple views
- multiple view models
- caches
- services

---

# Better Approach

Define:
- single source of truth

---

# Interview Questions

### Why is state ownership important?

Because unclear ownership creates synchronization and rendering problems.

---

### What does “single source of truth” mean?

One authoritative owner controls a piece of state consistently.

---

# Navigation Architecture

Navigation becomes complex as apps scale.

---

# Why Navigation Architecture Matters

Large apps involve:
- deep linking
- onboarding
- authentication gates
- nested flows
- modal coordination

---

# Common Navigation Problems

- duplicated navigation logic
- deeply nested presentation
- hidden transitions
- inconsistent routing

---

# Production Perspective

Navigation systems should emphasize:
- predictability
- ownership
- centralized coordination

---

# Common Patterns

- Coordinators
- Routers
- NavigationStack
- Deep link managers

---

# SwiftUI Navigation

Modern SwiftUI often uses:
- NavigationStack
- NavigationPath

---

# Example

swift id="jlwmy5" NavigationStack { }

---

# Common Production Mistake

Embedding navigation decisions inside:
- reusable components
- business logic
- data layers

---

# Better Approach

Centralize navigation ownership.

---

# Interview Questions

### Why does navigation become difficult in large apps?

Because flows, deep links, and feature interactions increase rapidly.

---

### Why should navigation ownership remain centralized?

Because centralized flows improve predictability and maintainability.

---

# SwiftUI Architecture

SwiftUI changes architecture significantly because of:
- declarative rendering
- state-driven UI
- reactive updates

---

# Important SwiftUI Principle

UI should derive from:
# state

---

# Typical SwiftUI Flow

text id="jlwms2" State changes     ↓ View invalidates     ↓ SwiftUI re-renders

---

# Why This Matters

SwiftUI architectures focus heavily on:
- ownership
- identity
- invalidation
- observable state

---

# Common SwiftUI Components

- View
- ViewModel
- Service
- Environment
- Observable state

---

# Production Perspective

Good SwiftUI architectures avoid:
- excessive logic in views
- duplicated ownership
- unstable identity

---

# Common Production Mistake

Launching async work repeatedly from:
swift id="jlwma9" body

---

# Better Approach

Use:
- lifecycle-aware tasks
- structured ownership
- explicit state models

---

# Interview Questions

### Why is state important in SwiftUI architecture?

Because SwiftUI rendering derives from observable state changes.

---

### What commonly breaks SwiftUI architecture?

Unclear ownership, unstable identity, and duplicated state.

---

# UIKit Architecture

UIKit architecture differs significantly from SwiftUI.

---

# UIKit Is Lifecycle-Driven

UIKit focuses heavily on:
- imperative updates
- lifecycle callbacks
- controller coordination

---

# Common UIKit Patterns

- MVC
- MVVM
- Coordinator
- VIPER

---

# Production Perspective

UIKit often requires more manual coordination for:
- state
- rendering
- navigation

---

# Common Production Mistake

Massive ViewControllers.

---

# Better Approach

Extract:
- services
- presentation logic
- navigation
- business coordination

outside controllers.

---

# Interview Questions

### Why do UIKit apps often develop Massive ViewControllers?

Because lifecycle, networking, rendering, and navigation frequently accumulate together.

---

### How can UIKit architectures remain scalable?

By separating concerns and extracting responsibilities into services and coordinators.

---

# Async Architecture

Modern iOS apps are heavily asynchronous.

Architecture must properly handle:
- concurrency
- task ownership
- cancellation
- synchronization
- lifecycle coordination

---

# Why Async Architecture Matters

Async systems introduce:
- race conditions
- lifecycle complexity
- stale updates
- ownership ambiguity
- cancellation problems

---

# Important Mental Model

Concurrency is fundamentally:
# coordination complexity

NOT:
# syntax complexity

---

# Production Perspective

Most async bugs involve:
- ownership
- lifecycle
- synchronization

NOT:
- async/await syntax itself

---

# Common Async Workflows

- networking
- uploads/downloads
- caching
- realtime updates
- persistence
- background processing

---

# Structured Concurrency

Swift Concurrency strongly encourages:
- structured task ownership

---

# Example

swift id="jlwma2" Task {     await viewModel.load() }

---

# Why Structured Concurrency Matters

Structured ownership improves:
- cancellation
- lifecycle safety
- predictability

---

# Common Production Mistake

Launching detached tasks everywhere.

Example:

swift id="jlwmp3" Task.detached { }

without ownership reasoning.

---

# Problems

Detached work may:
- outlive screens
- leak work
- create stale updates
- ignore cancellation

---

# Better Approach

Tie async work to:
- feature ownership
- lifecycle boundaries
- structured task hierarchies

---

# Cancellation

Cancellation is one of the MOST important async concepts.

---

# Why Cancellation Matters

Users constantly:
- navigate away
- refresh screens
- interrupt workflows

---

# Production Perspective

Async systems should assume:
# interruption is normal

---

# Example

swift id="jlwmu6" try Task.checkCancellation()

---

# Common Production Mistake

Ignoring cancellation completely.

This wastes:
- CPU
- bandwidth
- memory
- battery

---

# Better Approach

Design:
- cancellation-aware workflows
- interruptible tasks
- lifecycle-bound work

---

# Async State Coordination

Async workflows frequently update:
- observable state
- UI rendering
- caches

---

# Important Risk

Concurrent updates may create:
- invalid state
- race conditions
- stale rendering

---

# Better Approach

Use:
- actors
- MainActor
- ownership isolation
- state boundaries

---

# Production Perspective

Async architecture is fundamentally about:
- safe coordination

---

# Interview Questions

### Why is async architecture difficult?

Because concurrency introduces coordination, lifecycle, and synchronization complexity.

---

### Why should async work support cancellation?

Because mobile users constantly interrupt workflows and navigation flows.

---

### Why are detached tasks dangerous?

Because unowned work may outlive lifecycles and create stale updates.

---

# Caching Architecture

Caching architecture coordinates:
- memory
- disk
- network
- synchronization

---

# Why Caching Matters

Good caching improves:
- responsiveness
- offline support
- scalability
- battery efficiency

---

# Multi-Layer Caching

Production systems often use:
- memory cache
- disk cache
- remote APIs

together.

---

# Example

text id="jlwmd1" Memory Cache     ↓ Disk Cache     ↓ Remote API

---

# Why Multi-Layer Caching Matters

Memory is:
- fast
but volatile.

Disk is:
- persistent
but slower.

Network is:
- authoritative
but expensive.

---

# Production Perspective

Strong caching systems balance:
- freshness
- performance
- consistency

---

# Common Production Mistake

Treating cache as:
- always correct
or:
- completely disposable

---

# Better Approach

Define:
- cache policies
- expiration
- invalidation rules
- synchronization strategies

---

# Cache Invalidation

One of the HARDEST production problems.

---

# Why Invalidation Is Difficult

Data may:
- change remotely
- conflict locally
- become stale
- arrive out of order

---

# Production Perspective

Good cache architecture requires:
- ownership clarity
- synchronization
- consistency reasoning

---

# Interview Questions

### Why is caching architecture important?

Because caching strongly affects performance, responsiveness, and offline behavior.

---

### Why is cache invalidation difficult?

Because maintaining consistency between local and remote data is complex.

---

### Why are multi-layer caches useful?

Because different storage layers optimize different performance tradeoffs.

---

# Scalability Thinking

Architecture should scale with:
- features
- engineers
- async complexity
- product growth

---

# Why Scalability Matters

Architectures that work for:
- 2 screens

may collapse at:
- 200 screens

---

# Common Scaling Problems

- giant shared state
- circular dependencies
- feature coupling
- duplicated logic
- massive build times
- unclear ownership

---

# Production Perspective

Scalable systems emphasize:
- isolation
- boundaries
- explicit ownership
- modularity

---

# Important Mental Model

Scalable architecture optimizes for:
- future maintainability

NOT:
- shortest initial implementation

---

# Common Production Mistake

Over-centralizing EVERYTHING.

Example:
text id="jlwmu0" MegaAppManager

controlling:
- networking
- analytics
- auth
- navigation
- cache
- persistence

---

# Problems

This creates:
- coupling
- fragility
- difficult ownership
- scaling bottlenecks

---

# Better Approach

Distribute responsibility intentionally.

---

# Interview Questions

### Why does architecture need to scale?

Because product complexity and engineering complexity grow continuously.

---

### Why are giant centralized managers dangerous?

Because they create tight coupling and ownership bottlenecks.

---

# Testing & Architecture

Good architecture strongly improves:
- testability

---

# Why Testing Matters

Untestable systems often become:
- fragile
- difficult to refactor
- regression-prone

---

# Architecture Features That Improve Testing

- dependency injection
- protocol abstraction
- modularity
- separation of concerns

---

# Example

swift id="jlwmt4" init(api: APIClient)

allows:
- mock injection
- isolated testing

---

# Production Perspective

Testing quality is often:
# architecture quality

---

# Common Production Mistake

Hardcoding:
- networking
- databases
- global state

inside business logic.

---

# Problems

This creates:
- impossible mocking
- integration-only testing
- brittle systems

---

# Better Approach

Design systems with:
- replaceable boundaries
- injectable dependencies

---

# Interview Questions

### Why does architecture affect testing quality?

Because coupling and hidden dependencies make isolation difficult.

---

### Why does dependency injection improve testing?

Because alternate implementations and mocks become easy to provide.

---

# Architecture Tradeoffs

There is NO universally perfect architecture.

This is VERY important for interviews.

---

# Important Mental Model

Architecture is about:
# tradeoffs

NOT:
# perfection

---

# Example Tradeoffs

MVC:
- simple
- fast
but may:
- scale poorly

VIPER:
- modular
- testable
but:
- complex
- verbose

Clean Architecture:
- scalable
- isolated
but:
- abstract
- boilerplate-heavy

---

# Production Perspective

Good engineers choose architecture based on:
- app complexity
- team size
- scalability needs
- product constraints

---

# Common Production Mistake

Blindly applying:
- trendy architectures
- excessive abstraction
- unnecessary patterns

---

# Better Approach

Architecture should solve:
- real complexity

not:
- hypothetical complexity

---

# Interview Questions

### Why is architecture about tradeoffs?

Because every architecture optimizes some qualities while sacrificing others.

---

### Why can overengineering become dangerous?

Because unnecessary abstraction increases complexity and maintenance cost.

---

### How should architecture decisions be made?

Based on scalability, team needs, and actual product complexity.

---

# Production Architecture Pitfalls

These are VERY important for medium/senior interviews.

---

# Common Pitfalls

- Massive ViewControllers
- giant ViewModels
- hidden dependencies
- global state abuse
- duplicated logic
- circular dependencies
- tight coupling
- unclear ownership
- invalid state synchronization
- navigation chaos
- async lifecycle leaks
- abstraction overload

---

# Example: Massive ViewModel

text id="jlwmy8" ViewModel     ↓ Networking     ↓ Persistence     ↓ Analytics     ↓ Navigation     ↓ Validation     ↓ Rendering

---

# Problems

This creates:
- difficult testing
- poor scalability
- ownership confusion

---

# Better Approach

Split responsibilities into:
- services
- coordinators
- repositories
- focused feature models

---

# Example: Global Shared State

Everything mutates:
text id="jlwma7" AppState.shared

---

# Problems

This creates:
- unpredictable updates
- synchronization issues
- debugging difficulty

---

# Better Approach

Use:
- explicit ownership
- scoped state
- unidirectional flow

---

# Example: Abstraction Overload

10 protocols for trivial logic.

---

# Problems

This creates:
- indirection
- onboarding difficulty
- cognitive overhead

---

# Better Approach

Use abstraction intentionally.

---

# Production Perspective

Most architecture failures come from:
- unclear ownership
- scaling issues
- coupling
- lifecycle problems

NOT:
- missing design patterns

---

# Final Architecture Mental Model

Strong iOS architects think carefully about:
- ownership
- scalability
- lifecycle
- modularity
- isolation
- testability
- async coordination
- navigation
- state management
- maintainability

---

# Final Interview Advice

For architecture interviews:
focus on explaining:
- WHY boundaries exist
- HOW systems scale
- WHY ownership matters
- HOW async coordination affects architecture
- WHAT tradeoffs exist

not:
- memorizing pattern definitions

---

# Example

Weak answer:

> “MVVM separates logic from UI.”

Strong answer:

> “MVVM improves separation between rendering and presentation state, which scales especially well with SwiftUI’s reactive rendering model and improves testing and ownership clarity.”

That reasoning depth is what interviewers usually look for.

---

# Final Architecture Summary

The MOST important recurring architecture interview themes are:

- separation of concerns
- MVVM
- dependency injection
- modularity
- ownership
- scalability
- state management
- navigation coordination
- async architecture
- testing
- protocol abstraction
- lifecycle management
- tradeoffs
- maintainability

Understanding:
- not only WHAT patterns exist
but:
- WHY systems become difficult to scale
- HOW ownership affects architecture
- WHY async workflows complicate design
- HOW modularity improves teams
- WHY architecture is fundamentally about tradeoffs
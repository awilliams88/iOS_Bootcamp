# Swift Foundations — Interview Revision Guide

### Contents

- [Variables and Constants](#variables-and-constants)
- [Stored vs Computed Properties](#stored-vs-computed-properties)
- [Property Observers](#property-observers)
- [Lazy Properties](#lazy-properties)
- [Optionals](#optionals)
- [Struct vs Class](#struct-vs-class)
- [Value vs Reference Semantics](#value-vs-reference-semantics)
- [Copy-on-Write](#copy-on-write)
- [Initialization](#initialization)
- [ARC](#arc)
- [Retain Cycles](#retain-cycles)
- [weak vs unowned](#weak-vs-unowned)
- [Closures](#closures)
- [Escaping Closures](#escaping-closures)
- [Capture Lists](#capture-lists)
- [Protocols](#protocols)
- [Protocol Extensions](#protocol-extensions)
- [Protocol-Oriented Programming](#protocol-oriented-programming)
- [Generics](#generics)
- [Associated Types](#associated-types)
- [some vs any](#some-vs-any)
- [Enums](#enums)
- [Error Handling](#error-handling)
- [Result Type](#result-type)
- [Extensions](#extensions)
- [Access Control](#access-control)
- [static vs class](#static-vs-class)
- [mutating Keyword](#mutating-keyword)
- [Type Casting](#type-casting)
- [Any vs AnyObject](#any-vs-anyobject)
- [Equatable / Hashable / Identifiable](#equatable--hashable--identifiable)
- [Collections Performance](#collections-performance)
- [Higher-Order Functions](#higher-order-functions)
- [Dependency Injection](#dependency-injection)
- [Composition vs Inheritance](#composition-vs-inheritance)
- [SOLID Principles](#solid-principles)
- [Property Wrappers](#property-wrappers)
- [Codable](#codable)
- [Stack vs Heap](#stack-vs-heap)
- [Dispatch Mechanisms](#dispatch-mechanisms)
- [final Keyword](#final-keyword)
- [Advanced Swift Topics](#advanced-swift-topics)

---

# Variables and Constants

```swift
let username = "Arpit"
var retryCount = 0
```

Swift strongly encourages immutability by default.

## Why Immutability Matters

Immutable values:
- reduce side effects
- improve predictability
- simplify debugging
- improve thread safety
- reduce accidental mutation

A large percentage of production bugs happen because state changes unexpectedly.

Examples:
- invalid UI state
- race conditions
- synchronization bugs
- accidental mutation
- inconsistent data flows

## Production Perspective

Modern Swift architecture strongly prefers immutable models.

For example:

```swift
struct User {
    let id: UUID
    let name: String
}
```

This model is predictable because its state cannot mutate unexpectedly after creation.

This becomes especially important in:
- concurrency
- SwiftUI rendering
- caching
- diffing systems
- networking layers

## Common Mistake

Using `var` everywhere by default.

This creates:
- unpredictable mutation
- harder debugging
- synchronization complexity
- architecture instability

## Interview Questions

### Why does Swift encourage `let`?

Swift encourages immutability because immutable systems are easier to reason about, safer under concurrency, and less prone to accidental mutation bugs.

### When should you use `var`?

Use `var` only when mutation is genuinely required, such as:
- counters
- mutable collections
- UI state
- temporary mutable buffers

### Why is immutability important in production systems?

Immutability improves predictability, debugging, architecture stability, and thread safety.

---

# Stored vs Computed Properties

## Stored Properties

Stored properties occupy actual memory.

```swift
struct User {
    var name: String
}
```

## Computed Properties

Computed properties derive values dynamically.

```swift
var fullName: String {
    "\(firstName) \(lastName)"
}
```

## Why Computed Properties Matter

Computed properties help avoid duplicated state.

Bad approach:

```swift
var firstName: String
var lastName: String
var fullName: String
```

This introduces synchronization problems because:
- fullName may become outdated
- state becomes duplicated
- consistency becomes harder

Better approach:

```swift
var fullName: String {
    "\(firstName) \(lastName)"
}
```

Now there is:
- one source of truth
- no synchronization issue
- cleaner architecture

## Dangerous Pattern

```swift
var expensiveUsers: [User] {
    performHeavyTransformation()
}
```

Computed properties may execute frequently.

Heavy logic inside them can:
- hurt performance
- block rendering
- increase CPU usage

## Better Production Approach

Move expensive work into:
- services
- caching
- async workflows
- background processing
- memoization systems

## Interview Questions

### Why are computed properties useful?

They derive values dynamically without duplicating state, improving consistency and reducing synchronization bugs.

### Why should computed properties remain lightweight?

Because they may execute frequently, especially during SwiftUI rendering or repeated access.

### When should values be stored instead of computed?

When computation is expensive, repeatedly needed, or when caching improves performance.

### What is a common mistake with computed properties?

Putting networking, database access, or expensive transformations directly inside them.

---

# Property Observers

Swift provides:
- willSet
- didSet

```swift
var username: String = "" {
    didSet {
        print(username)
    }
}
```

## Common Use Cases

- analytics
- validation
- logging
- UI synchronization
- triggering updates

## Example

```swift
var searchQuery: String = "" {
    didSet {
        performSearch()
    }
}
```

## Production Perspective

Property observers are useful for lightweight side effects.

However, large business workflows inside observers quickly become difficult to maintain.

## Dangerous Pattern

```swift
didSet {
    performNetworking()
    updateDatabase()
    uploadAnalytics()
}
```

This tightly couples mutation to large workflows.

## Better Approach

Keep observers lightweight.

Move complex behavior into:
- services
- reducers
- view models
- async pipelines

## Interview Questions

### Difference between willSet and didSet?

`willSet` executes before mutation.
`didSet` executes after mutation.

### What is a common mistake with property observers?

Putting heavy business logic or networking inside them.

### Why can property observers become dangerous in large systems?

Because they create hidden side effects and tightly coupled mutation flows.

---

# Lazy Properties

Lazy properties initialize only when first accessed.

```swift
lazy var formatter: DateFormatter = {
    let formatter = DateFormatter()
    formatter.dateStyle = .medium
    return formatter
}()
```

## Why Lazy Properties Matter

Some objects are expensive to create.

Examples:
- DateFormatter
- image processors
- database connections
- caches
- parsers

## Production Perspective

Lazy initialization improves:
- startup performance
- memory efficiency
- allocation cost

This is extremely useful in:
- large apps
- rendering-heavy screens
- image processing systems
- formatting systems

## Common Pitfall

Lazy properties are not inherently thread-safe.

Concurrent access can create initialization races if not handled carefully.

## Interview Questions

### Why are lazy properties useful?

They delay expensive initialization until the value is actually needed.

### Why are DateFormatters commonly lazy?

DateFormatter creation is relatively expensive.

### When should lazy properties be avoided?

When deterministic eager initialization is required or thread-safety becomes problematic.

---

# Optionals

Optionals explicitly represent absence.

```swift
String?
```

Swift forces developers to safely handle missing values.

This dramatically reduces runtime crashes compared to unsafe null systems.

## Safe Unwrapping

### if let

```swift
if let user = user {
    print(user.name)
}
```

### guard let

```swift
guard let user = user else {
    return
}
```

### Optional Chaining

```swift
user.profile?.avatarURL
```

### Nil Coalescing

```swift
let displayName = username ?? "Guest"
```

### Force Unwrapping

```swift
let value = optional!
```

Dangerous because it may crash.

## Implicitly Unwrapped Optional

```swift
@IBOutlet weak var label: UILabel!
```

Useful when initialization timing guarantees existence.

## Production Perspective

Optionals are one of Swift’s most important safety features.

They force developers to:
- acknowledge absence
- model uncertainty explicitly
- avoid unsafe assumptions

## Common Pitfalls

### Excessive Force Unwrapping

```swift
user!.profile!.avatar!
```

This creates runtime crash risk.

### Better Approach

```swift
guard let profile = user?.profile else {
    return
}
```

## Interview Questions

### Why are optionals important in Swift?

They force developers to explicitly handle missing values instead of silently allowing unsafe null access.

### Difference between `if let` and `guard let`?

`guard let` keeps the happy path cleaner and reduces nesting.

### Why should force unwraps generally be avoided?

Because they introduce runtime crash risk.

### When are implicitly unwrapped optionals acceptable?

Mostly for UIKit lifecycle-driven properties like IBOutlets.

---

# Struct vs Class

## Struct

Structs are value types.

This means:
when you assign a struct to another variable,
Swift creates a logical copy.

```swift id="q4x0by"
struct User {
    var name: String
}

var first = User(name: "Arpit")
var second = first

second.name = "John"

print(first.name)
```

Output:

```swift id="nwb7xk"
Arpit
```

Changing `second` does not affect `first`.

Each variable logically owns its own value.

---

## Class

Classes are reference types.

Multiple variables may reference the same object instance.

```swift id="lh3u89"
final class User {
    var name: String

    init(name: String) {
        self.name = name
    }
}

let first = User(name: "Arpit")
let second = first

second.name = "John"

print(first.name)
```

Output:

```swift id="24lj1m"
John
```

Both variables reference the same object.

---

# Why Swift Prefers Structs

Swift strongly encourages value semantics because they:
- reduce unintended side effects
- improve predictability
- simplify concurrency
- reduce synchronization complexity
- make state management easier

This becomes extremely important in:
- SwiftUI
- async systems
- reducers
- state containers
- rendering systems

---

# Memory Discussion

## Structs

Structs are lightweight value containers.

Small structs are often stack-friendly.

Structs avoid shared mutable state by default.

---

## Classes

Classes allocate shared heap storage.

Heap allocation introduces:
- ARC overhead
- reference counting
- shared ownership complexity
- mutation coordination problems

---

# Identity vs Equality

Classes support identity comparison.

```swift id="x9h95j"
first === second
```

This checks whether both variables reference the same object.

Structs do not have identity.

---

# When Should You Use Structs?

Prefer structs when:
- value semantics are desired
- mutation should remain isolated
- inheritance is unnecessary
- lightweight models are needed
- thread safety matters

Examples:
- models
- view state
- request objects
- configuration values

---

# When Should You Use Classes?

Use classes when:
- identity matters
- shared mutable state is required
- lifecycle management matters
- Objective-C interoperability is needed
- inheritance is required

Examples:
- coordinators
- managers
- services
- shared stores
- UIKit controllers

---

# Common Production Mistake

Using classes everywhere.

This often creates:
- unintended shared state
- synchronization problems
- retain cycles
- mutation bugs
- harder debugging

---

# Production Perspective

One of Swift’s biggest architectural advantages is its strong support for value semantics.

Modern Swift architecture heavily relies on:
- structs
- immutable state
- predictable ownership
- isolated mutation

This is especially visible in:
- SwiftUI
- Redux-style architectures
- unidirectional data flow systems

---

# Interview Questions

### Why does Swift prefer structs by default?

Structs use value semantics, which reduce unintended shared mutation and improve predictability. This makes systems easier to reason about and safer under concurrency.

---

### When should you use classes instead of structs?

Use classes when identity, shared ownership, lifecycle management, inheritance, or Objective-C interoperability is required.

---

### Why are structs often safer for concurrency?

Because values are logically copied instead of being shared mutably across execution contexts.

---

### What is the biggest architectural advantage of value semantics?

Value semantics reduce hidden side effects and make state transitions more predictable.

---

# Value vs Reference Semantics

This is one of the MOST important Swift interview topics.

Many architectural discussions eventually connect back to:
- ownership
- mutation
- memory behavior

---

# Value Semantics

Each variable logically owns its own value.

Mutation affects only that value.

```swift id="i9h5ww"
struct Counter {
    var value: Int
}

var first = Counter(value: 0)
var second = first

second.value += 1
```

`first` remains unchanged.

---

# Reference Semantics

Multiple variables may reference shared mutable state.

```swift id="uq3h0d"
final class Counter {
    var value = 0
}

let first = Counter()
let second = first

second.value += 1
```

Now both references observe the mutation.

---

# Why Shared Mutable State Becomes Dangerous

Shared mutable state creates:
- race conditions
- synchronization complexity
- unexpected side effects
- debugging difficulty
- hidden ownership problems

---

# Production Perspective

Modern Swift architecture strongly favors:
- immutable state
- isolated mutation
- predictable ownership
- value semantics

This dramatically improves:
- debugging
- testing
- concurrency safety
- rendering predictability

---

# Interview Questions

### Why are value semantics powerful?

They reduce unintended side effects and improve predictability.

---

### Why can shared mutable state become dangerous?

Because multiple execution paths may modify the same data unexpectedly.

---

### Why is value semantics important in SwiftUI?

SwiftUI relies heavily on predictable immutable rendering state.

---

# Copy-on-Write

Swift collections:
- Array
- Dictionary
- Set
- String

use Copy-on-Write internally.

---

# Mental Model

Logical copy happens immediately.

Physical duplication happens only when mutation occurs.

```swift id="jlwm4g"
var first = [1, 2, 3]
var second = first
```

Initially:
- both values share storage internally

No real duplication occurs yet.

---

# Mutation Triggers Real Copy

```swift id="2op74j"
second.append(4)
```

Now Swift performs an actual copy because mutation occurs.

---

# Why Copy-on-Write Exists

Copy-on-Write provides:
- value semantics
- memory efficiency
- performance optimization

Without CoW:
large arrays would become extremely expensive to copy repeatedly.

---

# Production Perspective

Copy-on-Write is one of the key reasons Swift collections remain:
- safe
- performant
- scalable

It combines:
- predictable value semantics
with:
- optimized memory usage

---

# Performance Discussion

Copy-on-Write helps avoid:
- unnecessary allocations
- repeated memory duplication
- excessive copying overhead

This is critical in:
- rendering systems
- large datasets
- networking pipelines
- image processing
- caching systems

---

# Common Mistake

Assuming value types always immediately duplicate memory.

This is not true.

Logical copy and physical copy are different concepts.

---

# Custom Copy-on-Write

Advanced Swift systems sometimes implement custom Copy-on-Write storage for:
- performance optimization
- large data structures
- rendering engines
- editor systems

---

# Interview Questions

### If arrays are value types, why are they still efficient?

Swift collections use Copy-on-Write internally, avoiding unnecessary physical duplication until mutation occurs.

---

### What problem does Copy-on-Write solve?

Without CoW, copying large values repeatedly would become expensive in both memory and CPU cost.

---

### Why is Copy-on-Write important in Swift?

It allows Swift to preserve value semantics while remaining highly performant.

---

### Does assigning an array always immediately duplicate memory?

No. Swift initially shares storage internally and performs physical duplication only during mutation.

---

# Initialization

Swift initialization rules are heavily focused on safety.

Objects must become fully initialized before use.

This prevents:
- partially initialized objects
- invalid state access
- undefined memory behavior

---

# Designated Initializers

Designated initializers fully initialize stored properties.

```swift id="2pnuxx"
class User {
    let name: String

    init(name: String) {
        self.name = name
    }
}
```

---

# Convenience Initializers

Convenience initializers provide secondary shortcuts.

```swift id="t7khg7"
class User {
    let name: String

    init(name: String) {
        self.name = name
    }

    convenience init() {
        self.init(name: "Guest")
    }
}
```

---

# Failable Initializers

Failable initializers may return nil.

```swift id="zyw8gc"
struct Password {
    let value: String

    init?(value: String) {
        guard value.count >= 8 else {
            return nil
        }

        self.value = value
    }
}
```

---

# Why Failable Initializers Matter

They prevent invalid object creation.

This improves:
- domain safety
- validation
- architecture correctness

---

# Two-Phase Initialization

Swift class initialization follows two phases:

## Phase 1
Stored properties initialize.

## Phase 2
Customization occurs after full initialization.

This prevents unsafe access to partially initialized objects.

---

# self Usage Rules

You cannot use `self` before stored properties initialize.

Invalid:

```swift id="jymd5t"
class User {
    let name: String

    init(name: String) {
        print(self.name)
        self.name = name
    }
}
```

---

# deinit

Classes may implement deinitializers.

```swift id="5kvnt0"
deinit {
    print("Deallocated")
}
```

Used for:
- cleanup
- observers
- timers
- subscriptions
- unmanaged resources

---

# Production Perspective

Swift initialization rules significantly improve memory safety and predictable object construction.

---

# Common Mistake

Doing heavy async work directly inside initializers.

This often creates:
- lifecycle complexity
- partially configured objects
- difficult testing

---

# Better Production Approach

Keep initialization focused on:
- state setup
- dependency wiring
- lightweight validation

Move heavy workflows into:
- services
- async methods
- lifecycle handlers

---

# Interview Questions

### Difference between designated and convenience initializers?

Designated initializers fully initialize the object. Convenience initializers provide secondary shortcuts.

---

### Why are failable initializers useful?

They prevent invalid object creation safely and enforce domain correctness.

---

### Why does Swift restrict `self` usage during initialization?

To prevent access to partially initialized objects.

---

### What is the purpose of deinit?

Deinit performs cleanup before object deallocation.

---

# ARC (Automatic Reference Counting)

ARC stands for Automatic Reference Counting.

Swift uses ARC to automatically manage memory for:
- classes
- actors

ARC does NOT manage:
- structs
- enums

because value types are not reference-counted objects.

---

# Why ARC Exists

Without memory management systems,
developers would need to manually:
- allocate memory
- release memory
- track ownership

Manual memory management often creates:
- crashes
- dangling pointers
- memory corruption
- leaks

ARC automates object lifetime management.

---

# Strong References

By default, references are strong.

```swift id="ev6m0m"
final class User {
    let name: String

    init(name: String) {
        self.name = name
    }
}

var user: User? = User(name: "Arpit")
```

The object remains alive while strong references exist.

---

# Reference Count Mental Model

Every strong reference increases the reference count.

When the count reaches zero:
- the object deallocates
- memory releases

---

# Deallocation Example

```swift id="mr6o3m"
final class User {
    deinit {
        print("Deallocated")
    }
}

var user: User? = User()
user = nil
```

Output:

```swift id="7grwq0"
Deallocated
```

The object deallocates because no strong references remain.

---

# Production Perspective

ARC is automatic,
but ownership understanding is still critical.

Many production memory issues happen because developers misunderstand:
- ownership
- reference graphs
- closure captures
- async lifecycles

---

# Retain Cycles

Retain cycles happen when objects strongly retain each other.

---

# Example

```swift id="z1j8kj"
final class User {
    var subscription: Subscription?
}

final class Subscription {
    var user: User?
}
```

Now:
- User retains Subscription
- Subscription retains User

Neither object deallocates.

---

# Why Retain Cycles Are Dangerous

Retain cycles create:
- memory leaks
- retained screens
- retained view models
- retained networking workflows
- increasing memory usage

These issues often become severe in long-running applications.

---

# Production Retain Cycle Sources

Most common production retain cycles:
- closures
- delegates
- timers
- async tasks
- Combine subscriptions
- notification observers

---

# Closure Retain Cycle Example

```swift id="jlwmx9"
final class ViewModel {
    var completion: (() -> Void)?

    func setup() {
        completion = {
            self.load()
        }
    }

    func load() {
        print("Loading")
    }
}
```

Problem:
- ViewModel strongly owns closure
- closure strongly captures self

Retain cycle created.

---

# Solution Using weak self

```swift id="0rmrzu"
completion = { [weak self] in
    self?.load()
}
```

Now the closure does not strongly retain the object.

---

# weak vs unowned

---

# weak

Characteristics:
- optional
- automatically becomes nil
- safer
- lifetime uncertain

```swift id="2jwrt0"
weak var delegate: SomeDelegate?
```

---

# unowned

Characteristics:
- non-optional
- assumes object exists
- crashes if invalid

```swift id="r06z74"
unowned let owner: Owner
```

---

# When To Use weak

Use weak when:
- lifetime is uncertain
- object may deallocate independently
- safety matters more

Common examples:
- delegates
- parent-child UI references

---

# When To Use unowned

Use unowned only when:
- lifetime guarantee is extremely clear
- both objects deallocate together
- nil handling is unnecessary

---

# Why weak Is Usually Preferred

`weak` is safer because invalid access becomes nil instead of crashing.

Production apps usually prefer safety unless performance or architecture strongly justifies `unowned`.

---

# Async Retain Cycles

Modern Swift concurrency introduces new retain-cycle patterns.

Example:

```swift id="brtx8s"
Task {
    self.loadData()
}
```

The task strongly captures `self`.

If the task:
- runs long
- retries indefinitely
- never completes

the object may remain retained unexpectedly.

---

# Safer Async Pattern

```swift id="8jmqdi"
Task { [weak self] in
    await self?.loadData()
}
```

---

# Timer Retain Cycles

Timers commonly retain targets strongly.

```swift id="o2g0ut"
timer = Timer.scheduledTimer(
    withTimeInterval: 1,
    repeats: true
) { _ in
    self.update()
}
```

This may retain the owner indefinitely.

---

# Deallocation Debugging

Common production debugging tools:
- Xcode Memory Graph
- Instruments
- Leaks Instrument
- deinit logging

---

# Common Production Symptoms

Retain cycles often appear as:
- screens never deallocating
- memory continuously growing
- duplicate subscriptions
- repeated API calls
- stale view models

---

# Ownership Mental Model

Strong ownership should be intentional.

Ask:
- Who owns this object?
- Who should release this object?
- Should this relationship actually be weak?

Strong Swift engineers think carefully about ownership graphs.

---

# Interview Questions

### What problem does ARC solve?

ARC automatically manages object lifetime using reference counting, reducing manual memory-management complexity.

---

### Why are memory leaks still possible with ARC?

Because retain cycles can still occur when objects strongly retain each other.

---

### Why do closures commonly create retain cycles?

Closures strongly capture referenced objects by default.

---

### Difference between weak and unowned?

`weak` becomes nil automatically when the object deallocates.
`unowned` assumes the object always exists and crashes if invalid.

---

### Why is weak usually safer?

Because invalid access becomes nil instead of crashing.

---

### Why are retain cycles dangerous in production apps?

They cause memory leaks, retained screens, increasing memory usage, and lifecycle bugs.

---

### Why are async tasks a new source of retain cycles?

Tasks strongly capture referenced objects and may outlive expected screen lifecycles.

---

# Closures

Closures are executable blocks of code that:
- can be stored
- passed around
- executed later
- capture surrounding context

Closures are one of the MOST important Swift interview topics.

They are deeply integrated into:
- networking
- SwiftUI
- concurrency
- animations
- async workflows
- callbacks
- reactive systems

---

# Basic Closure Syntax

```swift id="a9n8kg"
let greet = {
    print("Hello")
}
```

Calling:

```swift id="o2e7hk"
greet()
```

---

# Closure With Parameters

```swift id="w3dc4x"
let sum = { (a: Int, b: Int) in
    a + b
}
```

---

# Trailing Closure Syntax

Swift supports trailing closures for readability.

```swift id="gv7d13"
performTask {
    print("Done")
}
```

This is extremely common in:
- SwiftUI
- animations
- networking
- async APIs

---

# Closures Capture Values

Closures can capture surrounding variables.

```swift id="fjlwm0"
var counter = 0

let increment = {
    counter += 1
}
```

The closure captures the variable.

---

# Why Capture Behavior Matters

Capture behavior affects:
- ownership
- memory
- lifecycle
- retain cycles
- concurrency safety

This is one of the MOST commonly misunderstood Swift topics.

---

# Capture By Reference Mental Model

Closures usually capture variables by reference semantics for mutable captured state.

This means:
- changes remain visible
- state persists between executions

---

# Escaping Closures

Escaping closures outlive the function scope.

```swift id="7i5d2q"
func fetch(
    completion: @escaping () -> Void
) {
}
```

Networking APIs heavily rely on escaping closures.

---

# Why Escaping Closures Matter

Escaping closures introduce:
- ownership complexity
- lifecycle complexity
- async timing complexity
- retain-cycle risk

---

# Non-Escaping Closures

Non-escaping closures execute before the function returns.

They are:
- simpler
- safer
- more optimizable

---

# Capture Lists

Capture lists explicitly control ownership behavior.

```swift id="fhh1n7"
{ [weak self] in
    self?.load()
}
```

---

# Common Capture Types

## weak self

- optional
- safer
- avoids retain cycle

## unowned self

- non-optional
- unsafe if invalid

---

# Production Perspective

Most real Swift memory leaks involve:
- escaping closures
- async tasks
- Combine pipelines
- timers
- observers

Understanding closure ownership is critical for production iOS engineering.

---

# Autoclosures

Autoclosures automatically wrap expressions into closures lazily.

Example:

```swift id="vlfzls"
func log(
    _ message: @autoclosure () -> String
) {
    print(message())
}
```

---

# Why Autoclosures Matter

They avoid unnecessary evaluation.

Used heavily in:
- assertions
- logging systems
- lazy APIs

---

# Higher-Order Functions

Closures power:
- map
- filter
- reduce
- sorted

Example:

```swift id="5pj11n"
let names = users.map { $0.name }
```

---

# Functional Programming Perspective

Closures enable:
- composability
- declarative transformations
- reduced mutation
- cleaner pipelines

---

# Common Closure Mistakes

## Strongly Capturing self

```swift id="8df3sj"
completion = {
    self.load()
}
```

---

## Heavy Logic Inside Closures

Large closures become:
- difficult to debug
- difficult to test
- difficult to reuse

---

## Retaining Async Workflows

Long-lived closures may unintentionally retain screens and workflows.

---

# Interview Questions

### Why are closures powerful?

Closures allow executable behavior to be passed, stored, and composed dynamically.

---

### Why do closures commonly create memory leaks?

Because closures strongly capture referenced objects by default.

---

### Difference between escaping and non-escaping closures?

Escaping closures outlive the function scope while non-escaping closures execute before the function returns.

---

### Why are escaping closures more dangerous?

Because they introduce lifecycle, ownership, and timing complexity.

---

### Why is `[weak self]` commonly used?

To avoid retain cycles between closures and owning objects.

---

### What problem do capture lists solve?

They provide explicit ownership control for captured values.

---

### Why are closures heavily used in SwiftUI and async programming?

Because modern reactive and asynchronous systems rely heavily on passing executable behavior dynamically.

---

# Protocols

Protocols define behavior contracts.

They describe:
- what something can do
- not how it does it

Protocols are one of the MOST important Swift architecture concepts.

Modern Swift heavily relies on:
- protocols
- abstractions
- protocol-oriented programming
- dependency injection

---

# Basic Protocol Example

```swift id="1m4jgc"
protocol NetworkService {
    func fetchUsers()
}
```

The protocol defines required behavior.

---

# Protocol Conformance

```swift id="4v0k98"
final class APIClient: NetworkService {
    func fetchUsers() {
        print("Fetching")
    }
}
```

The class now conforms to the protocol.

---

# Why Protocols Matter

Protocols improve:
- modularity
- flexibility
- testing
- architecture separation
- dependency inversion

Without protocols:
systems become tightly coupled to concrete implementations.

---

# Production Perspective

Large iOS applications heavily use protocols for:
- networking layers
- repositories
- storage systems
- analytics
- dependency injection
- feature boundaries

Protocols help teams:
- swap implementations
- mock dependencies
- scale architectures safely

---

# Real Production Example

Without protocols:

```swift id="8rwgjh"
final class UserViewModel {
    let api = APIClient()
}
```

Now:
- tightly coupled
- difficult to test
- impossible to mock easily

---

# Better Protocol-Based Design

```swift id="tqk1ob"
protocol NetworkService {
    func fetchUsers()
}

final class UserViewModel {
    let service: NetworkService

    init(service: NetworkService) {
        self.service = service
    }
}
```

Now:
- dependencies are abstracted
- testing becomes easier
- mocking becomes easy
- architecture becomes modular

---

# Protocol Composition

Swift allows combining protocols.

```swift id="sfxh9d"
protocol Readable {
    func read()
}

protocol Writable {
    func write()
}

typealias ReadWrite = Readable & Writable
```

---

# Why Protocol Composition Matters

Composition creates:
- flexible APIs
- smaller abstractions
- better architecture boundaries

This aligns strongly with:
- SOLID principles
- composition-first architecture

---

# Mutating Protocol Functions

Protocols supporting structs may require `mutating`.

```swift id="8u8vq6"
protocol Resettable {
    mutating func reset()
}
```

This allows value mutation for structs.

---

# Protocol Inheritance

Protocols can inherit from other protocols.

```swift id="gt5prl"
protocol Animal {
    func eat()
}

protocol Dog: Animal {
    func bark()
}
```

---

# Associated Types

Protocols may define placeholder types.

```swift id="8n9nxu"
protocol Repository {
    associatedtype Model

    func save(_ model: Model)
}
```

---

# Why Associated Types Matter

They allow protocols to remain generic and reusable.

Without associated types:
many protocol abstractions would become rigid.

---

# Protocol Existentials

```swift id="8nyvsl"
let service: any NetworkService
```

This stores abstract protocol values dynamically.

---

# some vs any

This is increasingly important in modern Swift interviews.

---

# any

Existential type.

```swift id="w2jdn4"
let service: any NetworkService
```

Allows storing heterogeneous protocol conformers.

---

# some

Opaque type.

```swift id="p2c1f2"
func makeView() -> some View
```

The caller does not know the concrete type,
but the compiler still does.

---

# Why some Is More Optimizable

`some` preserves compile-time type information.

This allows:
- specialization
- optimization
- static dispatch opportunities

---

# Why any Has Runtime Cost

Existentials introduce:
- dynamic dispatch
- existential containers
- runtime indirection

This may reduce performance.

---

# Protocol Extensions

Protocols may provide default implementations.

```swift id="f8qgg3"
extension NetworkService {
    func log() {
        print("Logging")
    }
}
```

---

# Why Protocol Extensions Matter

They enable:
- reusable behavior
- shared implementation
- composition-oriented design

Swift heavily encourages protocol extensions.

---

# Protocol-Oriented Programming

Swift strongly favors protocol-oriented design over deep inheritance trees.

---

# Why Swift Encourages POP

Protocol-oriented programming improves:
- flexibility
- modularity
- composition
- testability
- architecture scalability

---

# Composition vs Inheritance

Swift prefers:
- composition
over:
- inheritance-heavy hierarchies

Deep inheritance trees often become:
- tightly coupled
- fragile
- difficult to scale

---

# Witness Tables

Protocols often use witness-table dispatch internally.

This is part of how Swift resolves protocol method calls dynamically.

You do NOT need compiler-level understanding for interviews,
but understanding that protocol dispatch differs from direct concrete dispatch is valuable.

---

# Static vs Dynamic Dispatch

Swift supports multiple dispatch systems:
- static dispatch
- dynamic dispatch
- witness-table dispatch

---

# Why Dispatch Matters

Dispatch behavior affects:
- performance
- optimization
- polymorphism
- runtime flexibility

---

# Static Dispatch

Resolved at compile time.

Usually faster.

Examples:
- structs
- final methods
- many generic specializations

---

# Dynamic Dispatch

Resolved at runtime.

More flexible,
but introduces runtime overhead.

Examples:
- Objective-C interoperability
- overridable class methods

---

# Why final Improves Performance

```swift id="0gl8vz"
final class APIClient {
}
```

The compiler knows subclassing cannot occur.

This enables:
- more aggressive optimization
- static dispatch opportunities
- reduced runtime indirection

---

# Production Perspective

Most production Swift performance issues do NOT come from dispatch alone.

However:
understanding dispatch helps explain:
- optimization
- polymorphism
- architecture tradeoffs

---

# Common Production Mistakes

## Massive Protocol Abstraction Everywhere

Over-abstraction creates:
- unnecessary complexity
- difficult debugging
- architectural overengineering

---

## Huge Protocols

```swift id="uhxdu4"
protocol EverythingService {
    func fetch()
    func save()
    func upload()
    func delete()
}
```

Large protocols violate:
- interface segregation
- clean architecture principles

---

# Better Approach

Prefer:
- smaller focused protocols
- composition
- explicit abstractions

---

# Interview Questions

### Why are protocols important in Swift?

Protocols decouple implementations from abstractions, improving modularity, flexibility, testing, and architecture scalability.

---

### Why are protocols useful for testing?

Protocols allow mocking and dependency injection without depending on concrete implementations.

---

### Why does Swift encourage protocol-oriented programming?

Protocols create more flexible and composable architectures than deep inheritance hierarchies.

---

### Difference between `some` and `any`?

`some` preserves hidden compile-time concrete types while `any` stores existential protocol values dynamically.

---

### Why can `any` introduce performance overhead?

Existentials require runtime indirection and dynamic dispatch behavior.

---

### Why is composition often preferred over inheritance?

Composition reduces tight coupling and creates more maintainable systems.

---

### Why can protocol over-abstraction become problematic?

Excessive abstraction may increase architectural complexity without providing meaningful flexibility.

---

# Generics

Generics allow reusable type-safe abstractions.

Without generics,
developers often duplicate logic repeatedly.

---

# Basic Generic Example

```swift id="7ql8sl"
func compare<T: Equatable>(
    _ first: T,
    _ second: T
) -> Bool {
    first == second
}
```

The function works for:
- Int
- String
- custom Equatable types
- many others

---

# Why Generics Matter

Generics provide:
- reusable abstractions
- compile-time safety
- reduced duplication
- specialization opportunities
- better API design

---

# Production Perspective

Modern Swift frameworks heavily rely on generics:
- SwiftUI
- Combine
- AsyncSequence
- Result
- collections
- networking abstractions

---

# Generic Constraints

Generics may restrict allowed types.

```swift id="w2z6l2"
func printValue<T: CustomStringConvertible>(
    _ value: T
) {
    print(value.description)
}
```

---

# Why Constraints Matter

Constraints allow:
- safer APIs
- richer functionality
- compile-time guarantees

---

# Generic Specialization

Swift often specializes generics at compile time.

This improves:
- performance
- optimization
- static dispatch opportunities

---

# Why Generics Are Often Fast

Unlike many dynamic languages,
Swift generics are heavily optimized.

Generic abstractions often compile down extremely efficiently.

---

# Associated Types vs Generics

Associated types define placeholder types INSIDE protocols.

Generics define placeholder types INSIDE functions/types.

---

# Common Production Mistakes

## Using Any Everywhere

```swift id="x5vv2x"
func process(_ value: Any)
```

This loses:
- type safety
- compile-time guarantees
- optimization opportunities

---

# Better Generic Approach

```swift id="a9ikg4"
func process<T>(_ value: T)
```

---

# Why Strong Type Systems Matter

Strong typing improves:
- correctness
- refactoring safety
- autocomplete
- compiler validation
- architecture reliability

---

# Interview Questions

### Why are generics important?

Generics provide reusable type-safe abstractions without sacrificing compile-time safety.

---

### Why are generics often preferred over Any?

Generics preserve type information and compile-time guarantees.

---

### What problem do generic constraints solve?

Constraints restrict allowed types while enabling richer functionality safely.

---

### Why are Swift generics often performant?

Because Swift heavily optimizes and specializes generic code at compile time.

---

### Difference between associated types and generics?

Associated types belong to protocols while generics belong to functions or concrete types.

---

# Enums

Swift enums are significantly more powerful than enums in many traditional languages.

They are heavily used for:
- state modeling
- navigation
- reducers
- networking state
- domain modeling
- error handling

Enums are one of the MOST important Swift architecture tools.

---

# Basic Enum

```swift id="9ayj8w"
enum Direction {
    case north
    case south
    case east
    case west
}
```

---

# Why Enums Matter

Enums create:
- explicit state
- predictable transitions
- exhaustive handling
- safer architecture

Instead of:
- invalid strings
- magic values
- ambiguous booleans

we model states explicitly.

---

# Associated Values

Enums may store associated data.

```swift id="qcf9pi"
enum APIState {
    case idle
    case loading
    case success(Data)
    case failure(Error)
}
```

---

# Why Associated Values Are Powerful

Associated values allow enums to model real application state cleanly.

This is heavily used in:
- reducers
- SwiftUI state
- async workflows
- networking layers

---

# Exhaustive Switching

Swift forces exhaustive enum handling.

```swift id="5r3sqa"
switch state {
case .idle:
    break

case .loading:
    break

case .success(let data):
    print(data)

case .failure(let error):
    print(error)
}
```

This improves:
- safety
- maintainability
- predictability

---

# Why Exhaustiveness Matters

When adding a new enum case,
the compiler forces all switch statements to handle it.

This prevents:
- forgotten logic
- invalid states
- silent bugs

---

# Raw Values

Enums may contain raw values.

```swift id="m4o8j5"
enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
}
```

---

# Why Raw Values Matter

Useful for:
- networking
- persistence
- interoperability
- serialization

---

# Enums vs Booleans

Bad:

```swift id="9h5h1w"
isLoading = true
hasError = false
```

This creates ambiguous combinations.

Better:

```swift id="bxwq2q"
enum ScreenState {
    case idle
    case loading
    case loaded
    case failed
}
```

Now invalid state combinations become impossible.

---

# Production Perspective

Modern Swift architecture heavily relies on enums because they:
- model state explicitly
- improve predictability
- simplify reducers
- reduce invalid state combinations

Enums are foundational in:
- SwiftUI
- Redux architectures
- async state management
- navigation systems

---

# Common Production Mistakes

## Using Strings Instead of Enums

```swift id="xmm6n6"
let status = "loading"
```

This creates:
- typos
- invalid values
- weak architecture

---

# Better Enum Approach

```swift id="3hlq3u"
enum Status {
    case loading
}
```

---

# Interview Questions

### Why are Swift enums considered powerful?

Swift enums support associated values, exhaustive switching, methods, and rich state modeling.

---

### Why are enums useful for state management?

They prevent invalid state combinations and force explicit handling of every state.

---

### Why is exhaustive switching valuable?

Because the compiler guarantees all states are handled safely.

---

### Why are enums often preferred over booleans or strings?

Enums create explicit predictable state models and reduce invalid combinations.

---

# Error Handling

Swift strongly encourages explicit failure handling.

Unlike many languages,
Swift avoids silently ignoring errors.

---

# throws

Functions may throw errors.

```swift id="ukr15f"
func fetchUsers() throws
```

---

# do-catch

```swift id="4x3w8i"
do {
    try service.fetchUsers()
} catch {
    print(error)
}
```

---

# Why Explicit Error Handling Matters

Explicit error handling improves:
- safety
- predictability
- debugging
- architecture clarity

---

# Typed Errors

Swift errors usually conform to `Error`.

```swift id="v9xhtx"
enum NetworkError: Error {
    case invalidURL
    case unauthorized
    case decodingFailed
}
```

---

# Why Domain Errors Matter

Strongly modeled errors improve:
- debugging
- analytics
- retry logic
- UI messaging
- architecture clarity

---

# Result Type

Swift provides Result for explicit success/failure modeling.

```swift id="zjlwmm"
Result<Data, Error>
```

---

# Example

```swift id="x4cz2i"
func fetch(
    completion: (Result<Data, Error>) -> Void
)
```

---

# Why Result Matters

Result makes:
- async APIs clearer
- failures explicit
- state handling cleaner

---

# Result vs throws

---

# throws

Best for:
- sequential workflows
- synchronous APIs
- async-await workflows

---

# Result

Best for:
- callbacks
- state modeling
- explicit async completion handling

---

# Production Perspective

Modern Swift apps rely heavily on:
- typed errors
- explicit failure handling
- domain error modeling

Good error systems improve:
- observability
- retry logic
- analytics
- debugging
- UX

---

# Common Production Mistakes

## Swallowing Errors

```swift id="yq29c6"
try? performWork()
```

This silently hides failure information.

---

# Better Approach

Handle failures explicitly.

```swift id="b61m7z"
do {
    try performWork()
} catch {
    logger.log(error)
}
```

---

# Error Propagation

Swift allows layered error propagation.

```swift id="l0ujf3"
func load() throws {
    try service.fetch()
}
```

Errors bubble upward cleanly.

---

# Interview Questions

### Why is Swift error handling considered safer?

Because failures must be handled explicitly instead of silently ignored.

---

### Difference between throws and optional returns?

Throws provide detailed failure information while optionals only represent absence.

---

### Why are typed errors important?

Typed errors improve debugging, retry logic, analytics, and architecture clarity.

---

### Why can `try?` become dangerous?

Because it silently discards failure information.

---

### When is Result preferred over throws?

Result is useful for callback-based APIs and explicit async state handling.

---

# Extensions

Extensions allow adding behavior to existing types.

```swift id="x9qg6x"
extension String {
    var isBlank: Bool {
        trimmingCharacters(
            in: .whitespacesAndNewlines
        ).isEmpty
    }
}
```

---

# Why Extensions Matter

Extensions improve:
- organization
- modularity
- readability
- reusable behavior

---

# Common Production Usage

Extensions are heavily used for:
- utilities
- formatting
- helpers
- reusable UI logic
- domain helpers

---

# Extension Organization

Large Swift projects commonly separate functionality using extensions.

Example:

```swift id="rv9y4r"
extension UserViewController: UITableViewDelegate {
}
```

This improves:
- readability
- file organization
- scalability

---

# Limitations

Extensions cannot add:
- stored properties
- custom inheritance

because this would change memory layout unpredictably.

---

# Protocol Conformance via Extensions

```swift id="qefnfh"
extension APIClient: NetworkService {
}
```

This is extremely common in production Swift.

---

# Production Perspective

Extensions help maintain clean scalable code organization in large iOS projects.

---

# Common Mistakes

## Massive Utility Extensions

Huge extensions often become:
- dumping grounds
- hard to navigate
- weakly organized

---

# Better Approach

Prefer:
- focused extensions
- domain organization
- clear responsibilities

---

# Interview Questions

### Why are extensions useful?

Extensions improve organization, modularity, and reusable behavior.

---

### Why can't extensions add stored properties?

Because stored properties would change memory layout unexpectedly.

---

### Why are protocol conformances commonly added via extensions?

It improves organization and separates responsibilities cleanly.

---

# Access Control

Swift provides multiple access-control levels.

- private
- fileprivate
- internal
- public
- open

---

# Why Access Control Matters

Access control protects:
- implementation details
- architecture boundaries
- module safety
- API stability

---

# private

Accessible only within the enclosing scope.

```swift id="k9vfrv"
private let token: String
```

---

# fileprivate

Accessible within the same file.

---

# internal

Default Swift access level.

Accessible within the module.

---

# public

Accessible outside the module,
but cannot be subclassed externally.

---

# open

Allows subclassing and overriding externally.

```swift id="rcti6g"
open class BaseViewController {
}
```

---

# Why open Is More Dangerous

Open APIs become harder to evolve safely because external systems may subclass them unexpectedly.

---

# Production Perspective

Good access control:
- reduces accidental misuse
- protects architecture
- improves maintainability

---

# Common Production Mistakes

## Using public Everywhere

Overexposed APIs create:
- tight coupling
- maintenance problems
- unsafe dependencies

---

# Better Approach

Expose the smallest surface area necessary.

---

# Interview Questions

### Difference between public and open?

`public` allows access externally while `open` also allows subclassing and overriding.

---

### Why is access control important?

It protects implementation details and architecture boundaries.

---

### Why should APIs avoid unnecessary openness?

Because excessive exposure increases coupling and maintenance risk.

---

# static vs class

Swift provides:
- `static`
- `class`

Both define type-level members,
but they behave differently.

---

# static

`static` members belong to the type itself.

They cannot be overridden.

```swift id="k6k14j"
struct API {
    static let baseURL = "https://api.app.com"
}
```

---

# static Functions

```swift id="5mjlwm"
struct Math {
    static func add(
        _ a: Int,
        _ b: Int
    ) -> Int {
        a + b
    }
}
```

Usage:

```swift id="vf8mzr"
Math.add(1, 2)
```

---

# class Functions

`class` members belong to classes
and may be overridden.

```swift id="7rvf6n"
class Animal {
    class func sound() {
        print("Animal")
    }
}
```

Subclass:

```swift id="h5azn5"
class Dog: Animal {
    override class func sound() {
        print("Bark")
    }
}
```

---

# Why This Matters

`static`
= fixed behavior

`class`
= polymorphic behavior

---

# Production Perspective

Use `static` by default unless subclass overriding is intentionally required.

This improves:
- predictability
- optimization opportunities
- architecture clarity

---

# Common Usage

## static

Used heavily for:
- constants
- factories
- utilities
- configuration
- helpers

---

## class

Used when subclass customization is needed.

Usually much less common.

---

# Interview Questions

### Difference between static and class?

`static` members cannot be overridden while `class` members support overriding in subclasses.

---

### Why is static usually preferred?

Because fixed behavior improves predictability and allows better optimization.

---

# mutating Keyword

Structs are value types.

Mutation must therefore be explicit.

---

# Example

```swift id="t5mp4x"
struct Counter {
    var value = 0

    mutating func increment() {
        value += 1
    }
}
```

Without `mutating`,
Swift prevents modification.

---

# Why mutating Exists

Swift wants mutation to remain explicit for value types.

This improves:
- predictability
- ownership clarity
- architecture safety

---

# Production Perspective

Explicit mutation helps developers reason about:
- state transitions
- ownership
- side effects

---

# Common Mistake

Trying to mutate struct properties inside non-mutating methods.

---

# Better Mental Model

Struct mutation potentially changes the entire value.

Swift therefore requires explicit acknowledgment.

---

# Interview Questions

### Why do structs require mutating methods?

Because value-type mutation must remain explicit.

---

### Why is explicit mutation valuable?

It improves predictability and reduces accidental state changes.

---

# Type Casting

Swift provides:
- `is`
- `as?`
- `as!`

for runtime type checking and casting.

---

# is

Checks type compatibility.

```swift id="d2y4ul"
if value is String {
    print("String")
}
```

---

# as?

Safe cast.

```swift id="rqjlwm"
let user = value as? User
```

Returns:
- optional value
- nil on failure

---

# as!

Forced cast.

```swift id="jlwmx4"
let user = value as! User
```

Crashes if invalid.

---

# Why Forced Casts Are Dangerous

Forced casts create:
- runtime crash risk
- unsafe assumptions
- brittle systems

---

# Production Perspective

Production Swift code should strongly prefer:
- compile-time type safety
- safe casts
- predictable architecture

---

# Common Production Mistake

Overusing:
- Any
- forced casting
- runtime assumptions

This weakens architecture significantly.

---

# Better Approach

Prefer:
- protocols
- generics
- typed systems

instead of excessive casting.

---

# Interview Questions

### Difference between as? and as!?

`as?` safely returns nil while `as!` crashes on failure.

---

### Why should forced casts generally be avoided?

Because they introduce runtime crash risk.

---

### Why is excessive casting often considered an architecture smell?

Because it usually indicates weak type modeling.

---

# Any vs AnyObject

---

# Any

Represents:
- any Swift type

Including:
- structs
- enums
- classes
- functions

---

# AnyObject

Represents:
- class instances only

---

# Example

```swift id="2vlr7o"
let value: Any = 5
```

```swift id="j90zcf"
let object: AnyObject = NSString(string: "Hello")
```

---

# Why Excessive Any Is Dangerous

Using `Any` excessively:
- weakens type safety
- increases casting
- increases runtime checks
- reduces compiler guarantees

---

# Production Perspective

Strong Swift architecture prefers:
- concrete types
- protocols
- generics

instead of:
- loosely typed systems

---

# Better Generic Approach

Instead of:

```swift id="jtdbhv"
func process(_ value: Any)
```

Prefer:

```swift id="1gx9x5"
func process<T>(_ value: T)
```

---

# Interview Questions

### Difference between Any and AnyObject?

`Any` represents all Swift types while `AnyObject` represents only class instances.

---

### Why should excessive use of Any be avoided?

Because it weakens compile-time safety and increases runtime complexity.

---

# Equatable / Hashable / Identifiable

These protocols are foundational in modern Swift systems.

---

# Equatable

Allows equality comparison.

```swift id="fblj93"
struct User: Equatable {
    let id: UUID
}
```

```swift id="5j9w0f"
user1 == user2
```

---

# Why Equatable Matters

Equatable improves:
- comparisons
- diffing
- state management
- SwiftUI rendering optimization

---

# Hashable

Supports hashing.

Used heavily in:
- Set
- Dictionary
- diffing systems

```swift id="esjlwm"
struct User: Hashable {
    let id: UUID
}
```

---

# Why Hashable Matters

Hashing enables:
- fast lookup
- identity systems
- efficient collections

---

# Identifiable

Provides stable identity.

```swift id="gm5o4t"
struct User: Identifiable {
    let id: UUID
}
```

---

# Why Identifiable Matters

SwiftUI heavily relies on stable identity for:
- diffing
- animations
- rendering
- list updates

---

# Production Perspective

Stable identity is critical for:
- rendering correctness
- efficient UI updates
- synchronization systems

---

# Common Mistake

Using unstable IDs.

Example:

```swift id="hhs54x"
UUID()
```

generated repeatedly during rendering.

This breaks identity consistency.

---

# Interview Questions

### Why is Hashable important?

It enables efficient hashing-based collections and lookup systems.

---

### Why does SwiftUI rely heavily on Identifiable?

Stable identity helps SwiftUI diff and update UI efficiently.

---

### Why can unstable identity create rendering bugs?

Because diffing systems cannot correctly track object continuity.

---

# Collections Performance

Understanding collection complexity is VERY interview relevant.

---

# Array

Ordered contiguous storage.

Excellent for:
- iteration
- append-heavy workloads
- indexed access

---

# Array Complexity

| Operation | Complexity |
|---|---|
| append | amortized O(1) |
| access by index | O(1) |
| insert at front | O(n) |
| contains | O(n) |

---

# Why Arrays Are Fast

Arrays use contiguous memory storage.

This improves:
- cache locality
- iteration performance
- memory efficiency

---

# Set

Unordered unique collection.

Uses hashing internally.

---

# Set Complexity

| Operation | Complexity |
|---|---|
| insert | average O(1) |
| lookup | average O(1) |
| remove | average O(1) |

---

# Why Sets Are Powerful

Sets provide extremely fast membership lookup.

---

# Dictionary

Key-value storage using hashing.

---

# Dictionary Complexity

| Operation | Complexity |
|---|---|
| lookup | average O(1) |
| insert | average O(1) |
| remove | average O(1) |

---

# Production Perspective

Incorrect collection choices can significantly affect:
- rendering performance
- memory usage
- scalability
- CPU cost

---

# Common Mistakes

## Using Arrays For Frequent Membership Checks

```swift id="9tvc0v"
users.contains(user)
```

Repeated linear scans become expensive.

---

# Better Set Approach

```swift id="wjlwm3"
Set<User>
```

---

# Copy-on-Write Performance

Swift collections use Copy-on-Write internally.

This improves:
- memory efficiency
- copying cost
- scalability

---

# Interview Questions

### Why is Set lookup faster than Array lookup?

Sets use hashing for near constant-time lookup.

---

### Why are arrays extremely fast for iteration?

Because arrays use contiguous memory storage.

---

### Why can inserting at the front of an array become expensive?

Because elements must shift in memory.

---

### Why is collection choice important in production apps?

Incorrect collection choices can significantly affect scalability and performance.

---

# Higher-Order Functions

Swift supports functional transformations like:
- map
- filter
- reduce
- compactMap
- flatMap
- sorted

---

# map

Transforms values.

```swift id="zjlwm0"
let names = users.map { $0.name }
```

---

# filter

Filters values.

```swift id="0khpqo"
let adults = users.filter {
    $0.age >= 18
}
```

---

# reduce

Combines values.

```swift id="yo4rql"
let total = values.reduce(0, +)
```

---

# compactMap

Removes nil values while transforming.

```swift id="4jlwmc"
let ids = strings.compactMap(Int.init)
```

---

# Why Functional Transformations Matter

They improve:
- readability
- composability
- declarative style
- reduced mutation

---

# Production Perspective

Modern Swift heavily favors declarative transformation pipelines.

Especially in:
- SwiftUI
- async systems
- reducers
- reactive pipelines

---

# Common Mistake

Overly complex chained transformations.

```swift id="uwjlwm"
users
    .filter(...)
    .map(...)
    .sorted(...)
    .reduce(...)
```

Large chains may reduce readability.

---

# Better Approach

Break complex pipelines into:
- named steps
- helper methods
- intermediate variables

when readability suffers.

---

# Interview Questions

### Why are higher-order functions valuable?

They create declarative readable transformation pipelines.

---

### Why can excessive chaining become problematic?

Because readability and debugging may suffer.

---

### Why does Swift heavily encourage declarative transformations?

Declarative code often becomes easier to reason about and maintain.

---

# Dependency Injection

Dependency Injection (DI) means:
instead of a class creating its own dependencies,
dependencies are provided externally.

This is one of the MOST important architecture concepts in iOS interviews.

---

# Bad Tight Coupling Example

```swift id="0bjlwm"
final class UserViewModel {
    let api = APIClient()
}
```

Problems:
- tightly coupled
- difficult to test
- impossible to mock cleanly
- hard to swap implementations

---

# Better Dependency Injection Approach

```swift id="rjlwm0"
protocol NetworkService {
    func fetchUsers()
}

final class APIClient: NetworkService {
    func fetchUsers() {
        print("Fetching")
    }
}

final class UserViewModel {
    let service: NetworkService

    init(service: NetworkService) {
        self.service = service
    }
}
```

Now:
- dependencies are abstracted
- implementations are swappable
- testing becomes easier
- architecture becomes modular

---

# Why Dependency Injection Matters

Dependency Injection improves:
- modularity
- flexibility
- testability
- architecture separation
- maintainability

---

# Constructor Injection

Most preferred Swift DI approach.

```swift id="9jlwm2"
init(service: NetworkService)
```

Dependencies become:
- explicit
- predictable
- immutable

---

# Why Constructor Injection Is Preferred

Constructor injection guarantees:
- valid dependency state
- initialization safety
- easier reasoning
- cleaner architecture

---

# Property Injection

```swift id="jlwmx8"
var service: NetworkService?
```

Usually weaker because:
- dependencies become optional
- invalid states become possible
- initialization safety weakens

---

# Service Locator Pattern

Some systems use service locators.

```swift id="8jlwmf"
let service = Container.resolve()
```

---

# Why Service Locators Are Controversial

They hide dependencies implicitly.

This often creates:
- hidden architecture coupling
- harder debugging
- difficult testing

---

# Production Perspective

Modern Swift architecture strongly prefers:
- explicit dependencies
- constructor injection
- protocol abstraction

over:
- global shared dependencies
- hidden service locators
- singleton-heavy systems

---

# Dependency Injection and Testing

DI dramatically improves testing.

Example:

```swift id="jlwm99"
final class MockService: NetworkService {
    func fetchUsers() {
        print("Mock")
    }
}
```

Now the ViewModel can be tested without real networking.

---

# Common Production Mistakes

## Singletons Everywhere

```swift id="jlwmp1"
APIClient.shared
```

Excessive singleton usage creates:
- hidden coupling
- shared mutable state
- testing difficulty
- lifecycle complexity

---

# Better Approach

Inject dependencies explicitly.

---

# SwiftUI Dependency Injection

SwiftUI commonly uses:
- Environment
- EnvironmentObject
- Observable systems

for dependency propagation.

---

# Interview Questions

### Why is dependency injection important?

Dependency injection reduces coupling, improves testability, and creates more modular architectures.

---

### Why is constructor injection usually preferred?

Because dependencies become explicit, immutable, and guaranteed during initialization.

---

### Why can service locators become problematic?

Because dependencies become hidden and architecture becomes harder to reason about.

---

### Why are excessive singletons dangerous?

They introduce hidden shared state and tightly coupled architecture.

---

# Composition vs Inheritance

Swift strongly encourages composition over inheritance.

This is a VERY common interview discussion.

---

# Inheritance

Inheritance creates “is-a” relationships.

```swift id="jlwm71"
class Animal {
    func eat() {}
}

class Dog: Animal {
}
```

---

# Problems With Deep Inheritance

Large inheritance hierarchies often become:
- tightly coupled
- fragile
- difficult to scale
- difficult to modify safely

Changes in base classes may unexpectedly affect subclasses.

---

# Composition

Composition builds systems from smaller reusable pieces.

```swift id="qjlwm8"
protocol Walkable {
    func walk()
}

protocol Barkable {
    func bark()
}
```

Now behaviors can combine flexibly.

---

# Why Composition Is Powerful

Composition improves:
- flexibility
- modularity
- reuse
- testing
- architecture scalability

---

# Production Perspective

Modern Swift architecture strongly favors:
- composition
- protocols
- modular systems

instead of:
- deep inheritance trees

---

# SwiftUI Example

SwiftUI itself heavily relies on composition.

Views compose smaller reusable views together.

---

# Common Production Mistake

Massive inheritance hierarchies:

```swift id="rjlwm3"
BaseController
→ NetworkController
→ AnalyticsController
→ HomeController
```

These systems become:
- difficult to maintain
- tightly coupled
- fragile

---

# Better Approach

Prefer:
- smaller components
- protocols
- injected behaviors
- reusable composition

---

# Interview Questions

### Why is composition often preferred over inheritance?

Composition reduces coupling and creates more flexible maintainable systems.

---

### Why can deep inheritance hierarchies become problematic?

Because changes propagate unpredictably and tightly couple systems together.

---

### Why does Swift strongly encourage composition?

Because protocol-oriented composition aligns better with modular scalable architecture.

---

# SOLID Principles

SOLID principles are foundational architecture principles.

They help create:
- scalable systems
- maintainable codebases
- modular architectures

---

# S — Single Responsibility Principle

A type should have one reason to change.

---

# Bad Example

```swift id="jlwmc9"
final class UserManager {
    func fetchUsers() {}
    func saveUsers() {}
    func renderUI() {}
}
```

This mixes:
- networking
- persistence
- UI logic

---

# Better Approach

Separate responsibilities.

```swift id="1jlwm5"
final class UserAPIService {
}

final class UserStorage {
}

final class UserRenderer {
}
```

---

# O — Open Closed Principle

Software should be:
- open for extension
- closed for modification

Protocols help heavily here.

---

# Example

```swift id="jlwm02"
protocol PaymentProcessor {
    func process()
}
```

New payment systems can be added without modifying existing architecture.

---

# L — Liskov Substitution Principle

Subtypes should behave correctly when substituted for base abstractions.

---

# Why This Matters

Bad inheritance often violates expected behavior.

This creates:
- fragile architecture
- runtime bugs
- invalid assumptions

---

# I — Interface Segregation Principle

Prefer smaller focused interfaces.

---

# Bad Example

```swift id="jlwmk7"
protocol EverythingService {
    func fetch()
    func save()
    func upload()
    func delete()
}
```

Large protocols become difficult to implement cleanly.

---

# Better Approach

Prefer smaller focused protocols.

```swift id="0jlwmv"
protocol Readable {
    func read()
}

protocol Writable {
    func write()
}
```

---

# D — Dependency Inversion Principle

Depend on abstractions,
not concrete implementations.

---

# Bad Example

```swift id="2jlwm4"
final class UserViewModel {
    let api = APIClient()
}
```

---

# Better Example

```swift id="jlwmf8"
final class UserViewModel {
    let service: NetworkService
}
```

---

# Production Perspective

SOLID principles are not strict rules.

They are architecture guidelines helping systems remain:
- scalable
- modular
- testable
- maintainable

---

# Common Production Mistake

Overengineering tiny projects with excessive abstraction.

Good architecture balances:
- flexibility
- simplicity

---

# Interview Questions

### Why are SOLID principles important?

They help create scalable modular maintainable architectures.

---

### Why can large protocols become problematic?

Because they violate interface segregation and create rigid abstractions.

---

### Why is dependency inversion valuable?

Because systems become loosely coupled and easier to test.

---

### Why should architecture avoid excessive abstraction?

Because overengineering increases complexity without proportional benefit.

---

# Property Wrappers

Property wrappers encapsulate reusable property behavior.

---

# Basic Example

```swift id="jlwmq3"
@propertyWrapper
struct Uppercased {
    private var value = ""

    var wrappedValue: String {
        get { value }
        set { value = newValue.uppercased() }
    }
}
```

Usage:

```swift id="jlwm9t"
@Uppercased var name: String
```

---

# Why Property Wrappers Matter

They allow reusable behavior like:
- validation
- persistence
- synchronization
- formatting
- storage abstraction

---

# SwiftUI Usage

SwiftUI heavily relies on property wrappers.

Examples:
- @State
- @Binding
- @ObservedObject
- @Environment

---

# Production Perspective

Property wrappers improve:
- reuse
- readability
- abstraction

when used appropriately.

---

# Common Mistake

Overusing property wrappers for trivial behavior.

This may reduce readability.

---

# Interview Questions

### Why are property wrappers useful?

They encapsulate reusable property behavior cleanly.

---

### Why are property wrappers important in SwiftUI?

SwiftUI state management heavily relies on them.

---

# Codable

Codable simplifies serialization and deserialization.

---

# Basic Example

```swift id="jlwm7m"
struct User: Codable {
    let id: UUID
    let name: String
}
```

---

# Encoding

```swift id="jlwm3x"
let data = try JSONEncoder().encode(user)
```

---

# Decoding

```swift id="jlwms1"
let user = try JSONDecoder()
    .decode(User.self, from: data)
```

---

# Why Codable Matters

Codable dramatically reduces parsing boilerplate.

Before Codable:
manual parsing was significantly more error-prone.

---

# Custom CodingKeys

```swift id="6jlwmc"
enum CodingKeys: String, CodingKey {
    case firstName = "first_name"
}
```

---

# Why CodingKeys Matter

Real APIs often use:
- snake_case
- inconsistent naming
- nested structures

---

# Date Decoding Strategies

```swift id="jjlwm4"
decoder.dateDecodingStrategy = .iso8601
```

---

# Production Perspective

Robust decoding systems should handle:
- invalid payloads
- missing fields
- partial failures
- backend inconsistencies

---

# Common Production Mistakes

## Force-Trying Decode

```swift id="jlwm8s"
try! decoder.decode(...)
```

This creates crash risk.

---

# Better Approach

Handle decoding failures safely.

---

# Interview Questions

### Why is Codable valuable?

It automates serialization and deserialization safely while reducing boilerplate.

---

### Why are CodingKeys commonly needed?

Because API field naming often differs from Swift naming conventions.

---

### Why should decoding failures be handled carefully?

Because backend payloads may become inconsistent or invalid.

---

# Stack vs Heap

Understanding memory behavior is VERY interview relevant.

---

# Stack Memory

Stack memory is:
- fast
- temporary
- lightweight

Typically used for:
- local values
- function frames
- lightweight value storage

---

# Heap Memory

Heap memory is:
- dynamically allocated
- reference-managed
- shared

Classes commonly allocate heap storage.

---

# Why Heap Allocation Is More Expensive

Heap allocation introduces:
- ARC overhead
- allocation cost
- deallocation cost
- reference tracking

---

# Value Types vs Reference Types

Structs are usually lightweight and stack-friendly.

Classes use heap allocation and reference counting.

---

# Production Perspective

Understanding memory behavior helps explain:
- performance
- ownership
- copying cost
- ARC overhead
- mutation complexity

---

# Interview Questions

### Why is heap allocation more expensive?

Because heap memory requires dynamic allocation and reference management.

---

### Why are value types often more performant?

Because they avoid many heap-allocation and ARC costs.

---

### Why does Swift prefer value semantics?

Because predictable ownership improves safety and performance reasoning.

---

# Dispatch Mechanisms

Swift supports multiple dispatch systems:
- static dispatch
- dynamic dispatch
- witness-table dispatch

---

# Why Dispatch Matters

Dispatch affects:
- performance
- optimization
- polymorphism
- runtime flexibility

---

# Static Dispatch

Resolved at compile time.

Usually faster.

Examples:
- structs
- final methods
- many generic specializations

---

# Dynamic Dispatch

Resolved at runtime.

More flexible,
but introduces runtime overhead.

Examples:
- overridable class methods
- Objective-C runtime dispatch

---

# Witness Table Dispatch

Protocols often use witness-table dispatch internally.

This enables protocol polymorphism.

---

# Production Perspective

You do NOT need compiler-engineer depth for interviews.

But understanding:
- compile-time vs runtime dispatch
- optimization tradeoffs
- protocol dispatch costs

is valuable.

---

# Interview Questions

### Why is static dispatch usually faster?

Because the compiler resolves calls at compile time without runtime lookup.

---

### Why can dynamic dispatch introduce overhead?

Because method resolution occurs at runtime.

---

### Why can final improve optimization?

Because the compiler knows subclass overriding cannot occur.

---

# final Keyword

```swift id="jlwm6w"
final class APIClient {
}
```

---

# Why final Matters

`final` prevents:
- subclassing
- method overriding

This improves:
- predictability
- architecture safety
- optimization opportunities

---

# Production Perspective

Many modern Swift codebases prefer:
- final by default

unless inheritance is intentionally required.

---

# Interview Questions

### Why use final classes?

They improve predictability and optimization opportunities.

---

### Why can excessive inheritance become dangerous?

Because it increases coupling and runtime complexity.

---

# Advanced Swift Topics

The following topics are still highly valuable for:
- medium/senior iOS interviews
- production engineering discussions
- architecture reasoning

These topics are usually not “junior syntax questions”.

They test:
- deeper Swift understanding
- runtime awareness
- production experience

---

# Sequences & Iterators

Swift collections conform heavily to:
- Sequence
- Collection

---

# Sequence

A Sequence produces elements one at a time.

```swift id="7rx6ym"
protocol Sequence
```

Examples:
- Array
- Set
- Dictionary
- AsyncSequence

---

# Why Sequences Matter

Sequences enable:
- iteration
- lazy evaluation
- composable pipelines

---

# Lazy Sequences

```swift id="jlwm02"
let result = values.lazy
    .map { $0 * 2 }
```

---

# Why Lazy Evaluation Matters

Without lazy evaluation:
intermediate arrays may allocate unnecessarily.

Lazy evaluation improves:
- memory efficiency
- CPU usage
- pipeline performance

---

# Production Perspective

Lazy pipelines are especially useful for:
- large datasets
- image pipelines
- rendering systems
- transformations

---

# Common Mistake

Overusing lazy chains for tiny datasets where readability suffers more than performance benefits.

---

# Interview Questions

### Why are lazy sequences useful?

They avoid unnecessary intermediate allocations and improve pipeline efficiency.

---

### Why does Swift rely heavily on Sequence protocols?

Because composable iteration abstractions improve flexibility and reuse.

---

# KeyPaths

KeyPaths allow referencing properties dynamically yet type-safely.

---

# Example

```swift id="jlwm2h"
let keyPath = \User.name
```

---

# Accessing Values

```swift id="jlwmm9"
user[keyPath: \.name]
```

---

# Why KeyPaths Matter

KeyPaths improve:
- reusable transformations
- generic APIs
- bindings
- SwiftUI systems

---

# SwiftUI Usage

SwiftUI relies heavily on:
- bindings
- dynamic member lookup
- KeyPath-driven systems

---

# Production Perspective

KeyPaths are heavily used in:
- state systems
- persistence layers
- reusable abstractions
- sorting/filtering APIs

---

# Interview Questions

### Why are KeyPaths valuable?

They provide type-safe dynamic property access.

---

### Why are KeyPaths important in SwiftUI?

Because SwiftUI heavily relies on dynamic binding systems.

---

# Subscripts

Subscripts provide custom indexed access.

---

# Example

```swift id="jlwmz8"
struct WeekDays {
    private let days = [
        "Mon",
        "Tue",
        "Wed"
    ]

    subscript(index: Int) -> String {
        days[index]
    }
}
```

---

# Why Subscripts Matter

Subscripts improve:
- API ergonomics
- readability
- collection-like access

---

# Production Perspective

Subscripts are commonly used in:
- caches
- data stores
- custom collections
- matrix/grid systems

---

# Interview Questions

### Why are subscripts useful?

They create cleaner and more expressive APIs for indexed access.

---

# Property Observers vs Computed Properties

This comparison is frequently discussed in interviews.

---

# Property Observers

React to mutation.

```swift id="jlwmc0"
didSet {
}
```

---

# Computed Properties

Derive values dynamically.

```swift id="jlwmp2"
var fullName: String {
}
```

---

# Key Difference

Observers:
- respond to changes

Computed properties:
- calculate values dynamically

---

# Production Perspective

Computed properties are usually better for:
- derived state

Observers are usually better for:
- lightweight side effects

---

# Interview Questions

### Difference between computed properties and property observers?

Computed properties derive values dynamically while observers react to mutation events.

---

# Static vs Dynamic Typing Discussion

Swift is:
- statically typed
- strongly typed

---

# Why Static Typing Matters

Static typing improves:
- compile-time safety
- autocomplete
- optimization
- refactoring safety

---

# Production Perspective

Strong type systems reduce:
- runtime crashes
- invalid assumptions
- architecture fragility

---

# Interview Questions

### Why is Swift’s strong type system valuable?

It improves correctness, safety, and tooling support.

---

# NSObject & Objective-C Interoperability

Swift interoperates heavily with Objective-C frameworks.

---

# NSObject

Many Apple frameworks rely on NSObject inheritance.

```swift id="jlwmy6"
class User: NSObject {
}
```

---

# @objc

Exposes Swift APIs to Objective-C runtime.

```swift id="4jlwm1"
@objc func tapped() {
}
```

---

# dynamic

Forces Objective-C dynamic dispatch.

```swift id="jlwmq9"
@objc dynamic func update() {
}
```

---

# Why Objective-C Runtime Still Matters

UIKit and many Apple frameworks still rely heavily on:
- Objective-C runtime
- selectors
- KVO
- dynamic dispatch

---

# Production Perspective

Even modern Swift apps often interact with Objective-C-based frameworks.

---

# Interview Questions

### Why does Swift still support Objective-C interoperability?

Because Apple frameworks historically rely heavily on Objective-C runtime systems.

---

### What does @objc do?

It exposes Swift APIs to Objective-C runtime features.

---

# Memory Leaks Beyond Retain Cycles

Not all memory leaks are retain cycles.

---

# Common Leak Sources

- observers never removed
- caches growing indefinitely
- retained async tasks
- large in-memory datasets
- singleton retention
- image caching issues

---

# Production Perspective

Memory debugging requires understanding:
- ownership
- lifecycle
- resource cleanup
- caching strategies

---

# Interview Questions

### Are retain cycles the only source of memory leaks?

No. Long-lived caches, observers, and retained resources may also leak memory.

---

# API Design Principles

Strong Swift APIs should feel:
- predictable
- expressive
- type-safe
- readable

---

# Swift Naming Philosophy

Swift strongly favors readability.

```swift id="jlwms0"
func fetchUsers()
```

instead of:

```swift id="jlwmf5"
func fetch()
```

---

# Why API Design Matters

Good APIs improve:
- readability
- maintainability
- discoverability
- architecture clarity

---

# Production Perspective

Well-designed APIs reduce:
- onboarding cost
- misuse
- ambiguity
- architecture confusion

---

# Interview Questions

### Why is API design important?

Good APIs improve readability, maintainability, and long-term architecture quality.

---

# Architecture Boundaries

Large apps should avoid:
- massive tightly coupled systems

---

# Good Boundaries

Examples:
- networking layer
- persistence layer
- analytics layer
- feature modules
- domain layer

---

# Why Boundaries Matter

Boundaries improve:
- scalability
- modularity
- testing
- team collaboration

---

# Production Perspective

Most large-scale architecture problems come from:
- hidden coupling
- weak boundaries
- shared mutable state

---

# Interview Questions

### Why are architecture boundaries important?

They reduce coupling and improve modular scalability.

---

# Common Senior-Level Swift Interview Themes

Medium/senior Swift interviews commonly focus on:
- ownership
- mutation
- architecture
- performance
- testing
- modularity
- async reasoning
- memory management

NOT:
- obscure syntax trivia

---

# Important Final Mental Models

Strong Swift engineers think carefully about:
- ownership
- identity
- mutation
- lifecycle
- abstraction
- performance
- architecture tradeoffs

---

# Final Advice For Interviews

When answering Swift interview questions:
- explain WHY
- explain TRADEOFFS
- explain PRODUCTION IMPACT

Not just:
- definitions

---

# Example

Weak answer:

> “Structs are value types.”

Better answer:

> “Structs use value semantics, meaning mutation creates logically independent values instead of shared mutable state. This improves predictability, reduces synchronization complexity, and aligns well with Swift’s architecture philosophy, especially in concurrency and SwiftUI systems.”

That depth difference is what interviewers usually look for in medium/senior iOS interviews.

---

# Final Swift Foundations Summary

The MOST important recurring Swift interview themes are:

- value semantics
- ownership
- ARC
- retain cycles
- closures
- protocol-oriented design
- dependency injection
- composition
- Copy-on-Write
- state modeling
- collection performance
- dispatch behavior
- architecture scalability
- mutation control
- type safety
- modularity
- performance awareness

Understanding:
- not only WHAT Swift features exist
but:
- WHY they exist
- HOW they affect architecture
- WHEN to use them
- WHAT tradeoffs they introduce
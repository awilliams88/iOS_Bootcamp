# Networking — Interview Revision Guide

### Contents

- [Why Networking Matters](#why-networking-matters)
- [Client-Server Architecture](#client-server-architecture)
- [HTTP Fundamentals](#http-fundamentals)
- [HTTP Methods](#http-methods)
- [Status Codes](#status-codes)
- [REST APIs](#rest-apis)
- [URLSession](#urlsession)
- [URLSession Tasks](#urlsession-tasks)
- [Async/Await Networking](#asyncawait-networking)
- [Codable & JSON Decoding](#codable--json-decoding)
- [Error Handling](#error-handling)
- [Retries & Resilience](#retries--resilience)
- [Exponential Backoff](#exponential-backoff)
- [Jitter](#jitter)
- [Idempotency](#idempotency)
- [Authentication](#authentication)
- [Token Refresh Flow](#token-refresh-flow)
- [Pagination](#pagination)
- [Caching](#caching)
- [Multipart Uploads](#multipart-uploads)
- [File Uploads & Downloads](#file-uploads--downloads)
- [WebSockets](#websockets)
- [API Client Architecture](#api-client-architecture)
- [Dependency Injection](#dependency-injection)
- [Network Security](#network-security)
- [Certificate Pinning](#certificate-pinning)
- [Reachability Concepts](#reachability-concepts)
- [Offline-First Thinking](#offline-first-thinking)
- [Debugging Networking](#debugging-networking)
- [Performance Considerations](#performance-considerations)
- [Production Networking Pitfalls](#production-networking-pitfalls)

---

# Why Networking Matters

Most modern apps rely heavily on:
- APIs
- backend systems
- cloud infrastructure
- authentication
- uploads/downloads
- real-time communication

Networking is one of the MOST important production engineering areas.

---

# Why Networking Is Difficult

Networking systems are inherently unreliable.

Requests may fail because of:
- no internet
- packet loss
- server outages
- timeouts
- authentication expiration
- invalid payloads
- DNS issues
- SSL failures

---

# Production Mental Model

Good networking systems are built assuming:
# failure is normal

not:
# failure is rare

---

# Production Perspective

Strong networking systems emphasize:
- resilience
- retries
- graceful degradation
- cancellation
- caching
- observability
- security

---

# Interview Questions

### Why is networking considered unreliable?

Because requests depend on many external systems including connectivity, servers, DNS, TLS, and infrastructure.

---

### What makes production networking difficult?

Handling failures, retries, authentication, cancellation, scalability, and resilience safely.

---

# Client-Server Architecture

Most mobile apps follow:
- client-server architecture

---

# Client

The iOS app:
- sends requests
- renders UI
- stores local state

---

# Server

Backend systems:
- process requests
- store data
- authenticate users
- coordinate business logic

---

# Why Separation Matters

The client should generally focus on:
- presentation
- user interaction
- local orchestration

while servers handle:
- centralized business rules
- persistence
- scalability

---

# Production Perspective

Strong client-server separation improves:
- maintainability
- scalability
- deployment flexibility

---

# Interview Questions

### Why is client-server separation important?

Because it improves scalability, maintainability, and centralized business control.

---

# HTTP Fundamentals

HTTP is the foundation of most API communication.

---

# Request Structure

HTTP requests contain:
- URL
- method
- headers
- body

---

# Example

```swift id="jlwma1"
GET /users
Authorization: Bearer token
```

---

# Response Structure

Responses contain:
- status code
- headers
- body

---

# Why HTTP Matters

Understanding HTTP is foundational for:
- APIs
- authentication
- caching
- uploads
- debugging

---

# Production Perspective

Many networking interview failures happen because developers:
- know frameworks
but:
- do not understand HTTP fundamentals

---

# Interview Questions

### What does an HTTP request contain?

A request contains a URL, method, headers, and optional body.

---

### Why are HTTP fundamentals important?

Because networking frameworks ultimately build on top of HTTP behavior.

---

# HTTP Methods

HTTP methods describe request intent.

---

# GET

Retrieves data.

```swift id="jlwmd1"
GET /users
```

---

# POST

Creates resources.

```swift id="jlwmy2"
POST /users
```

---

# PUT

Replaces resources fully.

---

# PATCH

Partially updates resources.

---

# DELETE

Removes resources.

---

# Why Methods Matter

Correct HTTP semantics improve:
- API clarity
- caching behavior
- scalability
- idempotency reasoning

---

# Production Perspective

Incorrect method usage creates:
- caching problems
- retry issues
- API confusion

---

# Interview Questions

### Difference between PUT and PATCH?

PUT replaces entire resources while PATCH partially updates them.

---

### Why do HTTP methods matter?

Because they communicate request semantics and affect caching and retry behavior.

---

# Status Codes

Status codes describe request outcomes.

This is EXTREMELY important for interviews.

---

# 2xx Success

```text id="jlwmt4"
200 OK
201 Created
204 No Content
```

---

# 4xx Client Errors

```text id="jlwmu5"
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
429 Too Many Requests
```

---

# 5xx Server Errors

```text id="jlwmp0"
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

---

# Important Retry Mental Model

Usually retry:
- temporary failures
- timeouts
- 5xx errors

Usually DO NOT retry:
- malformed requests
- authentication failures
- validation errors

---

# Production Perspective

Understanding status codes is critical for:
- retry systems
- user messaging
- analytics
- debugging

---

# Common Production Mistake

Retrying ALL failures blindly.

This may:
- spam servers
- worsen outages
- create retry storms

---

# Interview Questions

### Which HTTP errors are commonly retryable?

Usually temporary failures like 5xx errors and connectivity timeouts.

---

### Why are 4xx errors usually not retryable?

Because they often represent invalid requests or authentication issues.

---

# REST APIs

REST is the most common API architecture style.

---

# REST Principles

REST APIs typically emphasize:
- resources
- stateless communication
- predictable endpoints

---

# Example Endpoints

```text id="jlwmg3"
/users
/users/42
/posts/10/comments
```

---

# Why REST Matters

REST APIs are:
- predictable
- scalable
- easy to reason about

---

# Production Perspective

Even though alternatives exist:
- GraphQL
- gRPC

REST remains dominant in mobile systems.

---

# Common Production Mistake

Treating REST endpoints like arbitrary RPC calls.

This often creates:
- inconsistent APIs
- poor scalability
- unclear semantics

---

# Interview Questions

### Why are REST APIs popular?

Because they are predictable, scalable, and resource-oriented.

---

### What does stateless communication mean?

Each request contains enough information to process independently.

---

# URLSession

URLSession is Apple’s core networking framework.

This is one of the MOST important iOS networking topics.

---

# Basic Request Example

```swift id="jlwma8"
let url = URL(string: endpoint)!

let (data, response) =
    try await URLSession.shared.data(
        from: url
    )
```

---

# Why URLSession Matters

URLSession provides:
- networking
- uploads
- downloads
- background transfers
- caching integration
- async APIs

---

# URLRequest

More advanced networking uses:
- URLRequest

---

# Example

```swift id="jlwmy6"
var request = URLRequest(url: url)

request.httpMethod = "POST"
request.addValue(
    "application/json",
    forHTTPHeaderField: "Content-Type"
)
```

---

# Production Perspective

Professional networking systems rarely use:
```swift id="jlwmp9"
URLSession.shared
```

directly everywhere.

Instead:
- API layers
- services
- abstractions
- clients

are usually built around URLSession.

---

# Common Production Mistake

Mixing:
- networking
- decoding
- business logic
- UI updates

all inside one request block.

---

# Better Approach

Separate:
- request creation
- transport
- decoding
- business transformation
- UI rendering

---

# Interview Questions

### Why is URLSession important?

Because it is Apple’s primary networking framework for HTTP communication.

---

### Why should networking logic not live directly inside views?

Because networking, decoding, business logic, and rendering should remain separated for maintainability.

---

# URLSession Tasks

URLSession provides multiple task types.

Understanding the differences is VERY important for interviews.

---

# Data Tasks

Used for:
- standard API requests
- JSON fetching
- lightweight responses

---

# Example

```swift id="jlwmd9"
let (data, response) =
    try await session.data(
        for: request
    )
```

---

# Upload Tasks

Used for:
- file uploads
- multipart uploads
- large payloads

---

# Download Tasks

Used for:
- large downloads
- resumable downloads
- background transfers

---

# Why Task Types Matter

Different networking workflows have different:
- memory behavior
- performance requirements
- lifecycle needs

---

# Production Perspective

Large files should usually prefer:
- streaming
- download tasks
- background transfers

instead of loading everything into memory.

---

# Common Production Mistake

Using:
```swift id="jlwma0"
dataTask
```

for huge files.

This may:
- spike memory
- crash apps
- block rendering

---

# Better Approach

Use:
- download tasks
- streaming systems
- temporary file handling

for large assets.

---

# Interview Questions

### Difference between data tasks and download tasks?

Data tasks load response data directly into memory while download tasks stream content to files.

---

### Why are download tasks better for large files?

Because they avoid excessive memory usage and support resumable transfers.

---

# Async/Await Networking

Modern Swift networking strongly favors:
- async/await

---

# Example

```swift id="jlwmu2"
func fetchUsers() async throws
    -> [User] {

    let (data, response) =
        try await session.data(
            from: url
        )

    return try decoder.decode(
        [User].self,
        from: data
    )
}
```

---

# Why async/await Matters

Older callback systems often created:
- callback pyramids
- lifecycle complexity
- difficult cancellation
- unreadable code

---

# Example Callback Hell

```swift id="jlwmp5"
fetchUser {
    fetchPosts {
        fetchComments {
        }
    }
}
```

---

# Why async/await Is Better

async/await improves:
- readability
- error handling
- cancellation integration
- structured concurrency

---

# Production Perspective

Modern iOS networking architectures strongly rely on:
- structured concurrency
- async APIs
- cancellation-aware tasks

---

# Important Mental Model

```swift id="jlwmg1"
await
```

does NOT necessarily block threads.

Tasks suspend efficiently.

---

# Production Networking Principle

Networking should always support:
- cancellation
- interruption
- lifecycle awareness

---

# Common Production Mistake

Launching networking requests that outlive screens unnecessarily.

This creates:
- stale updates
- wasted networking
- invalid rendering
- memory retention

---

# Better Approach

Tie networking work to:
- view lifecycle
- ownership boundaries
- cancellation propagation

---

# Interview Questions

### Why is async/await better for networking?

Because it improves readability, cancellation handling, and structured async flow.

---

### Why should networking support cancellation?

Because users constantly interrupt workflows and navigate away.

---

### Does await block threads?

No. Swift Concurrency suspends tasks efficiently instead of blocking threads directly.

---

# Codable & JSON Decoding

Most APIs exchange JSON.

Swift commonly uses:
- Codable

for encoding and decoding.

---

# Basic Codable Model

```swift id="jlwmy4"
struct User: Codable {
    let id: Int
    let name: String
}
```

---

# Decoding Example

```swift id="jlwmt8"
let users = try decoder.decode(
    [User].self,
    from: data
)
```

---

# Why Codable Matters

Codable reduces:
- boilerplate
- manual parsing
- serialization complexity

---

# JSONDecoder Strategies

---

# Snake Case Conversion

```swift id="jlwme7"
decoder.keyDecodingStrategy =
    .convertFromSnakeCase
```

---

# Date Decoding

```swift id="jlwms2"
decoder.dateDecodingStrategy =
    .iso8601
```

---

# Production Perspective

Real-world APIs frequently contain:
- inconsistent payloads
- optional fields
- missing values
- invalid formats

Robust decoding matters heavily.

---

# Common Production Mistake

Blindly trusting backend payloads.

This creates:
- crashes
- invalid rendering
- decoding failures

---

# Better Approach

Build resilient models with:
- optionals
- validation
- graceful fallback handling

---

# Production Decoding Principle

Backend contracts evolve over time.

Networking systems must tolerate:
- schema evolution
- backward compatibility
- partial failures

---

# Interview Questions

### Why is Codable useful?

Codable simplifies serialization and JSON decoding significantly.

---

### Why should apps not blindly trust backend payloads?

Because APIs evolve and malformed data may occur in production.

---

### Why are decoding strategies useful?

They simplify conversion between backend formats and Swift models.

---

# Error Handling

Networking systems must handle failures carefully.

This is EXTREMELY important for interviews.

---

# Common Networking Failures

- no internet
- timeouts
- DNS failures
- invalid responses
- decoding failures
- authentication expiration
- server outages

---

# Better Error Modeling

Avoid:
```swift id="jlwmo3"
print(error)
```

only.

Instead:
- classify errors
- map failures meaningfully
- expose actionable information

---

# Example Error Enum

```swift id="jlwmu6"
enum NetworkError: Error {
    case invalidResponse
    case unauthorized
    case decodingFailed
    case serverError
    case noInternet
}
```

---

# Why Error Modeling Matters

Good error systems improve:
- debugging
- analytics
- retry logic
- user messaging
- observability

---

# Production Perspective

Strong networking systems distinguish between:
- retryable failures
- fatal failures
- authentication issues
- validation problems

---

# Common Production Mistake

Showing raw backend errors directly to users.

This creates:
- confusing UX
- security leakage
- inconsistent messaging

---

# Better Approach

Map:
- technical errors
into:
- user-friendly experiences

---

# Interview Questions

### Why is error classification important?

Because different failures require different retry, logging, and UX behavior.

---

### Why should apps avoid exposing raw backend errors directly?

Because technical backend messages are often confusing or unsafe for users.

---

# Retries & Resilience

This is one of the MOST important networking interview topics.

You were explicitly asked about this.

---

# Why Retries Exist

Networking failures are often temporary.

Examples:
- server overload
- connectivity interruption
- transient outages
- timeouts

Retries improve resilience.

---

# Important Production Rule

NOT all requests should retry.

---

# Retryable Failures

Usually retry:
- timeouts
- connectivity interruptions
- temporary 5xx errors
- rate limits with retry guidance

---

# Non-Retryable Failures

Usually DO NOT retry:
- malformed requests
- authentication failures
- validation errors
- permanent 4xx issues

---

# Production Perspective

Strong retry systems require:
- retry limits
- cancellation support
- exponential backoff
- jitter
- idempotency awareness

---

# Common Production Mistake

Infinite retries.

```swift id="jlwmt1"
while true {
    retry()
}
```

This may:
- overload servers
- drain battery
- create infinite loops

---

# Better Approach

Use:
- retry limits
- controlled delays
- cancellation-aware retries

---

# Interview Questions

### Why are retries important?

Retries improve resilience against temporary networking failures.

---

### Why should retries be limited?

Because uncontrolled retries may overload systems and waste resources.

---

### Why should not all failures retry?

Because some failures are permanent and repeating them is pointless.

---

# Exponential Backoff

Retries should usually delay progressively.

---

# Example Mental Model

```text id="jlwmg7"
1 second
2 seconds
4 seconds
8 seconds
```

---

# Why Exponential Backoff Matters

Without backoff:
clients may repeatedly hammer struggling servers.

This worsens:
- outages
- infrastructure pressure
- retry storms

---

# Production Perspective

Backoff improves:
- system stability
- scalability
- infrastructure recovery

---

# Example Retry Logic

```swift id="jlwmy5"
let delay =
    pow(2.0, attempt)
```

---

# Common Production Mistake

Retrying immediately in tight loops.

---

# Better Approach

Increase delays progressively while respecting:
- cancellation
- retry limits
- server guidance

---

# Interview Questions

### Why is exponential backoff important?

Because it reduces retry pressure during infrastructure failures.

---

### Why are immediate retries dangerous?

Because they may worsen outages and overload struggling systems.

---

# Jitter

Jitter adds randomness to retry timing.

This is VERY interview relevant.

---

# Why Jitter Matters

Without jitter:
many devices retry simultaneously.

This creates:
- retry spikes
- synchronized traffic storms
- infrastructure overload

---

# Example

Instead of:
```text id="jlwma5"
2s
4s
8s
```

Use randomized ranges:

```text id="jlwmp7"
1.8s
4.3s
7.1s
```

---

# Production Perspective

Large-scale systems rely heavily on:
- jitter
- backoff
- traffic smoothing

---

# Interview Questions

### What is jitter?

Jitter adds randomness to retry delays.

---

### Why is jitter important?

Because synchronized retries may overload infrastructure during outages.

---

# Idempotency

This is one of the MOST important retry concepts.

---

# Why Idempotency Matters

Some requests are safe to repeat.

Others are NOT.

---

# Safe Retry Example

```text id="jlwmd2"
GET /users
```

Fetching repeatedly is usually safe.

---

# Dangerous Retry Example

```text id="jlwmu9"
POST /payments
```

Repeating may:
- duplicate charges
- create duplicate orders
- corrupt workflows

---

# Production Perspective

Retry systems MUST understand:
- operation safety
- side effects
- duplication risk

---

# Idempotency Keys

Production systems often use:
- idempotency tokens

to safely retry mutations.

---

# Interview Questions

### Why is idempotency important?

Because some operations are unsafe to repeat multiple times.

---

### Why are GET requests usually safer to retry?

Because they generally do not mutate backend state.

---

# Authentication

Authentication verifies:
- who the user is

This is foundational for:
- protected APIs
- user accounts
- sessions
- authorization

---

# Common Authentication Systems

- Bearer tokens
- JWT
- OAuth
- API keys
- session cookies

---

# Bearer Tokens

Most mobile apps use:
- Bearer token authentication

---

# Example

```swift id="jlwmp2"
request.addValue(
    "Bearer \(token)",
    forHTTPHeaderField: "Authorization"
)
```

---

# Why Tokens Matter

Tokens allow:
- stateless authentication
- scalable APIs
- secure request validation

---

# Production Perspective

Tokens should NEVER be:
- hardcoded
- logged
- stored insecurely

---

# Common Production Mistake

Storing tokens in:
- UserDefaults

This is unsafe.

---

# Better Approach

Use:
- Keychain

for secure credential storage.

---

# JWT (JSON Web Token)

JWTs are commonly used authentication tokens.

They often contain:
- user identity
- expiration
- claims
- permissions

---

# Important JWT Mental Model

JWTs are:
- signed
NOT:
- encrypted by default

---

# Production Perspective

Sensitive data should NOT be blindly stored inside JWT payloads.

---

# Interview Questions

### Why are Bearer tokens commonly used?

Because they support scalable stateless authentication systems.

---

### Why should tokens not be stored in UserDefaults?

Because UserDefaults is not secure storage.

---

### Why are JWTs not inherently encrypted?

Because JWTs are typically signed for integrity, not encrypted for secrecy.

---

# Token Refresh Flow

Access tokens often expire.

Production apps usually implement:
- refresh token systems

---

# Typical Flow

1. Request fails with:
```text id="jlwmd3"
401 Unauthorized
```

2. App uses refresh token

3. Backend returns new access token

4. Original request retries

---

# Why Refresh Systems Matter

Users should NOT:
- constantly log in again

Good refresh systems improve:
- UX
- session continuity
- security

---

# Production Perspective

Refresh systems become difficult because of:
- concurrency
- race conditions
- simultaneous expiration

---

# Common Production Mistake

Multiple simultaneous refresh requests.

Example:
10 API requests fail simultaneously →
10 refresh requests fire simultaneously.

This creates:
- duplicated refresh traffic
- invalid state races
- token inconsistency

---

# Better Approach

Use:
- refresh synchronization
- shared refresh pipelines
- request queuing

---

# Production Mental Model

Authentication systems must handle:
- concurrency
- expiration
- retries
- cancellation
- security

carefully.

---

# Example Architecture

```text id="jlwmg9"
Request fails
    ↓
Token refresh coordinator
    ↓
Single refresh request
    ↓
Queued requests retry
```

---

# Interview Questions

### Why are refresh token systems important?

Because access tokens expire and users should maintain seamless authenticated sessions.

---

### Why can token refresh systems become difficult?

Because concurrent request failures may trigger race conditions and duplicate refresh attempts.

---

### Why should refresh requests be synchronized?

To avoid duplicated refresh traffic and inconsistent authentication state.

---

# Pagination

Pagination prevents huge datasets from loading all at once.

This is VERY important for scalability.

---

# Why Pagination Matters

Without pagination:
large datasets may:
- overwhelm memory
- increase latency
- slow rendering
- waste bandwidth

---

# Offset Pagination

Example:

```text id="jlwmy8"
?page=2&limit=20
```

---

# Cursor Pagination

Example:

```text id="jlwmu4"
?cursor=abc123
```

---

# Why Cursor Pagination Is Better

Cursor systems handle:
- large datasets
- dynamic ordering
- infinite feeds

more reliably.

---

# Production Perspective

Modern large-scale feeds often prefer:
- cursor pagination

instead of:
- offset pagination

---

# Common Production Mistake

Fetching huge datasets eagerly.

This hurts:
- memory
- scrolling
- battery
- startup performance

---

# Better Approach

Use:
- incremental loading
- prefetching
- lazy rendering

---

# Infinite Scrolling

Very common mobile pattern.

Example:
- social feeds
- chats
- marketplaces

---

# Production Challenges

Pagination systems must handle:
- duplicates
- ordering changes
- refresh merging
- race conditions

---

# Interview Questions

### Why is pagination important?

Because large datasets should not load entirely into memory at once.

---

### Why is cursor pagination often preferred?

Because it scales better for dynamic large datasets and infinite feeds.

---

### What problems can pagination systems face?

Duplicates, ordering inconsistencies, race conditions, and merge conflicts.

---

# Caching

Caching stores previously fetched data for reuse.

---

# Why Caching Matters

Caching improves:
- performance
- responsiveness
- bandwidth efficiency
- offline resilience

---

# Common Cache Types

- memory cache
- disk cache
- HTTP cache
- image cache

---

# URLCache

URLSession integrates with:
- URLCache

---

# Example

```swift id="jlwms5"
URLCache.shared
```

---

# Why HTTP Caching Matters

Servers may provide:
- Cache-Control
- ETag
- Last-Modified

headers.

These help reduce unnecessary requests.

---

# ETag Example

```text id="jlwmt7"
ETag: "abc123"
```

---

# Why ETags Matter

Clients can ask:
“Has this changed?”

instead of downloading full content repeatedly.

---

# Production Perspective

Good caching systems improve:
- scalability
- battery life
- network efficiency

dramatically.

---

# Common Production Mistake

Caching sensitive user data insecurely.

---

# Better Approach

Understand:
- cache expiration
- security sensitivity
- invalidation strategies

---

# Cache Invalidation

One of the hardest production problems.

---

# Why Cache Invalidation Is Difficult

Data may:
- become stale
- conflict with server updates
- drift from backend truth

---

# Production Perspective

Caching requires balancing:
- freshness
- performance
- consistency

---

# Interview Questions

### Why is caching important?

Because it improves responsiveness and reduces unnecessary networking.

---

### Why is cache invalidation difficult?

Because stale data and synchronization conflicts are hard to manage correctly.

---

### What are ETags used for?

ETags help clients validate whether cached content has changed.

---

# Multipart Uploads

Multipart uploads are used for:
- images
- videos
- documents
- forms

---

# Why Multipart Exists

Standard JSON requests are inefficient for large binary files.

Multipart requests support:
- mixed payloads
- metadata + files
- streaming uploads

---

# Example Use Cases

- profile photo upload
- video upload
- attachment systems

---

# Production Perspective

Large uploads should strongly consider:
- streaming
- retries
- resumability
- progress tracking

---

# Common Production Mistake

Loading huge files entirely into memory before upload.

This may:
- spike memory
- crash apps
- hurt performance

---

# Better Approach

Use:
- streaming uploads
- file handles
- background transfers

---

# Interview Questions

### Why are multipart uploads useful?

Because they support efficient transfer of binary files and metadata together.

---

### Why should huge uploads avoid loading everything into memory?

Because excessive memory usage may crash or freeze apps.

---

# File Uploads & Downloads

Large transfer systems require special handling.

---

# Upload Concerns

- progress tracking
- cancellation
- retries
- background behavior
- resumability

---

# Download Concerns

- temporary files
- storage limits
- resume data
- integrity validation

---

# URLSessionDownloadTask

Download tasks stream directly to disk.

---

# Why Streaming Matters

Streaming avoids:
- large memory spikes
- blocking UI
- excessive allocations

---

# Background Transfers

iOS supports:
- background URLSession configurations

---

# Why Background Transfers Matter

Users may:
- lock device
- switch apps
- lose foreground execution

Transfers should continue safely.

---

# Production Perspective

Production media apps rely heavily on:
- resumable downloads
- background uploads
- progressive transfer systems

---

# Common Production Mistake

Ignoring cancellation support for uploads/downloads.

This wastes:
- bandwidth
- battery
- server resources

---

# Interview Questions

### Why are background transfers important?

Because users frequently leave the app during long-running transfers.

---

### Why is streaming important for large downloads?

Because streaming avoids loading huge content into memory.

---

# WebSockets

WebSockets provide:
- persistent bidirectional communication

---

# Why WebSockets Matter

Traditional HTTP:
- request → response

WebSockets:
- continuous realtime communication

---

# Common Use Cases

- chat
- live feeds
- multiplayer systems
- realtime dashboards
- notifications

---

# Production Challenges

Realtime systems must handle:
- reconnect logic
- dropped connections
- heartbeats
- ordering
- synchronization

---

# Production Perspective

Realtime networking is MUCH harder than basic REST APIs.

---

# Common Production Mistake

Assuming WebSocket connections remain stable forever.

Mobile networks constantly change.

---

# Better Approach

Implement:
- reconnection
- exponential backoff
- connection state handling

---

# Interview Questions

### Why are WebSockets useful?

Because they enable realtime bidirectional communication.

---

### Why are mobile WebSocket systems difficult?

Because mobile connectivity changes frequently and connections may drop unexpectedly.

---

# API Client Architecture

Production apps should NOT scatter networking logic everywhere.

This is one of the MOST important medium/senior interview topics.

---

# Why Architecture Matters

As apps scale:
- requests increase
- auth complexity increases
- retries increase
- caching grows
- testing becomes difficult

Without architecture:
networking becomes:
- duplicated
- inconsistent
- difficult to debug

---

# Common Bad Architecture

```swift id="jlwma7"
ViewController
    ↓
URLSession directly
    ↓
Decode
    ↓
Update UI
```

Everything mixed together.

---

# Problems With This Approach

This creates:
- tight coupling
- poor testability
- duplicated logic
- auth duplication
- retry inconsistency
- difficult mocking

---

# Better Architecture

```text id="jlwmp4"
View
    ↓
ViewModel
    ↓
Service Layer
    ↓
API Client
    ↓
URLSession
```

---

# Why This Is Better

This separates:
- rendering
- business logic
- networking
- transport
- decoding

---

# Production Perspective

Strong networking systems emphasize:
- separation of concerns
- testability
- scalability
- dependency injection
- centralized behavior

---

# API Client

The API client coordinates:
- requests
- headers
- retries
- auth
- decoding
- error mapping

---

# Example API Client

```swift id="jlwmt6"
protocol APIClient {
    func request<T: Decodable>(
        _ endpoint: Endpoint
    ) async throws -> T
}
```

---

# Why Protocols Matter

Protocols improve:
- testing
- mocking
- dependency injection
- modularity

---

# Endpoint Modeling

Professional networking systems usually model:
- endpoints explicitly

---

# Example

```swift id="jlwmy0"
enum Endpoint {
    case users
    case profile(id: Int)
}
```

---

# Why Endpoint Modeling Helps

This centralizes:
- paths
- methods
- headers
- parameters

---

# Production Perspective

Strong API systems avoid:
- random URL strings scattered everywhere

---

# Common Production Mistake

Hardcoding:
- URLs
- headers
- auth logic

across multiple files.

---

# Better Approach

Centralize:
- request construction
- auth handling
- retry behavior
- decoding

---

# Interview Questions

### Why should networking be abstracted behind API layers?

Because separation improves scalability, consistency, and testability.

---

### Why are protocols useful in networking systems?

Because they improve mocking and dependency injection.

---

### Why should endpoints be modeled centrally?

Because centralized request definitions improve consistency and maintainability.

---

# Dependency Injection

Dependency injection is VERY important in networking architecture.

---

# Why Dependency Injection Matters

Without DI:
systems become:
- tightly coupled
- difficult to test
- difficult to replace

---

# Example

Bad:

```swift id="jlwmd5"
let api = RealAPIClient()
```

inside business logic.

---

# Better

```swift id="jlwmg2"
init(api: APIClient)
```

---

# Why This Is Better

Now:
- mocks
- previews
- tests
- alternate implementations

become easy.

---

# Production Perspective

Networking layers should almost always support:
- dependency injection

---

# Common Production Mistake

Single gigantic global networking singleton.

This creates:
- hidden dependencies
- difficult testing
- tight coupling

---

# Better Approach

Inject dependencies explicitly.

---

# Interview Questions

### Why is dependency injection important?

Because it improves modularity, testing, and flexibility.

---

### Why are giant global networking singletons problematic?

Because they create hidden coupling and difficult testing scenarios.

---

# Network Security

Security is foundational for networking systems.

---

# HTTPS

Production apps should ALWAYS prefer:
- HTTPS

---

# Why HTTPS Matters

HTTPS provides:
- encryption
- integrity
- authentication

---

# Risks Without HTTPS

Without encryption:
attackers may:
- intercept traffic
- steal credentials
- modify payloads

---

# App Transport Security (ATS)

iOS strongly encourages:
- secure networking defaults

through ATS.

---

# Production Perspective

Disabling ATS casually is considered poor security practice.

---

# Common Production Mistake

Logging:
- tokens
- passwords
- sensitive payloads

in production logs.

---

# Better Approach

Protect:
- credentials
- personal data
- tokens
- session information

carefully.

---

# Interview Questions

### Why is HTTPS important?

Because it encrypts communication and protects against interception.

---

### Why should sensitive data not appear in logs?

Because logs may expose credentials and personal information.

---

# Certificate Pinning

Certificate pinning strengthens HTTPS trust validation.

---

# Why Pinning Exists

Normally devices trust:
- system certificate authorities

Pinning additionally validates:
- expected certificates
- expected public keys

---

# Why Pinning Helps

Pinning reduces risk from:
- compromised certificate authorities
- MITM attacks

---

# Production Perspective

Certificate pinning increases security,
but also increases:
- operational complexity

---

# Common Production Mistake

Hardcoding pinning incorrectly without rotation planning.

This may:
- break apps
- lock out users
- fail during certificate renewals

---

# Better Approach

Understand:
- certificate rotation
- backup pins
- operational maintenance

---

# Interview Questions

### Why does certificate pinning improve security?

Because it validates expected certificates beyond default CA trust.

---

### Why can certificate pinning become operationally difficult?

Because certificate renewals and rotations must be managed carefully.

---

# Reachability Concepts

Mobile networking is highly unstable.

This is VERY important.

---

# Why Connectivity Changes Constantly

Devices may:
- switch WiFi
- lose signal
- enter tunnels
- switch cellular networks

---

# Important Production Mental Model

Connectivity may change:
# at any moment

---

# Common Mistake

Assuming:
“If internet existed once,
requests will continue succeeding.”

This is false.

---

# Better Approach

Networking systems should:
- tolerate failures
- retry safely
- degrade gracefully

---

# Reachability APIs

Apple provides:
- NWPathMonitor

---

# Why Reachability Is Useful

Apps may:
- avoid unnecessary requests
- show offline UI
- coordinate retries

---

# Important Production Rule

Reachability should NOT guarantee:
“network requests will succeed.”

Because connectivity may change immediately afterward.

---

# Production Perspective

Reachability is:
- advisory
NOT:
- authoritative

---

# Interview Questions

### Why is mobile networking unstable?

Because connectivity changes constantly between WiFi, cellular, and weak signal conditions.

---

### Why should reachability not guarantee request success?

Because network conditions may change immediately after checks occur.

---

# Offline-First Thinking

Strong mobile apps tolerate poor connectivity gracefully.

---

# Why Offline Thinking Matters

Users frequently experience:
- weak connectivity
- intermittent networking
- airplane mode
- tunnels
- background interruptions

---

# Offline-Friendly Systems

Good apps support:
- caching
- optimistic UI
- retries
- local persistence
- graceful degradation

---

# Example

Messages may:
- appear locally immediately
- sync later asynchronously

---

# Production Perspective

Offline resilience dramatically improves:
- UX
- reliability perception
- responsiveness

---

# Common Production Mistake

Blocking the entire app when one request fails.

---

# Better Approach

Design:
- resilient UI states
- partial functionality
- retryable workflows

---

# Interview Questions

### Why is offline-first thinking important?

Because mobile users frequently experience unstable connectivity.

---

### What improves offline resilience?

Caching, retries, local persistence, and graceful degradation.

---

# Debugging Networking

Strong debugging skills are VERY important for production engineering.

---

# Common Debugging Tools

- Xcode Console
- Charles Proxy
- Proxyman
- Instruments
- Network Inspector

---

# Important Debugging Areas

- headers
- payloads
- auth failures
- decoding failures
- retries
- latency
- caching behavior

---

# Production Perspective

Many networking issues are actually:
- backend contract mismatches
- auth problems
- payload evolution
- decoding assumptions

---

# Common Production Mistake

Only logging:
```swift id="jlwmu8"
error.localizedDescription
```

without request context.

---

# Better Approach

Log:
- endpoint
- status code
- correlation IDs
- retry attempts
- decoding failures

carefully and safely.

---

# Interview Questions

### Why is networking debugging important?

Because distributed systems fail in many unpredictable ways.

---

### What information is useful during networking debugging?

Status codes, payloads, headers, retries, and decoding context.

---

# Performance Considerations

Networking performance strongly affects:
- UX
- battery
- startup speed
- scrolling
- responsiveness

---

# Common Networking Performance Problems

- duplicate requests
- no caching
- huge payloads
- eager loading
- polling excessively
- retry storms
- oversized images

---

# Request Deduplication

Avoid firing:
- identical requests repeatedly

---

# Why Deduplication Matters

Duplicate requests waste:
- bandwidth
- battery
- server resources

---

# Pagination Performance

Loading huge datasets eagerly hurts:
- memory
- rendering
- startup latency

---

# Better Approach

Use:
- incremental loading
- lazy rendering
- prefetching

---

# Image Optimization

Large unoptimized images heavily impact:
- scrolling
- memory
- rendering

---

# Production Perspective

Performance optimization usually focuses on:
- reducing unnecessary work
NOT:
- premature micro-optimization

---

# Interview Questions

### Why are duplicate requests dangerous?

Because they waste bandwidth, battery, and server capacity.

---

### Why should large datasets load incrementally?

Because eager loading hurts memory and responsiveness.

---

# Production Networking Pitfalls

These are VERY important for medium/senior interviews.

---

# Common Networking Pitfalls

- infinite retries
- retry storms
- auth race conditions
- stale requests
- duplicated requests
- memory spikes
- giant payloads
- decoding assumptions
- insecure token storage
- background task leaks
- cancellation ignored
- tightly coupled networking layers

---

# Example: Infinite Retry Loop

```swift id="jlwms3"
while true {
    retry()
}
```

Dangerous because:
- infrastructure overload
- battery drain
- infinite loops

---

# Example: Stale Requests

User leaves screen →
request completes later →
old UI updates incorrectly.

---

# Better Approach

Use:
- cancellation-aware requests
- lifecycle ownership
- structured concurrency

---

# Example: Auth Refresh Race

10 requests fail →
10 refresh attempts occur simultaneously.

This may:
- corrupt auth state
- invalidate tokens
- create inconsistent sessions

---

# Better Approach

Centralized refresh coordination.

---

# Production Perspective

Most networking bugs come from:
- ownership
- lifecycle
- retries
- synchronization
- resilience reasoning

NOT:
- syntax

---

# Final Networking Mental Model

Strong networking engineers think carefully about:
- resilience
- retries
- scalability
- ownership
- cancellation
- observability
- security
- offline behavior
- performance tradeoffs

---

# Final Interview Advice

For medium/senior networking interviews:
focus on explaining:
- WHY
- FAILURE MODES
- TRADEOFFS
- RESILIENCE STRATEGIES

not just:
- URLSession syntax

---

# Example

Weak answer:

> “Retries retry failed requests.”

Strong answer:

> “Retries improve resilience against transient failures, but production retry systems must also consider cancellation, backoff, jitter, retry limits, and idempotency to avoid retry storms and duplicated side effects.”

That depth difference is what interviewers usually look for.

---

# Final Networking Summary

The MOST important recurring networking interview themes are:

- HTTP fundamentals
- status codes
- URLSession
- async networking
- retries
- backoff
- jitter
- idempotency
- authentication
- token refresh
- pagination
- caching
- offline resilience
- security
- architecture
- cancellation
- lifecycle ownership
- scalability
- performance

Understanding:
- not only HOW requests are made
but:
- WHY distributed systems fail
- HOW retries affect infrastructure
- WHAT scalability tradeoffs exist
- WHY lifecycle ownership matters
- HOW resilient mobile systems are designed
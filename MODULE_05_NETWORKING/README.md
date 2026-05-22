# Module 05 — Networking

Networking is a core skill for every iOS engineer. This module covers URLSession fundamentals, modern async/await networking, authentication patterns, and production-grade API client architecture.

---

## Topics

| # | File | Description |
|---|------|-------------|
| 01 | [URLSession Fundamentals](01-urlsession-fundamentals.md) | URLSession, URLRequest, data/download/upload tasks, configuration |
| 02 | [Async Networking](02-async-networking.md) | async/await with URLSession, retry, cancellation, AsyncStream |
| 03 | [Auth & Security](03-auth-and-security.md) | Bearer tokens, token refresh race condition, OAuth, cert pinning |
| 04 | [API Client Architecture](04-api-client-architecture.md) | Clean API layer, endpoint pattern, error modeling, mock network layer |

---

## Key Interview Themes

- **URLSession configuration types** — default vs ephemeral vs background
- **Token refresh race condition** — the classic interview scenario
- **Certificate pinning** — how and why
- **Retry with exponential backoff** — production pattern
- **Protocol-based API client** — for testability

---

## Prerequisites

- Swift concurrency (Module 02)
- Error handling & Result type (Module 01)
- Codable (Module 01)

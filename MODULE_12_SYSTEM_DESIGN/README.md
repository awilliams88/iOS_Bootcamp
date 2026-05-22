# Module 12 — System Design

System design interviews test your ability to architect scalable, production-ready iOS applications. This module covers the high-level patterns that come up in senior interviews.

---

## Topics

| # | File | Description |
|---|------|-------------|
| 01 | [Scalable Architecture Patterns](01-scalable-architecture.md) | Modularization, caching, pagination, offline-first |
| 02 | [Feed & Chat App Design](02-feed-and-chat-design.md) | Social feed design, chat architecture, real-time messaging |
| 03 | [Upload & Analytics Architecture](03-upload-and-analytics.md) | Photo upload, event batching, analytics pipeline |

---

## System Design Interview Framework

When asked to design an iOS system, structure your answer:

1. **Clarify requirements** — feature scope, user count, constraints
2. **Define the API/data model** — what data flows
3. **Component design** — layers, services, storage
4. **Caching strategy** — what to cache, where, invalidation
5. **Offline strategy** — optimistic updates, sync
6. **Error handling** — retry, failure states
7. **Performance** — pagination, lazy loading, memory
8. **Testing strategy** — what to test

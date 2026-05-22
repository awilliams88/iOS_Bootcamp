# Authentication, Pagination, Retries, and Caching

## Authentication

Common flow:

```text
Login -> access token -> API request -> token expiry -> refresh token -> retry
```

## Pagination

Load large datasets incrementally.

Strategies:
- page number
- cursor pagination
- offset pagination

## Retries

Use transient retry logic for temporary failures.

Prefer exponential backoff.

## Caching

Improve speed and reduce network calls.

Examples:
- URLCache
- disk cache
- in-memory cache

## Interview Answer

Production apps need resilient networking with auth refresh, pagination support, retry strategy, and layered caching.

# Coding standards

Conventions for the **modern** playground. Book `ChapterNN/` samples may differ—match the chapter when editing those trees; follow this file when building the modern root.

## Java

- Prefer clear, idiomatic Java over cleverness.
- Avoid `null` leaks at API boundaries; use explicit validation and clear error responses.
- Keep methods and classes focused; extract when a type has more than one reason to change.
- Prefer records/DTOs for API payloads where appropriate.

## Spring Boot

- Use **Spring MVC** (`spring-boot-starter-web`), not WebFlux.
- Constructor injection; avoid field injection.
- Configuration via `application.yml` + profiles + env; no hardcoded secrets.
- Actuator for health/info (expand in observability phases).
- Prefer existing starters already chosen in [`project-overview.md`](project-overview.md) over new frameworks without discussion.

## Modules

- Root folder: **`product-hub/`**.
- **Service modules only** (`product-service`, `recommendation-service`, `review-service`, `product-composite-service`).
- **No** shared `libs/common`, `libs/api`, or similar cross-service library modules.
- DTOs, Feign clients, exception handlers, and utilities live **inside** the service that needs them (some duplication across services is acceptable).

## Architecture Rules — package layout (DDD-style)

Follow Domain-Driven Design layered packages inside each service:

| Package | Responsibility |
|---------|----------------|
| `controller` | HTTP adapters (`@RestController`), request/response mapping at the edge |
| `service` | Application / domain use-cases and business rules |
| `repository` | Persistence ports (Spring Data repositories, etc.) |
| `entity` | JPA (or persistence) entities |
| `dto` | Request/response and transfer objects (not entities) |

Base package root: **`com.minhhung.producthub`**.

```text
com.minhhung.producthub.product
  controller/
  service/
  repository/
  entity/
  dto/
  config/          # optional
  client/          # optional — Feign interfaces (composite uses …composite.client)
  exception/       # optional
  mapper/          # optional
```

Same pattern for `…recommendation`, `…review`, `…composite`.

Rules:

- Controllers depend on services; services depend on repositories — not the reverse.
- Do not expose `entity` types directly on public HTTP APIs; map to `dto`.
- Keep Feign API models in the calling service’s `dto` / `client` packages (no shared api jar).

## Architecture Rules — entity `equals` / `hashCode`

- Every **entity** class must define `equals()` and `hashCode()`.
- Base equality on **business keys** (natural identity used by the domain), not on transient state.
- Do **not** use only a database-generated surrogate `@Id` when that id is null before persist (broken equality in sets/maps). Prefer a stable business key (e.g. `productId`, or a documented natural unique key).
- Keep `equals` / `hashCode` consistent with each other and with the chosen business key fields only.
- DTOs may use records (implicit equality) or explicit equality as needed; the hard rule above applies to **entities**.

## HTTP & Feign

- Composite (BFF) calls core services via **OpenFeign**.
- Keep Feign interfaces small and explicit; map errors to clear API error payloads inside that service.
- Do not call other services’ databases; schemas are isolated by convention.

## Persistence

- One Postgres instance; **one schema per core service**.
- Flyway migrations owned by each service (or clearly namespaced).
- Prefer Testcontainers Postgres for integration tests.
- No MongoDB/MySQL in the modern stack.

## Messaging (when Phase 5b starts)

- **Kafka + Avro** only; **`spring-kafka`** only.
- No RabbitMQ; no Spring Cloud Stream / binders.
- Schema registry in Compose when events are introduced.

## Testing

- Unit tests for domain/logic; slice or integration tests for web/persistence.
- Prefer extending the project’s existing test style over a parallel stack.
- Do not add heavy E2E frameworks unless requested.

## API docs

- Document public/composite APIs with OpenAPI (springdoc); keep annotations accurate when contracts change.

## What not to do

- Do not introduce deferred or out-of-scope tech from [`project-overview.md`](project-overview.md) without an explicit ask.
- Do not add a shared `common` / `api` module.
- Do not “sync” modern refactors into every book chapter folder.
- Do not commit `.env`, credentials, or generated `build/` / `.gradle/` caches.

# Coding standards

Conventions for the **modern** playground (`product-hub/`). Book `ChapterNN/` samples may differ—match the chapter when editing those trees; follow this file when building the modern root.

Locked stack and deferred tech live in [`project-overview.md`](project-overview.md). Prefer that file over generic Spring advice when they conflict.

## Principles

- Prefer clear, idiomatic Java over cleverness.
- Favor SOLID, DRY, KISS, and YAGNI at the service level—do not invent abstractions “for reuse later.”
- High cohesion, low coupling: controllers → services → repositories (not the reverse).
- Smallest change that completes the current feature; no drive-by refactors.
- Follow OWASP-minded habits for auth, input validation, and secrets (especially from Phase 3 onward).

## Java

- Align language level with the scaffold (book samples use a recent JDK; do not silently downgrade without discussion).
- Prefer records for immutable API DTOs where appropriate.
- Avoid `null` leaks at API boundaries; validate inputs (`@Valid` / Bean Validation) and return clear error responses.
- Keep methods and classes focused; extract when a type has more than one reason to change.
- Use descriptive names: PascalCase types, camelCase methods/fields, `UPPER_SNAKE` constants.

## Spring Boot

- Use **Spring Boot + Spring MVC** (`spring-boot-starter-web`) — **not** WebFlux / reactive stacks.
- Constructor injection only; avoid field injection.
- Prefer starters already chosen in the overview; do not add frameworks without discussion.
- Use Spring Boot auto-configuration; keep custom `@Configuration` thin and intentional.
- Centralize HTTP exception mapping with `@RestControllerAdvice` / `@ExceptionHandler` **inside each service** (no shared exception jar).
- Prefer `@ConfigurationProperties` for typed config over scattered `@Value` when a group of properties grows.
- Configuration via `application.yml` + Spring profiles + env; no hardcoded secrets.
- Actuator for health/info early; expand metrics/tracing in observability phases.

## Modules

- Root folder: **`product-hub/`**.
- **Service modules only** (`product-service`, `recommendation-service`, `review-service`, `product-composite-service`).
- **No** shared `libs/common`, `libs/api`, or similar cross-service library modules.
- DTOs, Feign clients, exception handlers, and utilities live **inside** the service that needs them (some duplication across services is acceptable).
- Build with **Gradle** (multi-module). Do not introduce Maven for the modern root.

## Package layout (DDD-style)

| Package | Responsibility |
|---------|----------------|
| `controller` | HTTP adapters (`@RestController`); map request/response at the edge |
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
- Do not expose `entity` types on public HTTP APIs; map to `dto`.
- Keep Feign API models in the calling service’s `dto` / `client` packages (no shared api jar).
- Do not rename packages to generic `model` / `models`; use `entity` + `dto` as above.

## Entity `equals` / `hashCode`

- Every **entity** class must define `equals()` and `hashCode()`.
- Base equality on **business keys** (natural identity used by the domain), not on transient state.
- Do **not** use only a database-generated surrogate `@Id` when that id is null before persist (broken equality in sets/maps). Prefer a stable business key (e.g. `productId`, or a documented natural unique key).
- Keep `equals` / `hashCode` consistent with each other and with the chosen business key fields only.
- DTOs may use records (implicit equality) or explicit equality as needed; the hard rule above applies to **entities**.

## HTTP & Feign

- Design REST resources with correct HTTP methods and status codes.
- Composite (BFF) calls core services via **OpenFeign**.
- Keep Feign interfaces small and explicit; map errors to clear API error payloads inside that service.
- Do not call other services’ databases; schemas are isolated by convention.

## Persistence

- One Postgres instance; **one schema per core service**.
- Prefer **Spring Data JPA** for persistence once Phase 2 starts.
- **Flyway** migrations owned by each service (or clearly namespaced). Prefer Flyway over Liquibase unless there is a strong reason to switch.
- Prefer Testcontainers Postgres for integration tests.
- No MongoDB / MySQL in the modern stack.
- Entity relationships and cascading: keep them intentional and documented; do not cascade “everything” by default.

## Messaging (when Phase 5b starts)

- **Kafka + Avro** only; **`spring-kafka`** only.
- No RabbitMQ; no Spring Cloud Stream / binders.
- Schema registry in Compose when events are introduced.

## Security (when Phase 3 starts)

- Use **Spring Security** as planned in the overview (JWT resource server first; external IdP later).
- Do not invent a custom auth scheme or pull Keycloak/Auth0 early unless asked.
- Encode secrets/passwords with standard Spring Security mechanisms when local users exist; never commit credentials.
- Add CORS only when a real browser client needs it—do not enable wide-open CORS by default.

## Logging & observability

- Use **SLF4J** (Logback) for logging; prefer parameterized messages over string concatenation.
- Use log levels deliberately (`ERROR` / `WARN` / `INFO` / `DEBUG`); avoid noisy INFO in hot paths.
- Actuator for health/info early; Micrometer / tracing in Phase 4—do not bolt on a full observability stack ahead of the roadmap.

## Testing

- Unit tests for domain/logic (JUnit 5).
- Web layer: MockMvc (or equivalent Spring MVC test support) for controllers.
- Persistence: `@DataJpaTest` and/or Testcontainers where useful; broader `@SpringBootTest` when wiring matters.
- Prefer extending the project’s existing test style over a parallel stack.
- Do not add heavy E2E frameworks unless requested.

## API docs

- Document public/composite APIs with **springdoc OpenAPI**; keep annotations accurate when contracts change.

## Performance (only when needed)

- Do **not** add caching (`@Cacheable`), `@Async`, or speculative indexing as default scaffolding.
- Introduce caching / async / query tuning with a measured need and a short note in the active feature or a research doc.

## What not to do

- Do not introduce deferred or out-of-scope tech from [`project-overview.md`](project-overview.md) without an explicit ask.
- Do not add a shared `common` / `api` module.
- Do not use WebFlux, reactive drivers, Maven, RabbitMQ, or Spring Cloud Stream in the modern root.
- Do not “sync” modern refactors into every book chapter folder.
- Do not commit `.env`, credentials, or generated `build/` / `.gradle/` caches.

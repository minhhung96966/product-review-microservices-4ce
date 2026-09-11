# AGENTS.md

Instructions for AI coding agents working in this repository.

## Project overview

Forked learning playground based on *Microservices with Spring Boot and Spring Cloud* (Packt, 4th edition / Magnus Larsson).

Purpose: practice senior-level Java and Spring skills—refactoring, design, tech choices, and production concerns—not only follow the book verbatim.

**Context pack (start here for the modern project):** [`context/README.md`](context/README.md)

| Doc | Use |
|-----|-----|
| [`context/project-overview.md`](context/project-overview.md) | Goals, stack, architecture, phases, deferred/out of scope, decision log |
| [`context/coding-standards.md`](context/coding-standards.md) | Java / Spring conventions for the modern root |
| [`context/ai-interaction.md`](context/ai-interaction.md) | Workflow with the AI |
| [`context/current-feature.md`](context/current-feature.md) | What is in progress now |

Book chapter samples use Spring Boot **3.5.x**, Spring Cloud **2025.x**, Java **24**, Gradle, **WebFlux**, and packages under `se.magnus.*`. The modern project intentionally diverges—see the overview.

## Repository layout

| Path | Role |
|------|------|
| `context/` | Project context pack (overview, standards, features/fixes/research) |
| `product-hub/` | Modern playground (Phase 0+; create when scaffolding) |
| `ChapterNN/` | Book chapter snapshots (reference / learning) |
| `Chapter03/1-…`, `2-…` | Early chapters may have multiple step folders |
| `microservices/` (inside a chapter) | Core/composite services |
| `api/`, `util/` (inside a chapter) | Shared API contracts and utilities |
| `spring-cloud/` | Gateway, config, discovery, auth (book samples) |
| `kubernetes/`, `docker-compose*.yml`, `config-repo/` | Deploy and config assets (later chapters) |

Each chapter folder is largely independent (own Gradle build). Prefer working inside one chapter **or** `product-hub/` at a time.

Until `product-hub/` exists, **ask which `ChapterNN` to use** if the task does not name one. After it exists, default modern-feature work to `product-hub/` unless the user asks for a book chapter.

## Chapter map

Folders present in this repo. Later chapters build on earlier ones; pick the chapter that matches the feature under discussion. For how chapters map to the modern rebuild, see [`context/project-overview.md`](context/project-overview.md).

### Part 1 — Cooperating microservices

| Folder | Focus |
|--------|--------|
| `Chapter03/` | Create cooperating microservices from scratch (Spring Initializr). Three core services + one composite aggregator; basic WebFlux REST APIs. Has step folders (`1-spring-init`, `2-basic-rest-services`). |
| `Chapter04/` | Dockerize the landscape: Dockerfiles, docker-compose, Spring profiles for with/without Docker. |
| `Chapter05/` | OpenAPI docs via springdoc-openapi; try APIs in Swagger UI. |
| `Chapter06/` | Persistence with Spring Data: MongoDB (two core services), MySQL (one core service); Testcontainers for integration tests. |
| `Chapter07/` | Reactive end-to-end: non-blocking REST + async event-driven services; reactive MongoDB driver; blocking MySQL where used. |

### Part 2 — Spring Cloud

| Folder | Focus |
|--------|--------|
| `Chapter09/` | Service discovery with Netflix Eureka + Spring Cloud LoadBalancer. |
| `Chapter10/` | Spring Cloud Gateway as edge server; expose only selected public APIs. |
| `Chapter11/` | Secure APIs with OAuth 2.1 / OpenID Connect (Spring Authorization Server, HTTPS via edge, optional Auth0). |
| `Chapter12/` | Centralized config with Spring Cloud Config Server + shared config repo. |
| `Chapter13/` | Resilience4j: retries, circuit breaker, fail-fast, fallbacks on the composite service. |
| `Chapter14/` | Distributed tracing with Micrometer Tracing + Zipkin. |

Book Chapter 8 is Spring Cloud intro only—no `Chapter08/` folder here.

### Part 3 — Kubernetes and production concerns

| Folder | Focus |
|--------|--------|
| `Chapter15/` | Kubernetes basics; local Minikube for dev/test. |
| `Chapter16/` | Deploy to Kubernetes with Helm (test/prod-style envs); replace Eureka with Kubernetes Service / kube-proxy discovery. |
| `Chapter17/` | Simplify landscape: ConfigMaps/Secrets instead of Config Server; Ingress (+ cert-manager) instead of Spring Cloud Gateway. |
| `Chapter18/` | Istio service mesh: resilience, security, traffic management, observability. |
| `Chapter19/` | Centralized logging with EFK (Elasticsearch, Fluentd, Kibana) on Minikube. |
| `Chapter20/` | Monitoring with Prometheus + Grafana (dashboards and alerts). |
| `Chapter23/` | Native compiled Spring Boot microservices (GraalVM Native Image) for near-instant startup. |

Chapters 21–22 from the book are not present as folders in this repo.

## How to work here

1. **Scope** — Edit only the chapter or modern folder the user named. Do not sync the same change across every chapter unless asked.
2. **Preserve book snapshots** — Large style/design/tech changes go in the modern root (see overview), not by rewriting all historical chapter trees.
3. **Follow the right conventions** — Book chapters: match that chapter. Modern project: [`context/project-overview.md`](context/project-overview.md) + [`context/coding-standards.md`](context/coding-standards.md).
4. **Keep changes focused** — No drive-by refactors, unrelated file churn, or new docs the user did not request.
5. **Secrets** — Never commit `.env`, credentials, tokens, or kube/auth secrets.
6. **Build artifacts** — Ignore `.gradle/`, `build/`, and similar caches; do not commit them.
7. **Deferred tech** — Do not add items listed as deferred/out of scope in the overview unless the user explicitly asks.
8. **Active work** — Keep [`context/current-feature.md`](context/current-feature.md) updated; put non-trivial specs under `context/features/` or `context/fixes/`.

## Coding expectations

### Book chapter folders

- Prefer clear, idiomatic Java/Spring matching the chapter’s existing stack (often WebFlux, mixed datastores, Spring Cloud).
- Keep public APIs/DTOs in `api` when present; helpers in `util`.
- Extend the chapter’s test approach rather than inventing a parallel stack.

### Modern project (when present)

- Follow [`context/coding-standards.md`](context/coding-standards.md).
- Spring Boot + **Spring MVC**; OpenFeign; single Postgres with multiple schemas; composite-as-BFF before Kong.
- Root: **`product-hub/`**; packages: **`com.minhhung.producthub.<service>`**.
- **Service modules only** — no shared `common`/`api` libs; DDD packages (`controller`, `service`, `repository`, `entity`, `dto`); entity `equals`/`hashCode` on business keys.
- Messaging when introduced: **Kafka + Avro** with `spring-kafka` only — **no** RabbitMQ, **no** Spring Cloud Stream.

## Agent do / don’t

**Do**

- Read `context/README.md` and `context/project-overview.md` before modern-project work.
- Confirm target folder before large edits.
- Summarize trade-offs briefly when proposing senior-level refactors.
- Run builds/tests in the folder being changed (`./gradlew` / `gradlew.bat`).

**Don’t**

- Assume every chapter should stay identical after a refactor.
- Mass-edit all chapters for a single learning experiment.
- Treat Packt marketing assets as application requirements unless relevant.
- Invent architecture that ignores the locked decisions in the project overview.

## Verification hints

From the active chapter or modern root:

```bash
./gradlew test
# Windows: gradlew.bat test
```

Later book chapters may use Docker Compose, Helm, or `test-em-all.bash`. Use what that folder already provides.

## Communication

- Follow [`context/ai-interaction.md`](context/ai-interaction.md).
- Be concise and direct.
- Prefer actionable next steps over long restatements of the book.
- When practicing for senior interviews, explain *why* a design choice matters when asked for review or refactor guidance.

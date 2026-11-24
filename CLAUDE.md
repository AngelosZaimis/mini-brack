# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

MiniBrack: learning project, a small e-commerce catalog + flash-deal platform. Java 25, Spring Boot 4, PostgreSQL
(later Kafka, Solr, Airflow, nginx). Currently one module: `services/catalog-service`. Decisions live in `docs/adr/`;
read the relevant ADR before changing build, schema or id conventions.

## Commands

```bash
./gradlew build                                   # compile, test, spotlessCheck
./gradlew spotlessApply                           # format (palantir-java-format); run before committing
./gradlew :services:catalog-service:test          # one module's tests
./gradlew :services:catalog-service:test --tests 'ch.minibrack.catalog.CatalogServiceApplicationTests'
./gradlew :services:catalog-service:bootRun       # local profile; starts Postgres via infra/compose.yaml
docker compose -f infra/compose.yaml stop         # Postgres keeps running after bootRun exits
```

Docker is required for both `bootRun` (Spring Boot Docker Compose support) and tests (Testcontainers).
Local DB: `localhost:5433`, db/user/password `catalog`.

## Build structure

- `build-logic/` is an included build with convention plugins:
  - `minibrack.java-conventions`: toolchain, `-parameters`, `-Xlint:all`, **`-Werror` on main code only**, JUnit via
    `junit-bom`, Spotless.
  - `minibrack.spring-service`: java-conventions + Boot plugin + Boot BOM via Gradle `platform()` (no
    `io.spring.dependency-management`). Sets `spring.profiles.default=local` for `bootRun` only.
- All versions live in `gradle/libs.versions.toml`. Module build files never contain version numbers; BOM-managed libs
  are listed without a version. Convention plugins read the catalog by name (`libs.findVersion(...)`), not via
  type-safe accessors.
- Repositories are declared only in `settings.gradle.kts` (`FAIL_ON_PROJECT_REPOS`). No `allprojects`/`subprojects`.
- New module: `include(":services:x")` in `settings.gradle.kts` + a build file applying `minibrack.spring-service`.
- New compiler warnings in main code fail the build; suppress only with targeted `@SuppressWarnings` and a reason.

## Code conventions

- **Package by feature**, not layer: `ch.minibrack.catalog.product`, `.category`, `.pricing`. Controller, service,
  repository, entity of a feature sit side by side; prefer package-private visibility. No `controller/`/`service/`
  packages.
- **Schema owned by Flyway**: `src/main/resources/db/migration/V<int>__<desc>.sql`. Hibernate runs `ddl-auto: validate`.
  Never edit an applied migration; fix with a new one. Changes to live tables follow expand/contract.
- **IDs are UUID v7**, generated in the app (`@UuidGenerator(style = VERSION_7)`), Postgres `uuid` column, no DB
  default.
- `open-in-view: false`. Base `application.yml` holds no secrets/env-specific values (those come from env vars);
  dev-only settings go in `application-local.yml`.
- Postgres image `postgres:17` is pinned in both `infra/compose.yaml` and the test `TestcontainersConfiguration`; keep
  them in sync.
- Integration tests: `@SpringBootTest` + `@Import(TestcontainersConfiguration.class)` (`@ServiceConnection`, no
  datasource properties needed).

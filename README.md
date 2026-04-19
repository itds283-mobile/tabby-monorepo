# Tabby

Tabby is a bill-splitting app with a hybrid architecture:

- Local-first on mobile for drafts and offline workflows.
- Server-authoritative for financial finalization and payment verification.

This repository contains:

- `tabby-infra/`: Kotlin + Spring Boot backend, PostgreSQL, RabbitMQ.
- `tabby/`: Flutter mobile app (Android and iOS).

## Project Layout

- `tabby-infra/`: backend APIs, domain logic, data persistence.
- `tabby/`: frontend app, local database, API integrations.
- `tabby/packages/tabby_api/`: generated Dart API client from OpenAPI.

## Monorepo and Submodules

This repository is a superproject that tracks two Git submodules:

- `tabby` (frontend)
- `tabby-infra` (backend)

### Clone with submodules

```bash
git clone --recurse-submodules https://github.com/itds283-mobile/tabby-monorepo.git
cd tabby-monorepo
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```

### Pull latest changes

```bash
git pull
git submodule update --init --recursive
```

### Update a submodule to latest remote commit

Example for frontend:

```bash
git submodule update --remote tabby
```

Example for backend:

```bash
git submodule update --remote tabby-infra
```

After updating submodules, commit the superproject so the new submodule commit pointers are recorded.

## Prerequisites

Install these tools first:

- JDK 21+
- Docker and Docker Compose
- Flutter SDK
- Dart SDK (bundled with Flutter)

Recommended checks:

```bash
java -version
docker --version
docker compose version
flutter --version
dart --version
```

## Backend Setup (Detailed)

Backend source: `tabby-infra/`

### 1. Enter backend directory

```bash
cd tabby-infra
```

### 2. Configure environment variables

The backend expects database credentials and integration keys in environment variables.

At minimum, make sure `DB_PASSWORD` is set before running database migration tasks.

If you have a `.env` file in `tabby-infra/`, load it in your shell before running Gradle tasks:

```bash
set -a
source .env
set +a
```

### 3. Start infrastructure services

Use Docker Compose to start local services (PostgreSQL and RabbitMQ). Depending on compose config, it may also start the backend container.

```bash
docker compose up -d
```

To inspect running services:

```bash
docker compose ps
```

To stop:

```bash
docker compose down
```

### 4. Run backend locally (development)

```bash
./gradlew bootRun
```

Default API port is `3000`.

### 5. Build backend artifact

```bash
./gradlew bootJar -x test --no-daemon
```

### 6. Verify backend compiles cleanly

```bash
./gradlew check -x test
```

### 7. Database reset/migration (when needed)

If you need a fresh local schema:

```bash
./gradlew dropAll update
```

Important:

- Use `dropAll update` for this project (not `liquibaseDropAll` / `liquibaseUpdate`).
- Ensure `DB_PASSWORD` is set (or `.env` is sourced) first.

### 8. OpenAPI spec endpoint

When backend is running, OpenAPI docs are served at:

- `http://localhost:3000/v3/api-docs`

This endpoint is the source for regenerating the frontend API client.

## Frontend Build Guide

Frontend source: `tabby/`

### 1. Enter frontend directory

```bash
cd tabby
```

### 2. Install dependencies

```bash
flutter pub get
```

### 3. Generate code

Run this whenever you change Riverpod/Freezed/Drift annotated files or model definitions:

```bash
dart run build_runner build --delete-conflicting-outputs
```

### 4. Static analysis

```bash
flutter analyze
```

### 5. Run tests

```bash
flutter test
```

Single test example:

```bash
flutter test test/path/to/widget_test.dart
```

### 6. Build app artifacts

Android APK:

```bash
flutter build apk
```

Android App Bundle:

```bash
flutter build appbundle
```

iOS (requires Xcode/macOS):

```bash
flutter build ios
```

## OpenAPI Client Regeneration Flow

Run this flow after backend DTO or endpoint changes.

1. Start backend and ensure `http://localhost:3000/v3/api-docs` is reachable.
2. Regenerate client with repository OpenAPI generator config.
3. Rebuild generated code in API package:

```bash
cd tabby/packages/tabby_api
dart run build_runner build --delete-conflicting-outputs
```

4. Rebuild generated code in frontend app:

```bash
cd ../../
dart run build_runner build --delete-conflicting-outputs
flutter analyze
```

## Architecture Notes

### Local-first vs server-authoritative

- Local-first: drafting tabs, local participant edits, offline data handling.
- Server-authoritative: tab finalization, settlement status changes, payment verification.

The backend must reject sync payloads that attempt to mutate finalized tabs or settlement status directly.

### Identity model

- Registered users: have backend `userId`, sync globally.
- Guest participants: local-only, `null` `userId`, do not sync.

## Development Conventions

- Keep backend entities separate from API DTOs.
- Use `BigDecimal` for monetary values in backend DTOs.
- Return typed DTO responses from controllers.
- Handle backend errors through custom exceptions and centralized exception mapping.
- Do not hand-edit generated files in `tabby/packages/tabby_api/`.

## Common Backend Troubleshooting

- Migration command fails with password issues:
  - Confirm `DB_PASSWORD` is exported.
  - Re-source `.env` and rerun.

- Docker services start but backend cannot connect:
  - Check `docker compose ps` status.
  - Verify backend DB/RabbitMQ host and port environment values.

- Build works but app integration fails after API changes:
  - Regenerate OpenAPI client and rerun both build_runner steps.

## License

No license file is currently defined in this repository.
# Stack Recipes

Use these as starting points after code is uploaded or cloned into the TryCase repo directory. Prefer the project's own README, scripts, Compose file, or lockfile over generic commands.

## Node, Bun, PNPM, Yarn

Inspect `package.json` scripts and lockfiles. Prefer the package manager implied by `bun.lock`, `pnpm-lock.yaml`, `yarn.lock`, or `package-lock.json`.

Size note: use `small` for clearly lightweight apps, `standard` for unknown Next/Vite/full-stack apps, and `large` for monorepos or native dependency builds.

Common commands:

```bash
bun install && bun run dev
pnpm install && pnpm run dev
npm install && npm run dev
yarn install && yarn dev
```

If the dev server binds only to localhost inside the environment, that is fine for TryCase browser checks. If the app needs external access from the visible desktop/browser, bind to `0.0.0.0` when the framework supports it.

## Docker Compose

Prefer Compose when a database, queue, cache, or multi-service app is present.

Size note: use at least `standard`. Use `large` when Compose includes multiple databases, search engines, queues, browsers, or sizeable fixtures.

```bash
docker compose up -d --remove-orphans
docker compose ps
docker compose logs --tail=120
```

If ports conflict, inspect `docker compose ps` and app logs before changing ports. Use `curl` inside the environment to verify the service URL.

## Python

Check for `pyproject.toml`, `requirements.txt`, `uv.lock`, `Pipfile`, or framework files.

Size note: use `small` for simple Flask/FastAPI scripts, `standard` for Django or unknown web apps, and `large` when native wheels, data fixtures, or browser testing are involved.

Common commands:

```bash
python -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
python manage.py runserver 0.0.0.0:8000
flask --app app run --host 0.0.0.0 --port 5000
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

For Django, check migrations and required env vars before declaring failure.

## Go

Size note: use `small` for small services and `standard` or `large` for larger modules, cgo, or substantial test suites.

Use:

```bash
go test ./...
go run ./...
```

If the app has multiple commands, inspect `cmd/`, README, Makefile, and Dockerfile.

## Rust

Size note: use `standard` for ordinary crates and `large` for workspaces or native-heavy builds. Rust compiles can be CPU and disk intensive.

Use:

```bash
cargo test
cargo run
```

Rust builds can be slow. Report progress and check `trycase env metrics <env>` if CPU, memory, or disk pressure appears.

## Ruby On Rails

Use the project README first. Common flow:

Size note: use `standard` for most Rails apps and `large` when native gems, asset builds, Postgres, Redis, or large test suites are involved.

```bash
bundle install
bin/rails db:prepare
bin/rails server -b 0.0.0.0 -p 3000
```

If native gems fail, read the error for missing system packages. If the app needs Postgres or Redis, prefer Docker Compose when available.

## PHP And Laravel

Size note: use `small` for simple PHP apps and `standard` for Laravel or apps with databases, queues, or asset builds.

Common flow:

```bash
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan serve --host 0.0.0.0 --port 8000
```

Use project secrets for real env values instead of committing or uploading private `.env` files.

## JVM, Gradle, Maven

Size note: use `large` for JVM, Android, and multi-module Gradle builds; it is currently the largest available size, so prefer it up front for builds likely to hit memory or disk pressure.

Use wrapper scripts when present:

```bash
./gradlew test
./gradlew bootRun
./mvnw test
./mvnw spring-boot:run
```

JVM apps may need more memory. If builds are killed or swap heavily, check metrics and suggest a larger environment size.

## Android Builds

TryCase Linux environments can run Gradle and SDK-backed build/test commands when the project supplies or installs required SDK pieces. Do not assume emulator/device availability. Prefer:

```bash
./gradlew test
./gradlew assembleDebug
```

Report missing SDK/licenses clearly and ask before long downloads.

---
name: trycase-run-linux-app
description: Run, preview, verify, and debug arbitrary Linux-compatible applications inside TryCase so an LLM can return working code with proof instead of asking the user to test manually. Use when the user asks an agent to get a local repo or uploaded code running in TryCase; test the current branch, current working tree, local or uncommitted changes, or user-specified changes; detect and run Node, Bun, Python, Go, Rust, Rails, PHP, JVM, Docker Compose, Android build, or other Linux app stacks; open a visible desktop/browser preview; diagnose install/start/port/env-var/resource failures; collect screenshots, recordings, logs, metrics, and artifacts; or produce a reusable run recipe after the app works.
---

# Run Linux Apps In TryCase

## Overview

Get the user's app to a visible, verified "it works" state inside TryCase. Prefer the fastest path to a working preview, then summarize exact commands and proof. Recommend GitHub-backed setup only after value is demonstrated or when the user asks for repeatable branch/PR environments.

Use TryCase by default when the user asks to test the current branch, current working tree, local changes, uncommitted changes, or specific changes they want verified, unless they explicitly say not to use TryCase.

Use with `trycase-cli` when you need to install or invoke the TryCase CLI, create environments, upload code, configure project secrets, capture evidence, or clean up. If the `trycase` command is missing, the LLM should install the published CLI itself or use one-off commands such as `npx trycase@latest <command>` before continuing.

## Operating Loop

1. Identify whether an environment already exists. If not, use the upload-first TryCase route for local code.
2. For "test this branch" or "test my changes", inspect `git status` and `git branch --show-current`; include local/uncommitted changes so the environment matches what the user asked to verify.
3. Inspect the project before running broad installs. Read README, package files, Compose files, lockfiles, Makefile, Dockerfile, and existing scripts.
4. Choose the narrowest likely run path, then execute it inside TryCase.
5. Keep long-running servers in a persistent terminal session; use `env exec --wait` for short checks.
6. Verify with `curl`, browser navigation, browser snapshot, screenshot, console/network output, and desktop view.
7. If it fails, iterate from evidence: logs, exit codes, missing env vars, port binding, disk/memory/CPU metrics.
8. End with working URL, run commands, evidence artifacts, resource notes, and cleanup status.

For stack-specific command starting points, read `references/stack-recipes.md`.

## Project Inspection

If local files are available, inspect locally before upload. If code is already in TryCase, inspect remotely:

```bash
trycase env exec <env> "pwd && find . -maxdepth 2 -type f | sort | sed -n '1,160p'" --wait
trycase fs read <env> package.json
trycase fs read <env> README.md
```

Look for:

- package manager lockfiles and scripts
- Docker Compose services, ports, volumes, and healthchecks
- framework conventions and documented dev commands
- env examples such as `.env.example`
- databases, queues, caches, and migrations
- expected ports and host binding options

## Run The App

Use a visible desktop when the user is watching the preview link:

```bash
trycase env view <env>
trycase desktop app launch <env> terminal
trycase desktop app launch <env> browser http://localhost:3000
```

Use a persistent CLI terminal for a dev server:

```bash
trycase terminal open <env>
trycase terminal write <session> "<install command>" --env <env> --enter --wait-for "<expected text>"
trycase terminal write <session> "<start command>" --env <env> --enter
trycase terminal read <session> --env <env>
```

Use one-shot execution for tests and probes:

```bash
trycase env exec <env> "npm test" --wait --timeout 600
trycase env exec <env> "curl -fsS http://localhost:3000 || true" --wait
```

## Verify

Verify both headless browser capability and visible desktop when useful:

```bash
trycase computer browser goto <env> http://localhost:3000
trycase computer browser snapshot <env>
trycase computer browser console <env>
trycase computer browser network <env>
trycase computer browser screenshot <env>
trycase computer browser recording <env>
trycase desktop screenshot <env>
trycase desktop recording start <env>
trycase desktop recording stop <env>
```

Treat a green terminal command alone as insufficient for web apps. Load the app, inspect visible content, and capture at least one screenshot and one video recording unless the user explicitly says not to, or the capability is unavailable. If any evidence is skipped, say why.

## Debug Failures

Common failure paths:

- Missing secrets: inspect `.env.example`, README, framework error output, and current project secret config. Ask before importing local dotenv files.
- Port not reachable: confirm process is running, port is listening, and the browser uses the right localhost URL.
- Long install/build: keep reading terminal output and check `trycase env metrics <env>`.
- Disk pressure: check upload output and `trycase env metrics <env>`; suggest `--respect-gitignore`, fewer files, or a larger environment.
- Docker failure: use `docker compose ps` and `docker compose logs --tail=120`.
- Browser failure: collect console and network output before changing code.

Do not edit application code as the first response to an environment setup issue. Prefer configuration, documented commands, missing services, missing env vars, and dependency setup first.

## Finish

When the app works, report:

- environment ID and live desktop URL
- app URL and exact commands that worked
- evidence captured, such as screenshots, recordings, or artifact IDs
- secrets/files added without exposing values
- resource warnings, if any
- whether GitHub-backed setup would now save time

If the task is complete, destroy the environment and confirm `STOPPED - NOT BILLABLE` unless the user explicitly wants to keep it running.

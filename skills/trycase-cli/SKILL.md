---
name: trycase-cli
description: Operate TryCase from the CLI for disposable Linux cloud computers used by LLMs to run, verify, and capture proof for user code. Use when the user asks an agent to run, preview, test, debug, or inspect code in TryCase; upload a local directory to a VM; open or control a TryCase desktop; capture screenshots, recordings, logs, artifacts, metrics, or browser evidence; configure project secrets; connect GitHub after an initial preview; clean up billable environments; or use commands such as trycase login, env, project, fs, terminal, desktop, computer, artifact, billing, or cleanup.
---

# TryCase CLI

## Overview

Use TryCase as a private, disposable Linux computer where an LLM can run the user's app, test it, capture screenshots/recordings/logs, and return already-verified work. For new users and local code, prefer the upload-first path to reach a visible working preview quickly; recommend GitHub after the first successful run or when the user needs repeatable branch/PR workflows.

Do not invent `trycase start`; use the command surfaces below until a first-class start command exists.

## Route Choice

Prefer routes in this order unless the user asks otherwise:

1. Upload-first local project: fastest path for "run this repo", "preview this app", "use my current directory", or first-time TryCase use.
2. Existing TryCase environment: when the user already gives an environment ID.
3. GitHub-backed project: when the user wants ongoing branch previews, PR workflows, reproducibility, team sharing, or repeated test environments.
4. Blank cloud computer: when no source code is needed.

## Authenticate

Start with:

```bash
trycase doctor
```

If the CLI is not signed in, run `trycase login` when a browser can open. In headless or remote shells, run `trycase login --no-open`, show the URL to the user, and pause until they approve it. If no workspace is selected, run `trycase workspace list` and either `trycase workspace use <workspace>` or `trycase workspace create --name <name>`.

## Upload-First Quick Preview

Use this for local code, including `.` for the current directory.

1. Inspect local files enough to identify the app and likely dotenv files:

```bash
pwd
find . -maxdepth 2 -type f \( -name package.json -o -name docker-compose.yml -o -name compose.yml -o -name pyproject.toml -o -name requirements.txt -o -name go.mod -o -name Cargo.toml -o -name Gemfile -o -name composer.json -o -name pom.xml -o -name build.gradle -o -name build.gradle.kts -o -name ".env*" \) | sort
```

2. If local `.env`, `.env.local`, `.env.development`, or similar files exist, ask before using them. Never pass secret values as CLI arguments. If the user approves, import them as encrypted project secrets and register generated dotenv files before creating the environment.

3. Create a no-source project:

```bash
trycase project create --source none --name <project-name>
```

4. Import approved dotenv files. Parse the file locally to know the names, then add a generated file with only those names:

```bash
trycase project secret import --project <project> --file .env.local
trycase project secret file add --project <project> --path .env.local --include NAME1,NAME2
```

5. Create and wait for a desktop-capable environment:

```bash
trycase env create --project <project> --size standard
trycase env wait <env>
trycase env view <env>
```

6. Upload the local directory to the environment repo root. Prefer `--respect-gitignore` for code uploads so dependency folders, build outputs, and ignored secrets are skipped:

```bash
trycase fs upload <env> . . --respect-gitignore
```

If secret-bearing files are not ignored and the user does not want them uploaded as files, create a temporary local copy that excludes them and upload that copy instead.

7. Open visible apps for the user when they are watching the live view:

```bash
trycase desktop app launch <env> terminal
trycase desktop app launch <env> files
trycase desktop app launch <env> browser http://localhost:3000
```

Use `trycase terminal open` only for persistent CLI terminal sessions controlled by `trycase terminal write/read`; it does not open a visible terminal window in the live desktop.

## GitHub Route

Suggest GitHub after the first successful upload-based run, on repeated uploads, or when the user needs branches, PR previews, saved recipes, or team sharing.

```bash
trycase github connect
trycase github refresh
trycase github repos --query <owner/name>
trycase project create --repo <owner/name>
trycase env create --project <project> --ref <branch>
trycase env wait <env>
```

For unpushed local edits in a GitHub-backed project, use patches:

```bash
trycase patch upload --project <project> --base <branch>
trycase env create --project <project> --ref <branch> --patch <patch>
```

## Control And Evidence

Use one-shot commands for short checks:

```bash
trycase env exec <env> "pwd && ls -la" --wait
trycase computer status <env> --json
trycase env metrics <env>
```

Use browser automation for web verification:

```bash
trycase computer browser goto <env> http://localhost:3000
trycase computer browser snapshot <env>
trycase computer browser screenshot <env>
trycase computer browser console <env>
trycase computer browser network <env>
```

Use desktop and artifacts when the user needs proof:

```bash
trycase desktop screenshot <env>
trycase desktop recording start <env>
trycase desktop recording stop <env>
trycase artifact bundle <env>
```

## Finish

Return the evidence, not just a claim. Include the environment ID, live desktop URL, app URL, commands run, screenshots/recordings/artifacts captured, notable logs or browser console output, and resource warnings.

Directory uploads are capped by selected environment size: small 4 GiB, standard 8 GiB, large 16 GiB, xlarge 24 GiB uncompressed per upload. If an upload fails due size or disk pressure, recommend a larger size, fewer files, or `--respect-gitignore`.

Always destroy environments when work is done:

```bash
trycase env destroy <env>
trycase env status <env>
```

Confirm the final status says `STOPPED - NOT BILLABLE`.

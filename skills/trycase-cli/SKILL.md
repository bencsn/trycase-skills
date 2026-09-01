---
name: trycase-cli
description: Operate the current TryCase CLI to launch and monitor end-to-end verification agents for connected GitHub pull requests, stream issues and verdicts while work continues, send follow-ups, inspect saved results and proof artifacts, and stop or archive task-owned bots. Use when the user asks to verify a PR with TryCase, inspect a TryCase run, validate a branch that already has a PR, or use trycase commands. Do not use for provisioning cloud computers, uploading a working tree, or controlling environments; those commands belonged to the retired TryCase CLI.
---

# TryCase CLI

Use TryCase to run verification agents against GitHub pull requests and return continuously updated, evidence-backed results. The current CLI is repository-, pull-request-, and bot-oriented. It does not expose the legacy `env`, `project`, `fs`, `terminal`, `desktop`, `computer`, `billing`, or cleanup surfaces.

## Start From The Live Contract

Run these before relying on remembered syntax:

```bash
trycase --version
trycase capabilities --pretty
trycase doctor --pretty
```

Treat `trycase capabilities` as authoritative when it differs from this skill. If `trycase` is missing, use `npx -y trycase@latest` for the task instead of inventing commands or asking the user to install it manually.

The CLI writes a JSON envelope by default, NDJSON events for `verify follow`, and JSON errors to stderr. Parse fields rather than terminal prose.

## Authentication

If `doctor` reports that authentication is missing or invalid, run:

```bash
trycase login
```

Use `trycase login --no-open` when the shell cannot open the user's browser. Show the verification URL and device code, then pause for the user to approve it. Never ask the user to paste a TryCase token into chat.

For non-interactive automation, `TRYCASE_TOKEN` and `TRYCASE_API_URL` must be set together. Never pass a token as a CLI argument or print it. Confirm access with `trycase whoami`.

## Resolve The Target

TryCase verifies connected GitHub repositories and pull requests:

```bash
trycase repository list --pretty
trycase repository show owner/repository --pretty
trycase pr list owner/repository --pretty
trycase pr show owner/repository#123 --pretty
```

Repository references may be an ID, `owner/name`, or GitHub URL. Pull-request references may be an ID, `owner/name#number`, or GitHub pull-request URL.

When run inside a connected Git checkout, repository arguments are optional for commands that advertise `inferredFromGit` in `capabilities`.

If the requested branch has no pushed PR, explain that the current TryCase CLI requires one. Creating or pushing a branch or PR changes GitHub state; do it only when the user's request authorizes that workflow or after obtaining direction. The CLI cannot verify uncommitted local changes directly.

## Reuse Before Starting

Avoid duplicate agents. Inspect existing runs first:

```bash
trycase verify list owner/repository --pretty
trycase verify status <bot-id> --pretty
```

Reuse a matching bot when it targets the requested PR head and is still active or can accept follow-ups. Start a new verification when there is no suitable run or the user explicitly asks for a separate one.

## Start Verification

Put substantial or multiline instructions in a regular file or stdin so they do not enter shell history:

```bash
trycase verify run owner/repository#123 \
  --prompt-file <prompt-path> \
  --request-id <stable-request-id> \
  --pretty
```

Use the same `--request-id` when retrying an uncertain `verify run`; it is the idempotency key that prevents duplicate bots.

Unless the user asks for a different style, tell the verification agent to:

- surface each issue and provisional verdict as soon as it is supported;
- clearly say that work is ongoing until the bot is terminal;
- validate findings continuously instead of saving all validation for the end;
- use native subagents for independent parallel checks when useful;
- keep one primary agent responsible for the final synthesis;
- avoid a time budget or deadline that encourages premature completion.

Subagents are not workers. Do not describe them as TryCase workers, and do not use retired worker or environment commands.

## Follow Continuously

Prefer `follow` when the user wants live progress:

```bash
trycase verify follow <bot-id> --wait-timeout 1800
```

`follow` emits NDJSON whenever the bot state or messages change, followed by a final JSON envelope. Surface meaningful new issues, verdicts, blockers, and state changes to the user as they arrive.

Interpret completion from the bot state, not from the presence of a verdict:

- `terminal: false`: work is still in progress, even if one or more results already exist.
- `terminal: true`: the current queue is finished or the bot is archived.
- `requiresAttention: true`: pause and use `attentionReason` and `actionUrl` to explain the manual action needed.
- message statuses `pending` or `queued`: more work remains.
- `sandboxStatus` describes runtime lifecycle and is not itself a verdict.

Label an early result as **provisional / run still in progress** when `terminal` is false. A saved issue or verdict is useful immediately but must never be presented as the final run merely because it arrived first.

Use `wait` when only the terminal or attention state matters:

```bash
trycase verify wait <bot-id> --wait-timeout 1800 --pretty
```

`--wait-timeout` bounds the local polling command, not the verification agent's work. If it expires while the bot is healthy, run `status` and resume `follow` or `wait`; do not stop the bot or tell it to finish early.

## Inspect Results While Work Continues

Results and proof are available independently of final completion:

```bash
trycase result list <bot-id> --pretty
trycase result show <result-id> --pretty
trycase artifact list <bot-id> --pretty
trycase artifact url <artifact-id> --pretty
trycase artifact download <artifact-id> --output <path>
```

Report result wording, verdict, blocker or failure reason, proof status, and target commit. Use `artifact url` for a viewable link. Download only when local inspection is useful. An automated scenario may legitimately have no visual artifact; missing proof matters when the result says visual proof is required.

## Send Follow-ups

Continue the same bot instead of creating replacements for corrections or independent validation:

```bash
trycase verify message <bot-id> \
  --message-file <message-path> \
  --request-id <stable-message-id> \
  --pretty
```

Use the same request ID when retrying an uncertain message. A response with `queued: true` is accepted work, not an error. Continue following until that message and its resulting assistant response are complete.

When a result needs validation, ask the same primary agent to delegate a focused check to a native subagent where parallelism helps. Do not create a new TryCase bot merely to imitate subagent validation.

## Stop And Archive Safely

These commands require confirmation and change bot state:

```bash
trycase verify stop <bot-id> --yes --pretty
trycase verify archive <bot-id> --yes --pretty
```

Use them only when the user requests the lifecycle change or when cleaning up a disposable bot created for the current task. Resolve the exact bot first. Prefer `stop` for active work that should cease and `archive` after results have been collected. Do not stop unrelated or user-owned runs merely to free capacity.

`trycase logout` revokes the current CLI token and removes stored credentials. Run it only when the user wants to sign out or when an explicitly disposable authentication test requires it.

## Finish

Return:

- repository, PR, target commit, and bot ID;
- whether the bot is terminal or still working;
- surfaced issues and verdicts, distinguishing provisional from final;
- saved result IDs and viewable artifact links when available;
- any attention action the user must complete;
- any task-owned bot or disposable PR cleanup performed.

Do not claim completion while `terminal` is false or messages remain pending or queued.

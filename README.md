# TryCase Agent Skill

Agent guidance for the current TryCase CLI: launch and monitor end-to-end verification agents for connected GitHub pull requests, stream issues and verdicts while work continues, send follow-ups, and inspect saved results and proof.

The previous environment-oriented TryCase product has been retired. This repository no longer teaches `env`, `project`, `fs`, `terminal`, `desktop`, `computer`, worker, upload, machine-size, or cloud-computer workflows.

## Install

```bash
npx skills add bencsn/trycase-skills --skill trycase-cli -g
```

Update an existing installation:

```bash
npx skills update -g trycase-cli
```

Remove the obsolete skill if it was previously installed:

```bash
npx skills remove -g trycase-run-linux-app
```

## Use

```text
Use $trycase-cli to verify this pull request, surface supported issues and verdicts as they become available, validate continuously with parallel subagents where useful, and keep the overall in-progress versus finished status explicit.
```

The CLI is self-describing. Agents should run `trycase capabilities` before relying on remembered syntax.

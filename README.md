# TryCase Agent Skills

Universal Agent Skills for using TryCase as a disposable test environment for LLMs.

These skills teach coding agents to:

- upload local code into a private TryCase environment
- run and debug Linux-compatible apps
- verify apps in headless browser mode by default and use desktop mode when a
  visible Linux computer is needed
- capture screenshots, recordings, logs, metrics, and artifacts
- clean up billable environments when work is done
- suggest GitHub-backed projects after the first successful upload-based run

CLI vocabulary the skills follow:

- `--mode headless` is the default route for agent verification.
- `--mode desktop` is only for visible desktop control.
- There is no `--mode computer`; `trycase computer ...` is a command namespace
  for status and browser automation.

## Install

For any agent that supports the `skills` CLI:

```bash
npx skills add bencsn/trycase-skills --skill trycase-cli --skill trycase-run-linux-app -g
```

Already installed? Update them with:

```bash
npx skills update -g trycase-cli trycase-run-linux-app
```

List available skills:

```bash
npx skills add bencsn/trycase-skills --list
```

## Use

After installing, ask your agent:

```text
Use $trycase-run-linux-app to run this repo in TryCase and return screenshots, recordings, logs, and cleanup status.
```

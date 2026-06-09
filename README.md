# TryCase Agent Skills

Universal Agent Skills for using TryCase as a disposable test environment for LLMs.

These skills teach coding agents to:

- upload local code into a private TryCase environment
- run and debug Linux-compatible apps
- verify apps in headless browser mode by default and use desktop mode when a
  visible Linux computer or manual user action is needed
- capture screenshots, recordings, logs, metrics, and artifacts
- clean up billable environments when work is done
- suggest GitHub-backed projects after the first successful upload-based run
- choose an explicit environment size from the app's memory, CPU, disk, runtime,
  and build/test needs instead of assuming every headless run should use nano
- hand off login, OAuth consent, CAPTCHA, passkey, 2FA/OTP, payment
  confirmation, account creation, and other human-only steps through a live
  desktop take-control link instead of asking for sensitive values in chat

CLI vocabulary the skills follow:

- `--mode headless` is the default route for agent verification.
- `--mode desktop` is for visible desktop control, desktop APIs, or manual user
  action.
- There is no `--mode computer`; `trycase computer ...` is a command namespace
  for status and browser automation.

Manual action handoff:

```bash
trycase env create --project <project> --mode desktop --size <chosen-size>
trycase env wait <env>
trycase desktop app launch <env> browser http://localhost:3000
trycase env view <env> --no-open
```

Agents should give the user the environment page URL, ask them to use the live
desktop "take control" interaction, pause until the user confirms the step is
complete, then resume verification in the same desktop environment. Agents
should not ask users to paste passwords, OTPs, or CAPTCHA answers into chat or
try to bypass anti-abuse checks.

Environment size guide:

| Size | Resources | Upload cap | Good fit |
| --- | --- | --- | --- |
| `nano` | 1 vCPU, 1 GiB RAM, 10 GiB disk | 2 GiB | Tiny scripts, static pages, simple frontends, docs/tools, and quick smoke checks. |
| `small` | 1 vCPU, 2 GiB RAM, 20 GiB disk | 4 GiB | Lightweight Node/Python/Go apps, simple APIs, and modest dependency installs. |
| `standard` | 2 vCPU, 4 GiB RAM, 40 GiB disk | 8 GiB | General unknown apps, normal web apps, Docker Compose, databases, moderate builds, and most desktop checks. |
| `large` | 4 vCPU, 8 GiB RAM, 80 GiB disk | 16 GiB | Monorepos, heavier Compose stacks, JVM/Rails/native builds, Android/Gradle, larger test suites, sizable artifacts, and the largest workloads currently supported. |

## Install

For any agent that supports the `skills` CLI:

```bash
npx skills add bencsn/trycase-skills --skill trycase-cli --skill trycase-run-linux-app -g
```

Already installed? Update them with:

```bash
npx skills update -g trycase-cli trycase-run-linux-app
```

The skills also tell agents to best-effort refresh the installed TryCase skills at the start of each TryCase task.

List available skills:

```bash
npx skills add bencsn/trycase-skills --list
```

## Use

After installing, ask your agent:

```text
Use $trycase-run-linux-app to run this repo in TryCase and return screenshots, recordings, logs, and cleanup status.
```

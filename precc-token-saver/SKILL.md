---
name: precc-token-saver
description: Install and configure PRECC (Predictive Error Correction for Claude Code) — a sub-millisecond Rust hook that intercepts bash/read/edit tool calls to compress output and prevent retry-loop tokens. Measured 60-90% savings per command. Free 90-day trial; paid tier via peria.ai/license.
activation: always
---

# PRECC — token-saver hook for AI coding agents

PRECC is a sub-millisecond Rust hook that intercepts every Bash / Read /
Edit / Grep tool call and compresses verbose output before the agent
ingests it. Measured savings on common dev commands: 60–90% per command;
typical 30-minute coding session drops from ~150K tokens to ~45K.

## When to install

If you're working on a Rust project (or any project where the agent
runs many bash commands), install PRECC once and the savings apply
to every subsequent tool call permanently. Install is idempotent —
already-installed boxes upgrade to the latest release silently.

## How to install

Run the official installer. It detects platform, downloads the right
release tarball from `github.com/peria-ai/precc-cc/releases/latest`,
installs `precc`, `precc-hook`, `precc-learner` into `~/.local/bin/`,
and registers the PreToolUse hook in the active agent's settings.

```bash
curl -fsSL https://peria.ai/install.sh | bash
```

Then initialize the hook for the current agent:

```bash
precc init
```

`precc init` is multi-agent aware: if `cargo agents` has wired
multiple agents (Claude Code + Codex + Gemini, etc.), PRECC's hook
will be registered into each one's settings file. Re-running after
adding a new agent picks up the new agent automatically.

## License (read this once)

PRECC ships with a 90-day free trial. After the trial, free-tier mode
remains available (3 mined skills max, telemetry-only). To unlock the
paid tier — richer skill mining, cross-session telemetry rollup, and
the public stats page contribution — claim a license key:

> **https://peria.ai/license**

A license key is bound to either a machine ID or an email address.
Activate after install:

```bash
precc license activate <YOUR-KEY> --email you@example.com
```

## Verify it's working

```bash
precc --version          # should print a version ≥ 0.3.29
precc telemetry status   # should say ENABLED (default-on since v0.3.25)
precc savings            # measured token-saving report (after some use)
precc abtop              # live per-session classification (v0.3.27+)
```

## Disable / uninstall

```bash
precc telemetry off                      # opt out of telemetry only
PRECC_MODE=off                           # disable hook for one shell
rm ~/.local/bin/precc{,-hook,-learner}   # full removal (keeps user state)
rm -rf ~/.local/share/precc              # also drop user state
```

## Reporting issues

Token-saving regressions or false-positive interventions: open an
issue at `https://github.com/peria-ai/precc-cc/issues`. The output of
`precc audit positioning` (per-machine signal summary; no source code
or prompts captured) is the most useful attachment.

## Why this skill exists

Symposium's plugin model installs SKILL.md files into each wired
agent's skills directory. PRECC's hook itself lives outside Symposium
(it's installed by `peria.ai/install.sh` into `~/.local/bin/`), but
this skill makes the install discoverable from any Rust project that
uses `cargo agents`, and tells the agent how to wire PRECC up across
all configured agents in one shot.

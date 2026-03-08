---
name: safehub
description: Scan OpenClaw skills for malware and security issues before installation. Use when the user wants to verify a skill is safe, audit a ClawHub skill, or check a local or GitHub skill.
version: 1.0.0
author: sumeetghimire
metadata: {"openclaw":{"emoji":"🛡️","requires":{"bins":["node"]},"os":["darwin","linux","win32"]}}
---

# SafeHub

SafeHub is a security scanner for OpenClaw skills. It runs static analysis (Semgrep) and optional sandbox execution (Docker) on any skill—by name, local path, or GitHub URL—and returns a trust score and a clear recommendation: **safe to install**, **install with caution**, or **not safe**.

## Requirements

- **Node.js** (18+) — required to run the CLI.
- **Semgrep** — required for the scan command. Install with `brew install semgrep` or `npm install -g semgrep`.
- **Docker** — optional; used for sandbox execution. If Docker is not available, use `--no-sandbox` for static-only scanning.

## Commands

All commands are run via the `safehub` CLI (e.g. `safehub scan <target>` or `openclaw run safehub scan <target>`).

### scan

Scan a skill by ClawHub name, local path, or GitHub URL.

**Examples:**

```bash
safehub scan web-scraper
safehub scan ./my-local-skill
safehub scan https://github.com/user/their-skill
safehub scan https://github.com/BenedictKing/tavily-web --no-sandbox
```

**Options:**

- `--no-sandbox` — Skip Docker sandbox; run static analysis only (use when Docker is not installed).

### report

Show the last scan report for a skill without rescanning.

**Examples:**

```bash
safehub report web-scraper
safehub report risky-skill
```

### update

Pull the latest Semgrep scanner rules from the SafeHub GitHub repo (or your fork via `SAFEHUB_RULES_REPO`).

**Examples:**

```bash
safehub update
SAFEHUB_RULES_REPO=owner/repo safehub update
```

## Example output

After running `safehub scan <target>`, you’ll see:

- **Static analysis** — Findings from Semgrep (network, filesystem, eval/exec, env, obfuscation).
- **Sandbox behavior** — Whether the skill attempted network access or suspicious actions (when Docker is used).
- **Trust score** (0–100) and recommendation: **SAFE TO INSTALL**, **INSTALL WITH CAUTION**, or **NOT SAFE TO INSTALL**.

## Installation (users)

Once SafeHub is published on ClawHub:

```bash
openclaw install safehub
```

Or install the CLI globally from npm:

```bash
npm install -g safehub
```

Then run `safehub scan <target>` or use OpenClaw: `openclaw run safehub scan <target>`.

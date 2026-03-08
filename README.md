# SafeHub

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**SafeHub** is an open source OpenClaw skill that scans other OpenClaw skills for malware and security issues before you install them. It uses Semgrep for static analysis and Docker for sandboxed execution so you can see what a skill does and whether it’s safe to use.

---

## What SafeHub Does

- **Static analysis** — Runs Semgrep rules to find outbound network calls, filesystem writes, `eval`/`exec`, env access, and obfuscation.
- **Sandbox run** — Executes the skill in an isolated Docker container (no network, read-only filesystem except `/tmp`) and observes behavior.
- **Trust score** — Produces a 0–100 score and a clear recommendation: safe to install, install with caution, or not safe.

---

## Prerequisites

- **Node.js 18+**
- **Semgrep** (required for static analysis):  
  `npm install -g semgrep` or `brew install semgrep`
- **Docker** (optional): used for sandbox execution. If Docker is not available, use `--no-sandbox` to run static analysis only.

---

## Installation

Install SafeHub once with OpenClaw:

```bash
openclaw install safehub
```

---

## Commands

### Scan a skill

Scan a skill by **name** (from ClawHub), **local path**, or **GitHub URL**:

```bash
openclaw run safehub scan web-scraper
openclaw run safehub scan ./my-local-skill
openclaw run safehub scan https://github.com/someone/their-skill
```

To skip Docker and run only static analysis (e.g. if Docker is not installed):

```bash
openclaw run safehub scan ./my-local-skill --no-sandbox
```

**Example output:**

```
Scanning web-scraper v1.2.0...
─────────────────────────────
STATIC ANALYSIS:
✅ No outbound network calls detected
✅ No filesystem writes outside /tmp
⚠️  process.env access in index.js line 18
❌ eval() call in utils.js line 7

SANDBOX BEHAVIOR:
✅ No network connections attempted
✅ No suspicious syscalls
⚠️  Attempted to read /etc/passwd

TRUST SCORE: 42/100 ❌ NOT SAFE TO INSTALL
─────────────────────────────
RECOMMENDATION: Do not install this skill.
3 issues found. See full report above.
```

### Show last report

Show the last scan report for a skill without rescanning:

```bash
openclaw run safehub report web-scraper
```

### Update scanner rules

Download the latest Semgrep rules from the SafeHub repo (or your fork):

```bash
openclaw run safehub update
```

Optional: set `SAFEHUB_RULES_REPO=owner/repo` to use a different GitHub repo; set `SAFEHUB_RULES_BRANCH` (default `main`) for the branch name.

---

## Contributing

We welcome contributions, especially **new Semgrep rules** to improve detection.

- **[CONTRIBUTING.md](CONTRIBUTING.md)** — How to add and test new rules, and the pull request process.
- **[/rules](rules/)** — Semgrep rule files. Add or edit `.yml` files here and open a PR.

---

## Implementation status

| Phase | Description | Status |
|-------|-------------|--------|
| 1 | Folder structure, README, LICENSE, skill.json, package.json | Done |
| 2 | Static scanner (Semgrep) + 5 rule files (network, filesystem, obfuscation, execution, env) | Done |
| 3 | Docker sandbox (no network, read-only root, limits) | Done |
| 4 | Trust score calculator (0–100, SAFE / CAUTION / NOT SAFE) | Done |
| 5 | Commands wired: scan pipeline, report formatter, cached reports in `~/.safehub/reports` | Done |
| 6 | Update command: pull latest rules from GitHub (`SAFEHUB_RULES_REPO`) | Done |

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE).

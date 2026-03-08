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
- **Docker** (optional): used for sandbox execution. The sandbox runs skills in an isolated container; SafeHub needs Docker available on your machine if you do not use `--no-sandbox`. If Docker is not installed or not running, use `--no-sandbox` to run static analysis only.

---

## Installation

### Install SafeHub (users)

**Via OpenClaw (once published on ClawHub):**

```bash
openclaw install safehub
```

Then run scans with: `openclaw run safehub scan <skill>`.

**Via npm (global CLI):**

```bash
npm install -g safehub
```

Then run `safehub scan <target>`, `safehub report <name>`, or `safehub update` directly.

### Publish SafeHub to ClawHub (maintainers)

To publish or update SafeHub on the ClawHub registry:

1. Install the ClawHub CLI:
   ```bash
   npm install -g clawhub
   ```
2. Log in (browser or token):
   ```bash
   clawhub login
   ```
3. From the SafeHub repo root, publish:
   ```bash
   clawhub publish . --slug safehub --name "SafeHub" --version 1.0.0 --changelog "Initial release" --tags latest
   ```

Use `clawhub sync` to scan and publish multiple skills, or `clawhub publish ./path --slug safehub ...` for a single skill folder.

### Local development

From the project root:

```bash
npm install
npm link
```

After linking, `safehub` is available in your shell. For `openclaw run safehub` to work, OpenClaw must be able to run the `safehub` command (i.e. SafeHub must be installed globally or linked as above).

---

## Usage

**CLI (direct):**

```bash
safehub scan web-scraper
safehub scan ./my-local-skill
safehub scan https://github.com/someone/their-skill
safehub report web-scraper
safehub update
```

**OpenClaw:**

Once the `safehub` CLI is on your PATH, you can run it via OpenClaw:

```bash
openclaw run safehub scan web-scraper
openclaw run safehub scan ./my-local-skill
openclaw run safehub report web-scraper
openclaw run safehub update
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

## Publishing to ClawHub

To list SafeHub on the [ClawHub](https://clawhub.ai) marketplace so users can discover and install it:

1. **Create a ClawHub developer account**  
   Go to [clawhub.ai](https://clawhub.ai) → Sign up → choose **Developer Account** → complete profile (display name, bio, GitHub link) → verify your email. Your GitHub account should be at least one week old.

2. **Metadata**  
   The repo already includes **`clawhub.json`** with name, tagline, description, category (`utility`), tags, version, license, and `support_url` / `homepage` pointing to this repo. Adjust any fields if needed.

3. **Screenshots**  
   Add 3–5 screenshots to the **`screenshots/`** folder:
   - **Resolution:** 1920×1080 or 1280×720 PNG  
   - **Content:** e.g. terminal running `safehub scan`, example report output, `safehub report` or `safehub update`  
   - Use real runs (no placeholder text). ClawHub may reject low-quality or generic screenshots.

4. **Optional: demo video**  
   A 30–90 second video of SafeHub in use (e.g. scanning a skill and showing the report) can strengthen the listing. Host on YouTube or Vimeo and add the URL in the ClawHub submission form.

5. **Submit for review**  
   - Build the upload package (one folder with only essential files, no `node_modules` or `.git`):  
     `node scripts/prepare-clawhub.js`  
     This creates **`clawhub-package/`** and **`safehub-clawhub.zip`** in the repo root.  
   - In ClawHub dashboard → **Publish New Skill** → upload **`safehub-clawhub.zip`** (or zip the `clawhub-package` folder yourself).  
   - Fill the form (metadata is pre-filled from `clawhub.json`), upload screenshots, add demo URL if you have one  
   - In **Permission justification**, state why SafeHub needs **filesystem** (read skill source, write cached reports under `~/.safehub`) and **network** (clone GitHub URLs, optional rule updates)  
   - Submit for review. Review usually takes 2–5 business days.

6. **If you use the ClawHub CLI**  
   After logging in with `clawhub login` (GitHub auth), you can run `clawhub publish` from the skill root instead of uploading a tarball, if your ClawHub setup supports it.

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

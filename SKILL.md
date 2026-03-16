---
name: clawhub-cli
description: >
  Complete ClawHub CLI reference for searching, installing, updating, publishing,
  and managing agent skills from clawhub.ai. Use when working with the clawhub CLI:
  installing or updating skills, publishing new versions, syncing a skills directory,
  inspecting skill metadata, managing ownership transfers, or troubleshooting
  install/update/sync behavior. Triggers on clawhub, clawdhub, skill install,
  skill publish, skill sync, clawhub login.
metadata:
  openclaw:
    requires:
      bins:
        - clawhub
    install:
      - id: node
        kind: node
        package: clawhub
        bins:
          - clawhub
        label: Install ClawHub CLI (npm)
---

# ClawHub CLI

Full reference for the `clawhub` CLI (v0.7.0+). Source: [docs/cli.md](https://github.com/openclaw/clawhub/blob/main/docs/cli.md).

## Install

```bash
npm i -g clawhub
```

---

## Global Flags

| Flag | Description |
|------|-------------|
| `--workdir <dir>` | Working directory (default: cwd; falls back to OpenClaw workspace) |
| `--dir <dir>` | Skills install dir under workdir (default: `skills`) |
| `--site <url>` | Base URL for browser login (default: `https://clawhub.ai`) |
| `--registry <url>` | Registry API base URL (default: `https://clawhub.ai`) |
| `--no-input` | Disable all interactive prompts |

**Env equivalents:**
- `CLAWHUB_SITE` (legacy: `CLAWDHUB_SITE`)
- `CLAWHUB_REGISTRY` (legacy: `CLAWDHUB_REGISTRY`)
- `CLAWHUB_WORKDIR` (legacy: `CLAWDHUB_WORKDIR`)

---

## HTTP Proxy Support

For systems behind corporate proxies or restricted networks (Docker, Hetzner VPS, firewalls):

```bash
export HTTPS_PROXY=http://proxy.example.com:3128
export NO_PROXY=localhost,127.0.0.1
clawhub search "my query"
```

Supported env vars: `HTTPS_PROXY`, `https_proxy`, `HTTP_PROXY`, `http_proxy`, `NO_PROXY`, `no_proxy`

---

## Config File

Stores API token + cached registry URL.

- **macOS:** `~/Library/Application Support/clawhub/config.json`
- **Override:** `CLAWHUB_CONFIG_PATH` (legacy: `CLAWDHUB_CONFIG_PATH`)

---

## Commands

### Auth

```bash
# Browser login (opens clawhub.ai/cli/auth via loopback callback)
clawhub login

# Headless login (CI, servers)
clawhub login --token clh_...

# Verify stored token
clawhub whoami

# Remove stored token
clawhub logout
```

---

### Search & Discover

```bash
# Vector search skills
clawhub search "postgres backups"

# Browse latest skills
clawhub explore
clawhub explore --sort trending --limit 50
clawhub explore --sort downloads --json

# --sort options: newest | downloads | rating | installs | installsAllTime | trending
# Output: <slug>  v<version>  <age>  <summary> (summary truncated to 50 chars)
```

---

### Inspect (without installing)

```bash
# Fetch metadata for latest version
clawhub inspect my-skill

# Inspect a specific version or tag
clawhub inspect my-skill --version 1.2.0
clawhub inspect my-skill --tag latest

# List version history
clawhub inspect my-skill --versions
clawhub inspect my-skill --versions --limit 50

# List files for selected version
clawhub inspect my-skill --files

# Fetch raw file content (text files only; 200KB limit)
clawhub inspect my-skill --file SKILL.md
clawhub inspect my-skill --file scripts/setup.sh

# Machine-readable output
clawhub inspect my-skill --json
```

---

### Install & Uninstall

```bash
# Install latest version
clawhub install my-skill

# Install to specific directory (recommended for agent environments)
clawhub install my-skill --dir .agents/skills

# Install specific version
clawhub install my-skill --version 1.2.3

# Uninstall (prompts for confirmation)
clawhub uninstall my-skill

# Uninstall non-interactively
clawhub uninstall my-skill --no-input --yes
```

**Important: Use `--dir` parameter**
- Most agent environments only recognize skills installed in `.agents/skills/` directory
- Always specify `--dir .agents/skills` when installing skills for agent use
- Without `--dir`, skills are installed to `skills/` directory and may not be recognized
- Example: `clawhub install my-skill --dir .agents/skills`

**What install writes:**
- `<workdir>/.clawhub/lock.json` — lockfile (legacy: `.clawdhub`)
- `<skill>/.clawhub/origin.json` — origin metadata

---

### Update

```bash
# Update a specific skill
clawhub update my-skill

# Update to a specific version
clawhub update my-skill --version 1.2.3

# Update all installed skills
clawhub update --all

# Force overwrite even if local files are modified
clawhub update my-skill --force
clawhub update --all --no-input --force
```

**How fingerprinting works:**
- Computes fingerprint from local files
- If fingerprint matches a known version → no prompt
- If fingerprint does not match → refuses by default; use `--force` to overwrite

---

### List Installed Skills

```bash
# List from lockfile (current workdir)
clawhub list

# List from specific workdir
clawhub list --workdir /path/to/workspace
```

**Important: Path dependency**
- `clawhub list` only shows skills installed in the current workdir
- By default, workdir is the current directory (cwd)
- Lockfile location: `<workdir>/.clawhub/lock.json`
- To list skills from a different directory, use `--workdir` parameter
- Example: `clawhub list --workdir D:\agentSpace`

---

### Publish

```bash
# Publish a skill folder
clawhub publish ./my-skill --slug my-skill --version 1.2.0
clawhub publish ./my-skill --slug my-skill --version 1.2.0 --changelog "Fixes + new commands"

# Always use absolute path to avoid slug mismatch
clawhub publish /home/user/clawd/my-skill --slug my-skill --version 1.2.0
```

**Publishing notes:**
- Requires `--version` with a valid semver string
- Published skills are released under **MIT-0** — free to use, modify, redistribute without attribution
- Folder name = slug: the folder name is used as the slug, not the `name` field in SKILL.md
- After publishing, sync local copy: `clawhub update my-skill --force`
- Publish often times out but still succeeds — wait ~10 min then `clawhub inspect` to verify

---

### Sync (bulk publish)

```bash
# Scan and publish new/changed skills (interactive)
clawhub sync

# Specify extra scan roots
clawhub sync --root ./my-skills-dir --root ./other-dir

# Dry run — show plan only, no uploads
clawhub sync --dry-run

# Upload all without prompting
clawhub sync --all

# Control version bump type
clawhub sync --bump minor
clawhub sync --bump major --changelog "Breaking changes"

# Set tags on publish
clawhub sync --tags latest,stable

# Control concurrency
clawhub sync --concurrency 2

# Non-interactive (CI)
clawhub sync --all --no-input --changelog "Automated sync"
```

**Auto-detected roots** (when `~/.openclaw/openclaw.json` or `~/.clawdbot/clawdbot.json` present):
- `agent.workspace/skills`
- `routing.agents.*.workspace/skills`
- `~/.clawdbot/skills`
- `skills.load.extraDirs`

**Telemetry:** sent during `sync` when logged in. Disable with `CLAWHUB_DISABLE_TELEMETRY=1`.

---

### Star / Unstar

```bash
clawhub star my-skill
clawhub unstar my-skill --yes   # skip confirmation
```

---

### Delete / Restore (owner, moderator, admin)

```bash
# Soft-delete (hide) a skill
clawhub delete my-skill --yes
clawhub hide my-skill --yes      # alias for delete

# Restore a deleted/hidden skill
clawhub undelete my-skill --yes
clawhub unhide my-skill --yes    # alias for undelete
```

---

### Transfer Ownership

```bash
# Request transfer to another user
clawhub transfer request my-skill target-handle
clawhub transfer request my-skill target-handle --message "Handing off maintenance"

# List incoming/outgoing transfers
clawhub transfer list
clawhub transfer list --outgoing

# Accept / reject / cancel
clawhub transfer accept my-skill --yes
clawhub transfer reject my-skill --yes
clawhub transfer cancel my-skill --yes
```

---

### Admin Commands

```bash
# Ban a user (moderator/admin only)
clawhub ban-user their-handle --reason "spam"
clawhub ban-user --id user_abc123 --yes

# Change user role (admin only)
clawhub set-role their-handle moderator
clawhub set-role --id user_abc123 admin --fuzzy
```

---

## Lockfile Format

- Location: `<workdir>/.clawhub/lock.json`
- Legacy location: `<workdir>/.clawdhub`
- Per-skill origin: `<skill>/.clawhub/origin.json`

---

## Known Gotchas

- **Publish timeouts** are often false negatives — the publish succeeds server-side even when the CLI shows `Timeout`. Wait ~10 min and verify with `clawhub inspect <slug>`.
- **Slug = folder name**: `clawhub publish` uses the folder name, not the `name` field in SKILL.md. Rename the folder before publishing if needed.
- **Version source of truth**: always check `git tag --sort=-v:refname | head -1` for the correct next version — don't rely on `clawhub list` or `_meta.json`.
- **Rate limiting**: multiple API calls in short succession will trigger rate limit errors. Space out publishes.
- **After publishing**: run `clawhub update <slug> --force` to sync your local installed copy to the newly published version.

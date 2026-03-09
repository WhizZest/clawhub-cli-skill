# clawhub-cli-skill

A complete OpenClaw skill for the [ClawHub CLI](https://clawhub.ai), covering all commands documented in [`docs/cli.md`](https://github.com/openclaw/clawhub/blob/main/docs/cli.md).

## Why this exists

The bundled `clawhub` skill shipped with OpenClaw is outdated — it's missing `explore`, `inspect` flags, `transfer`, `star/unstar`, `sync` flags, proxy support, and more. This skill fills that gap based on the authoritative CLI docs.

## Installation

Copy `SKILL.md` into your OpenClaw skills directory:

```bash
cp SKILL.md ~/.openclaw/agents/main/skills/clawhub-cli/SKILL.md
```

Or install directly into your clawd workspace:

```bash
cp SKILL.md ~/clawd/skills/clawhub-cli/SKILL.md
```

## Coverage

- Auth: `login` (browser + headless), `whoami`, `logout`
- Search: `search`, `explore` (with all `--sort` options)
- Inspect: all `--versions`, `--files`, `--file`, `--tag`, `--json` flags
- Install / Uninstall / Update / List
- Publish + post-publish sync workflow
- Sync: all flags (`--root`, `--dry-run`, `--all`, `--bump`, `--changelog`, `--tags`, `--concurrency`)
- Star / Unstar
- Delete / Undelete / Hide / Unhide
- Transfer ownership (all subcommands)
- Admin: `ban-user`, `set-role`
- Global flags, HTTP proxy support, config file location
- Known gotchas (publish timeouts, slug = folder name, version source of truth)

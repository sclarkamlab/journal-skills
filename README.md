# journal-skills

A Claude Code plugin that automates a daily/weekly/monthly/quarterly work journal. It pulls activity from GitHub and Jira, writes structured notes to your Obsidian vault, and (optionally) commits them to a git-backed knowledge base.

## What it gives you

| Command | What it does |
|---|---|
| `/journal-init` | One-time setup. Collects your name, GitHub username, Jira site, vault path, etc. |
| `/standup` | Pulls yesterday's GitHub + Jira activity, generates a Slack-ready standup, saves to `Journal/{YYYY}/{Mon}/{DD}/standup.md`. |
| `/wrap` | End-of-day reflection. Compares the morning's standup to actual activity, saves `wrap.md`. |
| `/weekly-summary` | Aggregates the week's standups + wraps into `Journal/{YYYY}/{Mon}/{DD}-{DD}/summary.md`. |
| `/monthly-summary` | Aggregates weekly summaries into `Journal/{YYYY}/{Mon}/summary.md`. |
| `/quarterly-summary` | Aggregates monthly summaries into `Journal/{YYYY}/Q{X}/summary.md`. |

## Install

This plugin is distributed as a Claude Code marketplace. From within Claude Code:

```
/plugin marketplace add sclarkamlab/journal-skills
/plugin install journal-skills@journal-skills
```

Then run `/journal-init` to set up your config.

## Requirements

The commands shell out to these tools — install whatever you don't already have:

- [`gh`](https://cli.github.com) — GitHub CLI, authenticated
- [`acli`](https://developer.atlassian.com/cloud/acli/) — Atlassian CLI, authenticated against your Jira site
- An Obsidian CLI for reading/writing vault files. The commands assume an `obsidian` binary supporting `obsidian read path=...` and `obsidian create path=... content=...`.
- Optional: `git` if you want journal entries committed and pushed automatically

## Config

`/journal-init` writes `~/.config/journal-skills/config.json`:

```json
{
  "user_name": "Jane Doe",
  "github_username": "janedoe",
  "jira_site": "company.atlassian.net",
  "jira_projects": ["PROJ", "OTHER"],
  "vault_path": "/Users/jane/work-docs",
  "journal_subpath": "Journal",
  "git_enabled": true
}
```

Edit the file directly if you want to change values without re-running setup. Re-run `/journal-init` if your environment changes (new vault, new Jira tenant, etc.).

## Vault layout

Once you've been running for a few weeks the layout looks like this:

```
{vault_path}/
└── {journal_subpath}/
    └── 2026/
        ├── Jan/
        │   ├── 05/
        │   │   ├── standup.md
        │   │   └── wrap.md
        │   ├── 05-09/
        │   │   └── summary.md      # weekly
        │   └── summary.md          # monthly
        └── Q1/
            └── summary.md          # quarterly
```

## Customising

Each command is a single markdown file under `commands/`. Fork the plugin, edit the prompt, install your fork. Sensible places to customise:

- The standup output format (the `:white_tick:` / `:dart:` / `:this-is-fine-fire:` emoji style is one team's convention — change to match your Slack)
- The Jira JQL queries if your team uses different status names than `In Progress` / `In Review` / `In Testing`
- The wrap-up section headings if your team prefers a different reflection structure

## License

MIT

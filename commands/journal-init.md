---
description: One-time setup for the journal-skills plugin. Collects user info (name, GitHub username, Jira site, vault path) and writes them to ~/.config/journal-skills/config.json.
---

You are running the one-time setup for the `journal-skills` plugin. Your job is to collect a small set of values from the user and write a config file that the other journal commands (`/standup`, `/wrap`, `/weekly-summary`, `/monthly-summary`, `/quarterly-summary`) read on every run.

## Step 1: Check for existing config

```bash
CONFIG_DIR="$HOME/.config/journal-skills"
CONFIG_FILE="$CONFIG_DIR/config.json"
test -f "$CONFIG_FILE" && cat "$CONFIG_FILE"
```

If the file already exists, show its current contents and ask whether to overwrite or edit specific fields.

## Step 2: Collect values via AskUserQuestion

Ask the user for each of the following (use the `AskUserQuestion` tool, one question per value with a sensible default option plus "Other" for free input):

| Key | Description | Example |
|---|---|---|
| `user_name` | The user's display name | `Jane Doe` |
| `github_username` | GitHub login (used by `gh` CLI queries) | `janedoe` |
| `jira_site` | Atlassian site domain | `company.atlassian.net` |
| `jira_projects` | Comma-separated Jira project keys the user works in | `AM,GMA` |
| `vault_path` | Absolute path to the Obsidian vault root | `/Users/jane/work-docs` |
| `journal_subpath` | Folder inside the vault where journal entries go (default `Journal`) | `Journal` |
| `git_enabled` | Whether to commit + push journal entries to git after each save (`true`/`false`) | `true` |

You may collect multiple values in a single `AskUserQuestion` call when it makes sense, but don't try to guess any of them — every value must be confirmed by the user.

## Step 3: Write config

```bash
mkdir -p "$CONFIG_DIR"
cat > "$CONFIG_FILE" << EOF
{
  "user_name": "...",
  "github_username": "...",
  "jira_site": "...",
  "jira_projects": ["...", "..."],
  "vault_path": "...",
  "journal_subpath": "Journal",
  "git_enabled": true
}
EOF
```

## Step 4: Verify dependencies

Check whether the tools the journal commands rely on are available, and report which are missing:

```bash
command -v gh && gh --version | head -1
command -v acli && acli --version 2>/dev/null
command -v obsidian
test -d "<vault_path>/.git" && echo "vault is a git repo" || echo "vault is NOT a git repo"
```

For anything missing, point the user at install instructions:
- `gh`: https://cli.github.com
- `acli` (Atlassian CLI): https://developer.atlassian.com/cloud/acli/
- `obsidian` CLI: https://github.com/Yakitrak/obsidian-cli (or whichever the team uses — see plugin README)

If `git_enabled` is `true` but the vault isn't a git repo, tell the user to `git init` and add a remote, or set `git_enabled` to `false`.

## Step 5: Confirm

Print the final config and tell the user they can now run `/standup`, `/wrap`, `/weekly-summary`, `/monthly-summary`, or `/quarterly-summary`.

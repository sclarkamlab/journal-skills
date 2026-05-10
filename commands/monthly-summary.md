---
description: Aggregate the month's weekly summaries into a monthly overview focused on major accomplishments, recurring patterns, and growth.
---

You are a monthly summary generator. You aggregate weekly summaries into a monthly overview focused on accomplishments, patterns, and growth.

## Step 0: Load config

```bash
CONFIG_FILE="$HOME/.config/journal-skills/config.json"
if [ ! -f "$CONFIG_FILE" ]; then
  echo "Config not found. Run /journal-init first."
  exit 1
fi
jq -e . "$CONFIG_FILE" > /dev/null || { echo "Config is not valid JSON"; exit 1; }

VAULT=$(jq -r .vault_path "$CONFIG_FILE")
JOURNAL=$(jq -r .journal_subpath "$CONFIG_FILE")
GIT_ENABLED=$(jq -r .git_enabled "$CONFIG_FILE")
echo "VAULT=$VAULT JOURNAL=$JOURNAL GIT_ENABLED=$GIT_ENABLED"
```

Use these shell variables in every command below.

## Vault paths

- Weekly inputs: `$JOURNAL/{YYYY}/{Mon}/{DD}-{DD}/summary.md`
- Monthly output: `$JOURNAL/{YYYY}/{Mon}/summary.md` (e.g. `Journal/2026/Apr/summary.md`)

## Step 1: Determine the month

Use `YYYY-MM` format. If we're in the first few days of a new month, default to summarizing the PREVIOUS month. Ask if ambiguous.

## Step 2: Read all weekly summaries for the month

Use the `obsidian` CLI (NOT MCP tools):

```bash
obsidian search query="Week" path="$JOURNAL/{YYYY}/{Mon}" format=json
```

Read each match. Also check the previous month's folder for weeks that span the boundary (e.g. `Journal/2026/Mar/31-Apr04/summary.md`).

## Step 3: Generate the summary

```markdown
# {Month YYYY} Summary

## Major Accomplishments
{The big things that got done. Features shipped, bugs fixed, projects completed. Group by project/initiative.}

## Ongoing Work
{Projects and initiatives still in progress at month end.}

## Key Metrics (if available)
{PRs merged, tickets closed, any quantifiable output from the weekly files.}

## Recurring Friction
{Patterns that showed up across multiple weeks. This is the most valuable section — it surfaces systemic issues.}

## Learnings & Growth
{New skills, technologies explored, processes improved. What did you get better at?}

## Next Month Focus
{Carry-forward from the last week + any bigger-picture goals.}
```

## Step 4: Save and (optionally) commit

```bash
SUM_FILE=$(mktemp)
cat > "$SUM_FILE" << 'EOF'
# April 2026 Summary
...
EOF
obsidian create path="$JOURNAL/{YYYY}/{Mon}/summary" content="$(cat "$SUM_FILE")"
rm "$SUM_FILE"
```

If `git_enabled` is true:

```bash
cd "$VAULT"
git add -A
git commit -m "Add monthly summary"
git push
```

Skip commit/push if nothing to commit. Show the user the key highlights.

## Guidelines

- The monthly level is where patterns become visible — prioritize surfacing recurring themes
- Quantify where possible (X PRs, Y tickets, Z features)
- This document should be useful for performance reviews and 1:1s
- Keep it to 1-2 pages max

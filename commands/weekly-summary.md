---
description: Aggregate the week's daily standups and wraps into a weekly summary highlighting themes, accomplishments, and recurring issues.
---

You are a weekly summary generator. You aggregate the user's daily standups and wrap-ups into a concise weekly overview that highlights themes, accomplishments, and patterns.

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

- Standups: `$JOURNAL/{YYYY}/{Mon}/{DD}/standup.md`
- Wraps: `$JOURNAL/{YYYY}/{Mon}/{DD}/wrap.md`
- Weekly output: `$JOURNAL/{YYYY}/{Mon}/{DD}-{DD}/summary.md` (Mon-Fri range, e.g. `Journal/2026/Apr/07-11/summary.md`). If the week spans two months, use the month the week starts in and show both dates (e.g. `Journal/2026/Mar/31-Apr04/summary.md`).

## Step 1: Determine the week

Calculate the ISO week number and the date range (Mon-Fri). Use format `YYYY-WXX` for the heading.

If today is Monday, default to summarizing LAST week. Otherwise summarize the current week so far. Ask the user if ambiguous.

## Step 2: Read all daily files for the week

Use the `obsidian` CLI (NOT MCP tools):

- `obsidian read path="$JOURNAL/{YYYY}/{Mon}/{DD}/standup.md"` for each weekday
- `obsidian read path="$JOURNAL/{YYYY}/{Mon}/{DD}/wrap.md"` for each weekday

Non-zero exit = missing file; note which days have data and which don't.

## Step 3: Generate the summary

```markdown
# Week {YYYY-WXX} ({Mon date} - {Fri date})

## Accomplishments
{Bullet list of everything completed this week, synthesized from standups and wraps. Group by theme/project where possible.}

## In Progress
{Work that's ongoing — carried forward items, things started but not finished.}

## Key Decisions & Discussions
{Any notable decisions, architectural choices, or discussions from wrap-ups or user additions.}

## Friction & Blockers
{Recurring issues from daily wraps. Call out patterns — e.g., "context switching was a theme on 3/5 days".}

## Themes
{2-3 sentence summary of what the week was about. What was the main focus? How productive did it feel?}

## Next Week
{Carry-forward items from Friday's wrap or the last recorded day.}
```

## Step 4: Save and (optionally) commit

```bash
SUM_FILE=$(mktemp)
cat > "$SUM_FILE" << 'EOF'
# Week 2026-W15 ...
EOF
obsidian create path="$JOURNAL/{YYYY}/{Mon}/{DD}-{DD}/summary" content="$(cat "$SUM_FILE")"
rm "$SUM_FILE"
```

If `git_enabled` is true:

```bash
cd "$VAULT"
git add -A
git commit -m "Add weekly summary"
git push
```

Skip commit/push if nothing to commit. Show the user a brief summary of the week.

## Guidelines

- Synthesize, don't just concatenate — the value is in patterns and themes
- If friction items repeat across days, call that out explicitly
- Keep it scannable — someone reading this in a month should get the gist in 30 seconds
- Reference Jira tickets where relevant (just keys like `PROJ-1234`)

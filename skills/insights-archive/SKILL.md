---
name: insights-archive
description: Archive Claude Code /insights reports for historical tracking. Supports archiving, listing, comparing, and opening reports.
license: MIT
metadata:
  author: jonathanprozzi
  version: "2.0.0"
argument-hint: "[archive|list|open|diff] [args...]"
allowed-tools: "Bash, Read, Glob"
---

# Insights Archive

Archive and manage Claude Code `/insights` reports to track how your usage patterns evolve over time.

## Usage

```
/insights-archive archive      # Save current report with timestamp
/insights-archive list         # See all archived reports
/insights-archive open         # Open current report in browser
/insights-archive open <date>  # Open archived report by date
/insights-archive diff <d1> <d2>  # Compare two archived reports
/insights-archive              # Show help
```

## Configuration

```
INSIGHTS_DIR=~/.claude/usage-data
ARCHIVE_DIR=~/.claude/usage-data/archives
CURRENT_REPORT=~/.claude/usage-data/report.html
```

## Implementation

This skill is fully self-contained. Do NOT download or execute any external scripts. All logic is implemented directly below using the allowed tools.

### Input Validation

Before processing any command, validate all user-provided arguments:

- **Subcommand** must be one of: `archive`, `list`, `open`, `diff`, or empty (show help)
- **Date arguments** (for `open` and `diff`) must match the pattern `YYYY-MM-DD` or `YYYY-MM-DD-HHMM` (digits and hyphens only). Reject any argument containing characters other than `[0-9-]` or the literal string `current`.
- If validation fails, show the help text and stop.

### Commands

#### No arguments / help

If no arguments are provided or the subcommand is not recognized, display:

```
Usage: /insights-archive <command> [args]

Commands:
  archive          Archive current /insights report with timestamp
  list             List all archived reports
  open [date]      Open current report or archived report by date
  diff <d1> <d2>   Open two reports for comparison

Examples:
  /insights-archive archive
  /insights-archive list
  /insights-archive open current
  /insights-archive open 2026-02-04
  /insights-archive diff 2026-02-01 2026-02-04
```

#### archive

1. Ensure the archive directory exists:
   ```bash
   mkdir -p ~/.claude/usage-data/archives
   ```

2. Check that the current report exists:
   ```bash
   test -f ~/.claude/usage-data/report.html && echo "exists" || echo "missing"
   ```
   If missing, tell the user: "No current report found. Run /insights first to generate a report."

3. Generate the archive filename with a timestamp:
   ```bash
   date +%Y-%m-%d-%H%M
   ```

4. Check for duplicate and copy:
   ```bash
   ARCHIVE_FILE=~/.claude/usage-data/archives/report-<TIMESTAMP>.html && test -f "$ARCHIVE_FILE" && echo "duplicate" || cp ~/.claude/usage-data/report.html "$ARCHIVE_FILE" && echo "Archived to: $ARCHIVE_FILE"
   ```

5. Report the result to the user.

#### list

1. List archived reports using Glob for `~/.claude/usage-data/archives/report-*.html`

2. If no files found, tell the user: "No archived reports found. Run '/insights-archive archive' to create one."

3. Otherwise, display the list with dates extracted from filenames (strip `report-` prefix and `.html` suffix), and remind: "Use '/insights-archive open <date>' to view one."

#### open

1. Validate the date argument (must be `current`, empty, or match `[0-9-]+`).

2. If argument is empty or `current`:
   ```bash
   test -f ~/.claude/usage-data/report.html && open ~/.claude/usage-data/report.html || echo "missing"
   ```
   If missing, tell user to run /insights first.

3. If a date is provided, use Glob to find matching files: `~/.claude/usage-data/archives/report-<DATE>*.html`
   - If exactly one match, open it:
     ```bash
     open <matched-file>
     ```
   - If no matches, tell the user and suggest running `list`.
   - If multiple matches, show them and ask the user to be more specific.

#### diff

1. Require exactly two date arguments. If not provided, show usage: "Usage: /insights-archive diff <date1> <date2>"

2. Validate both date arguments (must match `[0-9-]+`).

3. Find matching archives for each date using Glob: `~/.claude/usage-data/archives/report-<DATE>*.html`

4. If either date has no match, report which one is missing.

5. If both found, open them:
   ```bash
   open <file1> && open <file2>
   ```
   Tell the user: "Opening both reports for comparison."

---

## Why Archive?

Tracking how your Claude Code usage patterns evolve over time is valuable:

- **See workflow improvements** — Are git operations decreasing as you add custom skills?
- **Track friction resolution** — Did the suggestions from last month's report get addressed?
- **Measure learning** — How are your interaction patterns maturing?

This aligns with the "knowledge compounds" thesis — your usage data is another form of knowledge that benefits from historical context.

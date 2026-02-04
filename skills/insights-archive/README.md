# insights-archive

Archive Claude Code `/insights` reports for historical tracking.

## Installation

```bash
npx skills add jonathanprozzi/agent-skills --skill insights-archive
```

## Prerequisites

This skill requires the `insights-archive` shell script. Install via:

**Option A: jonathanprozzi/dotfiles**
```bash
# If using the dotfiles repo, the script is at bin/insights-archive
# Ensure ~/Documents/projects/web-projects/dotfiles/bin is in your PATH
```

**Option B: Standalone install**
```bash
mkdir -p ~/.local/bin
curl -o ~/.local/bin/insights-archive \
  https://raw.githubusercontent.com/jonathanprozzi/dotfiles/main/bin/insights-archive
chmod +x ~/.local/bin/insights-archive
```

## Usage

```bash
# Archive current report
/insights-archive archive

# List all archives
/insights-archive list

# Open current or archived report
/insights-archive open
/insights-archive open 2026-02-04

# Compare two reports
/insights-archive diff 2026-02-01 2026-02-04
```

## Why?

Claude Code's `/insights` command generates a usage report analyzing your sessions. By archiving these reports over time, you can:

- Track how your usage patterns evolve
- See which friction points get resolved
- Measure workflow improvements

## File Locations

- **Current report:** `~/.claude/usage-data/report.html`
- **Archives:** `~/.claude/usage-data/archives/report-YYYY-MM-DD-HHMM.html`

## License

MIT

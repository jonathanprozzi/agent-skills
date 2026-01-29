# jonathanprozzi/agent-skills

Agent skills monorepo for Claude Code and compatible AI agents.

## Structure

```
agent-skills/
├── skills/              # Individual skill directories
│   └── pre-pr-scan/     # Pre-PR compliance scan
│       ├── SKILL.md     # Skill definition
│       └── README.md    # Skill documentation
├── README.md            # Repo overview
├── CLAUDE.md            # This file
└── LICENSE              # MIT
```

## Adding a New Skill

1. Create directory: `skills/my-skill/`
2. Add `SKILL.md` with required frontmatter:
   ```yaml
   ---
   name: my-skill
   description: Brief description of what the skill does
   ---
   ```
3. Add `README.md` with installation and usage docs
4. Update root `README.md` with new skill in the table

## Skill Frontmatter Options

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Skill identifier (used in `/name` command) |
| `description` | Yes | Brief description |
| `context` | No | `fork` for isolated execution |
| `agent` | No | Agent type (`general-purpose`, `Explore`) |
| `argument-hint` | No | Usage hint shown to users |
| `allowed-tools` | No | Tool restrictions |

## Testing Skills Locally

```bash
# From any project directory, test a skill before publishing
claude --skill /path/to/agent-skills/skills/pre-pr-scan
```

## Publishing Flow

1. Develop locally in this repo
2. Push to GitHub
3. Skills auto-appear on [skills.sh](https://skills.sh) when users install via `npx skills add`

## Skill Spec Reference

- [Agent Skills Specification](https://agentskills.io/specification)
- [skills.sh Leaderboard](https://skills.sh)
- [Claude Code Skills Docs](https://docs.anthropic.com/en/docs/claude-code/skills)

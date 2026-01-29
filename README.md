# jonathanprozzi/agent-skills

Agent skills for Claude Code and compatible AI agents.

## Available Skills

| Skill | Description | Install |
|-------|-------------|---------|
| [pre-pr-scan](./skills/pre-pr-scan/) | Pre-PR compliance and security scan | `npx skills add jonathanprozzi/agent-skills --skill pre-pr-scan` |

## Installation

```bash
# Install all skills
npx skills add jonathanprozzi/agent-skills

# Install specific skill
npx skills add jonathanprozzi/agent-skills --skill pre-pr-scan

# Install globally (user-level)
npx skills add jonathanprozzi/agent-skills -g
```

## Skill Format

Skills follow the [Agent Skills specification](https://agentskills.io/specification).

Each skill is a directory containing:
- `SKILL.md` - The skill definition (YAML frontmatter + markdown instructions)
- `README.md` - Human-readable documentation

## Contributing

1. Create a new directory: `skills/my-skill/`
2. Add `SKILL.md` with required frontmatter (`name`, `description`)
3. Add `README.md` with usage documentation
4. Submit a PR

## License

MIT

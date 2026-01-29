# pre-pr-scan

Pre-PR compliance and security scan that catches issues before creating a pull request.

## What it does

Runs a comprehensive scan of your changes against:
- **CI failure detection** - Compilation errors, duplicate definitions, test/implementation mismatches
- **CLAUDE.md compliance** - Checks changes against project-specific guidelines
- **Security issues** - OWASP Top 10, Web3/blockchain vulnerabilities
- **Logic bugs** - Null access, race conditions, missing error handling
- **Commit hygiene** - Flip-flopping, incomplete changes, contradictions

## Why use it

Save API credits by catching issues locally before the GitHub PR review bot runs.

**Complements the GitHub bot** - The official Anthropic code-review plugin explicitly excludes general security issues (by design). This skill catches:
- Security vulnerabilities (OWASP, Web3) that the bot skips
- Commit history issues the bot doesn't analyze
- Everything the bot catches (CI failures, CLAUDE.md compliance)

**Cost efficient** - Run locally on your Max plan (subsidized) instead of org API credits.

## Installation

```bash
# Install from GitHub
npx skills add jonathanprozzi/agent-skills --skill pre-pr-scan

# Or install all skills from this repo
npx skills add jonathanprozzi/agent-skills

# Or install globally (user-level)
npx skills add jonathanprozzi/agent-skills --skill pre-pr-scan -g
```

### Manual installation

Copy the `SKILL.md` to your project:

```bash
mkdir -p .claude/skills/pre-pr-scan
cp SKILL.md .claude/skills/pre-pr-scan/
```

Anyone who clones the repo will have the skill available.

## Usage

```bash
# Scan current branch vs main
/pre-pr-scan

# Scan against a different base branch
/pre-pr-scan develop

# With flags
/pre-pr-scan --validate          # Add HIGH issue validation pass
/pre-pr-scan --all               # Include speculative issues (60-79% confidence)
/pre-pr-scan --quick             # Fast mode (Haiku for all agents)
/pre-pr-scan develop --validate  # Combine base branch + flag

# Explicit CLAUDE.md paths (monorepos)
/pre-pr-scan --guidelines ./CLAUDE.md ./packages/api/CLAUDE.md
```

## Flags

Control the cost/precision trade-off:

| Flag | Effect | Token Impact |
|------|--------|--------------|
| (default) | ≥80% confidence, no validation | ~170-190k |
| `--validate` | Spawn validators for each HIGH issue | +100-120k |
| `--all` | Include 60-79% confidence issues | ~same |
| `--quick` | Use Haiku for all agents | ~80-100k |
| `--guidelines <paths...>` | Use explicit CLAUDE.md file paths | ~same |

**When to use each:**
- **Default** - Most scans, good balance of coverage and cost
- **`--validate`** - Pre-merge on critical branches, worth the extra tokens for 80% precision
- **`--all`** - Exploratory, see what the scan noticed but wasn't confident about
- **`--quick`** - Fast feedback during development, not for final scan
- **`--guidelines`** - Monorepos with multiple CLAUDE.md files, or non-standard locations

## How it works

### Adaptive mode selection

| PR Size | Mode | Agents |
|---------|------|--------|
| < 1000 lines AND < 15 files | Sequential | 1 general-purpose |
| ≥ 1000 lines OR ≥ 15 files | Parallel | 4 specialized |

### Parallel agents (large PRs)

Each agent has **exclusive responsibility** (no overlap) for token efficiency:

| Agent | Type | Focus |
|-------|------|-------|
| CI + CLAUDE.md | general-purpose | Compilation errors, test coherence, guideline violations |
| Security | general-purpose | OWASP + Web3 vulnerabilities |
| Logic Bugs | general-purpose | Runtime errors, null access, error handling |
| History | Explore | Commit patterns (Haiku, git commands only) |

**Why general-purpose?** Agents 1-3 need file read access for deep-dives. The Explore type has restricted permissions when running in parallel.

**Token efficiency**: Agents analyze the diff first, only reading full files (max 5 each) when a potential issue is spotted.

### HIGH issue validation (opt-in)

With `--validate` flag, HIGH severity issues go through a second validation pass:
- Each HIGH issue spawns a validator agent
- Validator confirms or rejects with full file context
- Only CONFIRMED issues are reported

This multi-pass approach (inspired by Anthropic's code-review plugin) reduces false positives on critical issues. Adds ~100-120k tokens but achieves ~80% precision on HIGH issues.

### No CLAUDE.md?

Works without project guidelines - falls back to security, bugs, and TypeScript checks. Portable to any repo.

## Output format

```markdown
## Pre-PR Scan: feature-branch vs main

**Mode:** Parallel (4 agents)
**Flags:** --validate
**Files scanned:** 23 changed files
**Commits analyzed:** 5 commits
**Issues found:** 4 (1 High, 2 Medium, 1 Low)
**Validated:** 3/4 HIGH confirmed

---

### HIGH: Missing error state handling
**Confidence:** 95%
**File:** `src/components/UserProfile.tsx:42`
**Category:** CLAUDE.md Compliance
**Guideline:** CLAUDE.md lines 326-339 - "ALWAYS handle loading/error states"
**Issue:** useQuery result used without checking isLoading or error states
**Fix:** Add loading/error state handling before accessing data

---
...
```

## Configuration

The skill uses these frontmatter settings:

```yaml
context: fork          # Runs in isolated subagent
agent: general-purpose # Has Task tool for parallel agents
allowed-tools: Read, Grep, Glob, Bash(git *), Task
```

## Confidence threshold

**Default**: Reports issues with ≥80% confidence. Filters out:
- Speculative issues (60-79%)
- Context-dependent patterns that might be intentional
- Issues linters would catch

**With `--all`**: Includes 60-79% issues marked as "SPECULATIVE" for awareness.

## Related

- [Agent Skills Standard](https://agentskills.io)
- [Claude Code Skills Docs](https://docs.anthropic.com/en/docs/claude-code/skills)

## License

MIT

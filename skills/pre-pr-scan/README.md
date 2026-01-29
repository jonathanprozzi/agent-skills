# pre-pr-scan

Pre-PR compliance and security scan that catches issues before creating a pull request.

## What it does

Runs a comprehensive scan of your changes against:
- **CI failure detection** - Compilation errors, duplicate definitions, test/implementation mismatches
- **CLAUDE.md compliance** - Checks changes against project-specific guidelines
- **Security issues** - OWASP Top 10, Web3/blockchain vulnerabilities
- **Logic bugs** - Null access, race conditions, missing error handling
- **Commit hygiene** - Flip-flopping, incomplete changes, contradictions

## Workflow

Run `/pre-pr-scan` **before** creating your PR:

```
1. Finish your changes
2. Run /pre-pr-scan locally (subsidized via Max plan)
3. Fix any issues found
4. Create PR
5. GitHub bot sees clean code (saves org API credits)
```

**Without the skill:**
```
PR created → Bot finds 5 issues → Fix → Push → Bot re-scans →
Finds 2 more → Fix → Push → Bot re-scans → Clean
= 3 full scans × API cost
```

**With the skill:**
```
Run locally (subsidized) → Fix 5 issues → PR created →
Bot finds 0-1 issues → Done
= 1 full scan × API cost
```

Each prevented iteration saves a full PR re-scan.

## Why use it

**Shift-left approach** - Catch issues earlier in the dev cycle when they're cheaper to fix.

**Complements the GitHub bot** - The official Anthropic code-review plugin explicitly excludes general security issues (by design). From the source:

> "Do NOT flag: General code quality concerns (e.g., lack of test coverage, **general security issues**) unless explicitly required in CLAUDE.md"

This skill catches what the bot skips:
- Security vulnerabilities (OWASP, Web3) - **explicitly excluded** by bot
- Commit history issues - bot doesn't analyze
- Everything the bot catches (CI failures, CLAUDE.md compliance)

**Cost efficient** - Run locally on your Max plan (subsidized) instead of org API credits.

### Cost savings model

| Context | Token Cost |
|---------|------------|
| GitHub Action | Real $ per token (org pays) |
| Claude Code Max | Subsidized (~$100-200/mo subscription) |

**Example (real org data):**
- GitHub Action API cost: ~$300/month
- Peak days: $70-80/day during active PR periods

**If devs run `/pre-pr-scan` locally first:**
- Catch issues BEFORE creating PR (subsidized)
- Fewer issues for GitHub bot to find
- Fewer PR iterations (each push re-triggers the bot)
- Potential savings: 20-50% of monthly API costs

The skill pays for itself if it prevents even a few PR iterations per month.

### Comparison: This skill vs GitHub bot

| Aspect | pre-pr-scan | GitHub Bot |
|--------|-------------|------------|
| **When** | Before PR (shift-left) | After PR created |
| **Cost** | Subsidized (Max plan) | Org API credits |
| **Security** | Always scans | Excluded unless in CLAUDE.md |
| **History analysis** | Yes | No |
| **CLAUDE.md** | Yes | Yes (2 redundant agents) |
| **CI failures** | Yes | Yes (primary focus) |

**Bottom line:** Run both. This skill catches issues earlier and covers security the bot explicitly skips.

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

### Codex installation

The skill is compatible with OpenAI Codex (same Agent Skills spec):

```bash
~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo jonathanprozzi/agent-skills \
  --path skills/pre-pr-scan
```

**Note**: Claude Code extensions (`context: fork`, `agent: general-purpose`) are ignored by Codex. The core scan logic works, but invocation differs.

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
/pre-pr-scan --run-checks        # Run lint/test/build like CI
/pre-pr-scan develop --validate  # Combine base branch + flag

# Explicit CLAUDE.md paths (monorepos)
/pre-pr-scan --guidelines ./CLAUDE.md ./packages/api/CLAUDE.md
```

## Flags

Control the cost/precision trade-off:

| Flag | Effect | Token Impact |
|------|--------|--------------|
| (default) | Static analysis, ≥80% confidence, no validation | ~170-190k |
| `--validate` | Spawn validators for each HIGH issue | +100-120k |
| `--all` | Include 60-79% confidence issues | ~same |
| `--quick` | Use Haiku for all agents | ~80-100k |
| `--run-checks` | Run lint/test/build commands (like CI) | +varies |
| `--guidelines <paths...>` | Use explicit CLAUDE.md file paths | ~same |

**When to use each:**
- **Default** - Most scans, good balance of coverage and cost (static analysis only)
- **`--validate`** - Pre-merge on critical branches, worth the extra tokens for 80% precision
- **`--all`** - Exploratory, see what the scan noticed but wasn't confident about
- **`--quick`** - Fast feedback during development, not for final scan
- **`--run-checks`** - Match what CI will find (runs lint/test/build commands)
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

## Validation

Tested against real PRs and controlled scenarios:

| Test | Size | Result |
|------|------|--------|
| PR 497 | Small | Matched GitHub bot (3 issues) |
| PR 495 | Small | Matched + found 1 additional |
| PR 493 | Small | Correctly found no issues |
| PR 440 | 12k lines, 43 files | 19 issues (parallel mode) |
| Controlled test | 16 files, 15 planted issues | Found all + bonus discoveries |

**HIGH issue validation accuracy:** 80% precision (4/5 confirmed in PR 440 test)

**What it catches that GitHub bot misses:**

| Category | This Skill | GitHub Bot |
|----------|------------|------------|
| Security (OWASP) | Unsafe JSON parsing, injection | Excluded by design |
| Security (Web3) | Unchecksummed addresses | Excluded by design |
| Logic bugs | Missing deps, null access | Some coverage |
| Commit hygiene | Flip-flopping, incomplete changes | Not analyzed |

## Related

- [Agent Skills Standard](https://agentskills.io)
- [Claude Code Skills Docs](https://docs.anthropic.com/en/docs/claude-code/skills)

## License

MIT

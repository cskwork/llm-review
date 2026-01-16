# What Does Claude Codex Do?

Claude Codex is a **multi-AI code review pipeline** for Claude Code. Instead of trusting a single AI's output, your code goes through two independent reviewers before being finalized.

## The Problem It Solves

When you ask Claude to write code, it generates a solution and you trust it works. But what if there's a security vulnerability? A logic bug? An over-engineered abstraction?

**Claude Codex adds a second opinion.** Actually, two.

## How It Works

```
Your Request
     |
     v
[Planning Phase]
  Claude creates a plan
     |
     v
[Review 1: Sonnet]
  Fast scan for obvious issues
     |
     v
[Review 2: Codex]
  Fresh perspective from OpenAI's Codex
     |
     v
[Implementation]
  Code gets written
     |
     v
[Review 1: Sonnet]
  Code quality + security + tests
     |
     v
[Review 2: Codex]
  Final approval
     |
     v
Done (or loop back if issues found)
```

## What Each Reviewer Checks

| Reviewer | Focus Areas |
|----------|-------------|
| **Sonnet** | Obvious bugs, code style, security basics, test coverage |
| **Codex** | Different AI perspective, catches what Sonnet missed |

Both check against:
- OWASP Top 10 vulnerabilities
- Error handling
- Input validation
- Over-engineering detection

## The Loop-Until-Pass Model

Code doesn't proceed until **both** reviewers approve. If Codex finds an issue after Sonnet approved, the cycle restarts from Sonnet with the fix applied.

## Quick Start

```bash
# Install the plugin
/plugin marketplace add cskwork/claude-codex
/plugin install claude-codex@claude-codex --scope user

# Use it
/claude-codex:multi-ai Add user authentication with JWT
```

## Why Two AIs?

Different AI models have different blind spots. Sonnet might miss something Codex catches. The sequential review process means:

1. **More bugs caught** - Two reviewers > one reviewer
2. **Better security** - Independent security checks
3. **Less over-engineering** - Each reviewer flags unnecessary complexity
4. **Production-ready code** - Nothing ships until both approve

## Files Created During Pipeline

The plugin creates a `.task/` directory in your project:

| File | What It Contains |
|------|------------------|
| `user-request.txt` | Your original request |
| `plan.json` | Initial plan |
| `plan-refined.json` | Detailed technical plan |
| `impl-result.json` | Implementation summary |
| `review-sonnet.json` | Sonnet's review |
| `review-codex.json` | Codex's review |
| `state.json` | Current pipeline state |

Add `.task` to your `.gitignore`.

## Configuration (pipeline.config.json)

The plugin behavior is controlled by `pipeline.config.json`. You can override settings by creating `pipeline.config.local.json` in your project.

### autonomy

| Setting | Default | Meaning |
|---------|---------|---------|
| `mode` | `semi-autonomous` | Pipeline runs automatically but pauses at key points |
| `approvalPoints.planning` | `false` | No human approval needed for plans |
| `approvalPoints.implementation` | `false` | No human approval needed for code |
| `approvalPoints.review` | `false` | No human approval needed for reviews |
| `approvalPoints.commit` | `true` | **Requires human approval before committing** |
| `maxAutoRetries` | `3` | Retry failed operations before stopping |
| `planReviewLoopLimit` | `5` | Max plan review iterations |
| `codeReviewLoopLimit` | `7` | Max code review iterations |

### models

| Role | Provider | Model | Temperature | Purpose |
|------|----------|-------|-------------|---------|
| `orchestrator` | Claude | Opus | 0.7 | Planning, coordination |
| `coder` | Claude | Opus | 0.3 | Code implementation |
| `reviewer` | OpenAI | Codex | 0.2 | Final review |

### errorHandling

| Setting | Default | Meaning |
|---------|---------|---------|
| `autoResolveAttempts` | `3` | Try to auto-fix errors before pausing |
| `pauseOnUnresolvable` | `true` | Stop pipeline if error can't be fixed |
| `notifyOnError` | `true` | Alert user on errors |
| `errorLogRetention` | `30d` | Keep error logs for 30 days |

### commit

| Setting | Default | Meaning |
|---------|---------|---------|
| `strategy` | `per-task` | One commit per completed task |
| `messageFormat` | `conventional` | Use conventional commits (feat:, fix:, etc.) |
| `signOff` | `true` | Add Signed-off-by line |
| `branch.createFeatureBranch` | `true` | Create new branch for each task |
| `branch.namePattern` | `feature/{task-id}-{short-title}` | Branch naming template |

### timeouts

| Setting | Default | Meaning |
|---------|---------|---------|
| `implementation` | `600` | 10 min max for implementing code |
| `review` | `300` | 5 min max for each review |
| `autoResolve` | `180` | 3 min max for auto-fixing errors |

### debate (disabled)

| Setting | Default | Meaning |
|---------|---------|---------|
| `enabled` | `false` | Reviewers don't debate each other |
| `maxRounds` | `0` | No debate rounds |

### Local Overrides

Create `pipeline.config.local.json` in your project to override defaults:

```json
{
  "autonomy": {
    "codeReviewLoopLimit": 10,
    "approvalPoints": {
      "planning": true
    }
  }
}
```

## Requirements

- [Claude Code](https://claude.ai/code) with MAX subscription
- [Codex CLI](https://github.com/openai/codex) with Plus subscription
- [Bun](https://bun.sh/) runtime

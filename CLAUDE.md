# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Codex is a Claude Code plugin marketplace providing multi-AI orchestration for code planning, implementation, and review. The main plugin (`claude-codex`) implements a sequential review workflow: sonnet -> codex.

## Repository Structure

```
claude-codex/
├── .claude-plugin/marketplace.json   # Marketplace catalog (lists all plugins)
├── plugins/
│   └── claude-codex/                 # Main plugin
│       ├── .claude-plugin/plugin.json
│       ├── skills/                   # Pipeline skills (multi-ai, implement-sonnet, review-*)
│       ├── scripts/                  # Shell scripts (orchestrator, state-manager, recover)
│       ├── docs/                     # Standards, workflow documentation
│       └── pipeline.config.json      # Pipeline configuration
└── README.md
```

## Key Commands

```bash
# Validate plugin structure
/plugin validate .

# Test marketplace locally
/plugin marketplace add .

# Install plugin locally for testing
/plugin install claude-codex@claude-codex --scope user
```

## Architecture

### Pipeline Flow

```
idle -> plan_drafting -> plan_refining -> implementing -> complete
                              |                |
                              v                v
                    Sequential reviews: sonnet -> codex
```

### Skills (`plugins/claude-codex/skills/`)

| Skill | Purpose | Output File |
|-------|---------|-------------|
| `multi-ai` | Entry point, runs full workflow | - |
| `implement-sonnet` | Code implementation | `.task/impl-result.json` |
| `review-sonnet` | Fast review | `.task/review-sonnet.json` |
| `review-codex` | Final review via Codex CLI | `.task/review-codex.json` |

### Scripts (`plugins/claude-codex/scripts/`)

- `orchestrator.sh` - Shows current state, guides next action
- `state-manager.sh` - State transitions (get/set)
- `recover.sh` - Error recovery
- `json-tool.ts` - Cross-platform JSON processing (replaces jq)

### Path Variables in Skills

- `${CLAUDE_PLUGIN_ROOT}` - Plugin installation directory
- `${CLAUDE_PROJECT_DIR}` - User's project directory (where `.task/` is created)

## Task State Files

All pipeline state lives in `.task/` within the target project:

| File | Purpose |
|------|---------|
| `state.json` | Current pipeline state |
| `user-request.txt` | Original user request |
| `plan.json` | Initial plan |
| `plan-refined.json` | Refined plan with technical details |
| `impl-result.json` | Implementation result |
| `review-*.json` | Review outputs per reviewer |

## Review Decision Rules

From `docs/standards.md`:
- Any `error` severity -> `needs_changes`
- 3+ `warning` severity -> `needs_changes`
- Only `suggestion` severity -> `approved`

## Adding a New Plugin

1. Create `plugins/your-plugin/`
2. Add `.claude-plugin/plugin.json` manifest
3. Add skills in `skills/` directory
4. Update `.claude-plugin/marketplace.json` to list the new plugin

## Cross-Platform Notes

- All JSON processing uses `bun ./scripts/json-tool.ts` instead of `jq`
- Shell scripts work on Windows (Git Bash), macOS, and Linux

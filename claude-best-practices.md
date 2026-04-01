# Claude Code Best Practices — Complete Reference Guide

> A comprehensive guide to setting up `.claude/`, writing CLAUDE.md, creating skills & subagents, configuring hooks, MCP servers, and structuring your project for optimal Claude Code usage.

---

## Table of Contents

1. [Project Structure Overview](#1-project-structure-overview)
2. [CLAUDE.md — Project Instructions](#2-claudemd--project-instructions)
3. [Settings Configuration](#3-settings-configuration)
4. [Custom Skills (Slash Commands)](#4-custom-skills-slash-commands)
5. [Subagents](#5-subagents)
6. [Hooks](#6-hooks)
7. [MCP Servers](#7-mcp-servers)
8. [Rules (Modular Instructions)](#8-rules-modular-instructions)
9. [Memory System](#9-memory-system)
10. [Setup Checklist](#10-setup-checklist)

---

## 1. Project Structure Overview

```
your-project/
├── CLAUDE.md                        # Project instructions (alternative location)
├── .claude/
│   ├── CLAUDE.md                    # Project instructions
│   ├── settings.json                # Project settings (commit to git)
│   ├── settings.local.json          # Local overrides (gitignored)
│   ├── .mcp.json                    # MCP server configuration
│   ├── skills/                      # Custom slash commands
│   │   ├── deploy/
│   │   │   └── SKILL.md
│   │   └── review-pr/
│   │       └── SKILL.md
│   ├── agents/                      # Custom subagents
│   │   ├── code-reviewer/
│   │   │   └── code-reviewer.md
│   │   └── debugger/
│   │       └── debugger.md
│   ├── rules/                       # Modular instruction files
│   │   ├── code-style.md
│   │   ├── testing.md
│   │   └── security.md
│   ├── hooks/                       # Hook scripts
│   │   └── protect-files.sh
│   └── agent-memory/                # Subagent persistent memory
│       └── code-reviewer/
│           └── MEMORY.md
├── src/
├── tests/
└── .gitignore
```

### User-Level Configuration (~/.claude/)

```
~/.claude/
├── CLAUDE.md                        # Personal instructions (all projects)
├── settings.json                    # Global user settings
├── skills/                          # Reusable skills across projects
├── agents/                          # Reusable subagents across projects
├── rules/                           # Personal rules for all projects
└── projects/
    └── <project-name>/
        └── memory/                  # Auto memory (machine-local)
            ├── MEMORY.md
            └── topic-notes.md
```

---

## 2. CLAUDE.md — Project Instructions

CLAUDE.md is loaded at the start of every session. It provides persistent instructions that guide Claude's behavior.

### Placement Hierarchy

| Location | Scope | Shared with team? |
|---|---|---|
| `~/.claude/CLAUDE.md` | All your projects | No |
| `./CLAUDE.md` or `./.claude/CLAUDE.md` | Project root | Yes (via git) |
| `./src/CLAUDE.md` | Subdirectory | On-demand when working there |

### What to Include

- Build, test, and deploy commands
- Code style rules that differ from defaults
- Repository conventions (branch naming, PR process)
- Architecture decisions
- Common gotchas

### What NOT to Include

- Standard conventions Claude already knows
- Information derivable from reading code
- Long tutorials or explanations
- Frequently changing data

### Example CLAUDE.md

```markdown
# MyApp

## Build & Test
- Install: `npm install`
- Build: `npm run build`
- Test: `npm test`
- Lint: `npm run lint`

## Code Style
- TypeScript for all new code
- 2-space indentation, semicolons
- Use `import`/`export`, not `require`

## Git Workflow
- Branch from `main`: `feature/description` or `fix/issue-123`
- Run tests before committing
- Create PR for review before merging

## Architecture
- API handlers: `src/api/handlers/`
- Business logic: `src/logic/`
- Tests colocated: `src/module.test.ts`
```

### Importing External Files

```markdown
See @README.md for setup.
Git workflow: @docs/git-instructions.md
Personal prefs: @~/.claude/my-project-instructions.md
```

### Best Practices

- **Keep under 200 lines** — longer files reduce adherence
- **Be specific** — "Use 2-space indentation" not "format properly"
- **Split when large** — use `.claude/rules/` for modular instructions
- **Review regularly** — remove contradictions and outdated info

---

## 3. Settings Configuration

### Scopes (highest to lowest priority)

1. **CLI flags** (`--model`, `--permissions`)
2. **Local** — `.claude/settings.local.json` (gitignored)
3. **Project** — `.claude/settings.json` (committed)
4. **User** — `~/.claude/settings.json`
5. **Managed** — system-level (IT-deployed)

### Example .claude/settings.json

```json
{
  "permissions": {
    "allow": [
      "Bash(npm *)",
      "Bash(git *)",
      "Edit(src/**/*.ts)"
    ],
    "deny": [
      "Bash(rm -rf)",
      "Write(.env)",
      "Write(package-lock.json)"
    ]
  },
  "env": {
    "NODE_ENV": "development"
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

### Example ~/.claude/settings.json (User)

```json
{
  "model": "opus",
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(npm *)"
    ]
  },
  "env": {
    "GITHUB_TOKEN": "$GITHUB_TOKEN"
  }
}
```

---

## 4. Custom Skills (Slash Commands)

Skills are markdown files in `.claude/skills/` that create custom `/commands`.

### Directory Structure

```
.claude/skills/
└── my-skill/
    ├── SKILL.md              # Required: skill definition
    ├── template.md           # Optional: output template
    └── examples/
        └── sample.md         # Optional: example outputs
```

### SKILL.md Format

```markdown
---
name: fix-issue
description: Fix a GitHub issue by number
argument-hint: [issue-number]
user-invocable: true
allowed-tools: Read, Grep, Glob, Bash, Edit
---

# Fix GitHub Issue

1. Fetch issue details: `gh issue view $ARGUMENTS`
2. Analyze the problem
3. Find relevant code
4. Implement the fix
5. Write tests
6. Create a commit
```

### Frontmatter Fields

| Field | Description |
|---|---|
| `name` | Slash command name (lowercase, hyphens) |
| `description` | What it does (Claude uses this for auto-invocation) |
| `argument-hint` | Hint shown in autocomplete |
| `user-invocable` | `true`/`false` — show in `/` menu |
| `disable-model-invocation` | `true` — only manual invocation |
| `allowed-tools` | Comma-separated tool list |
| `model` | `sonnet`, `opus`, `haiku`, or `inherit` |
| `context` | `fork` — run in isolated subagent |
| `agent` | Subagent type when `context: fork` |
| `paths` | Glob patterns for auto-loading |

### Using Arguments

```markdown
---
name: migrate-component
description: Migrate a component between frameworks
---

Migrate **$0** from **$1** to **$2**.
```

Invoke: `/migrate-component SearchBar React Vue`

- `$ARGUMENTS` — all arguments as string
- `$0`, `$1`, `$2` — positional arguments
- `${CLAUDE_SKILL_DIR}` — skill directory path

### Dynamic Context

```markdown
---
name: pr-summary
context: fork
agent: Explore
---

## PR Data
- Diff: !`gh pr diff`
- Comments: !`gh pr view --comments`

Summarize this PR.
```

Shell commands in `` !`command` `` run before Claude sees the skill.

### Running in Subagent

```markdown
---
name: deep-research
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly using Glob and Grep.
```

---

## 5. Subagents

Subagents are specialized AI assistants that run in isolated contexts.

### Built-in Subagents

| Name | Tools | Purpose |
|---|---|---|
| **Explore** | Read-only | Search and analyze codebases |
| **Plan** | Read-only | Plan before implementation |
| **general-purpose** | All tools | Complex multi-step tasks |

### Creating a Subagent

Create `.claude/agents/code-reviewer/code-reviewer.md`:

```markdown
---
name: code-reviewer
description: Expert code review specialist
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior code reviewer. When invoked:

1. Run `git diff` to see recent changes
2. Review for:
   - Code clarity and readability
   - Security vulnerabilities
   - Performance issues
   - Test coverage
3. Provide feedback organized by severity.
```

### Frontmatter Fields

| Field | Description |
|---|---|
| `name` | Unique identifier (lowercase, hyphens) |
| `description` | When Claude should delegate to this agent |
| `tools` | Allowed tools (allowlist) |
| `disallowedTools` | Tools to deny (denylist) |
| `model` | `sonnet`, `opus`, `haiku`, or `inherit` |
| `permissionMode` | `default`, `acceptEdits`, `dontAsk`, `bypassPermissions` |
| `maxTurns` | Max agentic turns |
| `memory` | `user`, `project`, or `local` |
| `background` | `true` — always run as background task |
| `isolation` | `worktree` — run in isolated git worktree |

### Invoking Subagents

```
# Natural language (Claude decides)
Use the code-reviewer to analyze my recent changes

# @-mention (guaranteed invocation)
@"code-reviewer (agent)" look at the auth changes

# Session-wide (main agent)
claude --agent code-reviewer
```

### Example: Debugger Agent

```markdown
---
name: debugger
description: Debug errors and test failures
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger. When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate the failure
4. Implement minimal fix
5. Verify solution
```

### Example: Read-Only Researcher

```markdown
---
name: researcher
description: Research codebase without modifications
tools: Read, Grep, Glob
model: haiku
---

You are a codebase researcher. Find information
and report findings. Never modify files.
```

### Persistent Memory

```yaml
---
name: code-reviewer
memory: project
---
```

Memory scopes:
- `user` → `~/.claude/agent-memory/<name>/`
- `project` → `.claude/agent-memory/<name>/`
- `local` → `.claude/agent-memory-local/<name>/`

---

## 6. Hooks

Hooks are shell commands that execute at specific lifecycle points. They provide **deterministic control** — unlike CLAUDE.md, hooks always execute.

### Hook Events

| Event | When | Can Block? |
|---|---|---|
| `SessionStart` | Session begins/resumes | No |
| `UserPromptSubmit` | Before processing prompt | No |
| `PreToolUse` | Before tool executes | Yes (exit 2) |
| `PostToolUse` | After tool succeeds | No |
| `PostToolUseFailure` | After tool fails | No |
| `Notification` | Claude needs attention | No |
| `Stop` | Claude finishes responding | No |
| `SubagentStart` | Subagent starts | No |
| `SubagentStop` | Subagent finishes | No |

### Configuration

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "./.claude/hooks/protect-files.sh"
          }
        ]
      }
    ]
  }
}
```

### Exit Codes

- **0** — Action proceeds. Stdout injected as context.
- **2** — Action blocked. Stderr shown as feedback.
- **Other** — Action proceeds. Stderr logged.

### Example: Protect Files

`.claude/hooks/protect-files.sh`:

```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

PROTECTED=(".env" "package-lock.json")
for pattern in "${PROTECTED[@]}"; do
  if [[ "$FILE" == *"$pattern"* ]]; then
    echo "Blocked: $FILE is protected" >&2
    exit 2
  fi
done
exit 0
```

### Example: Auto-Format After Edits

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

### Hook Types

| Type | Description |
|---|---|
| `command` | Shell command |
| `prompt` | Single LLM call |
| `agent` | Full subagent execution |
| `http` | POST to HTTP endpoint |

---

## 7. MCP Servers

MCP (Model Context Protocol) connects Claude to external tools and services.

### Configuration (.claude/.mcp.json)

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "$DATABASE_URL"
      }
    },
    "slack": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "$SLACK_BOT_TOKEN"
      }
    }
  }
}
```

### Server Types

| Type | Connection | Use Case |
|---|---|---|
| `stdio` | Local process | CLI tools, npm packages |
| `http` | HTTP POST | Remote servers |
| `sse` | Server-Sent Events | Real-time streams |
| `ws` | WebSocket | Persistent connections |

### Scope

- **Project** — `.claude/.mcp.json` (this project only)
- **User** — `~/.claude/.mcp.json` (all projects)

---

## 8. Rules (Modular Instructions)

Rules split CLAUDE.md into topic-specific files.

### Structure

```
.claude/rules/
├── code-style.md
├── testing.md
├── security.md
└── api/
    └── endpoint-design.md
```

### Path-Specific Rules

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Rules
- All endpoints must include input validation
- Use standard error format from `src/api/errors.ts`
- Include OpenAPI documentation comments
```

Rules with `paths` only load when Claude works with matching files.

---

## 9. Memory System

### Auto Memory

Claude saves learnings across sessions at:

```
~/.claude/projects/<project>/memory/
├── MEMORY.md            # Index (first 200 lines loaded at startup)
├── user_role.md         # User information
├── feedback_testing.md  # User feedback
└── project_goals.md     # Project context
```

### Memory Types

| Type | Purpose |
|---|---|
| `user` | User's role, preferences, knowledge |
| `feedback` | Corrections and confirmed approaches |
| `project` | Ongoing work, goals, decisions |
| `reference` | Pointers to external resources |

### Enable/Disable

```json
{
  "autoMemoryEnabled": false
}
```

---

## 10. Setup Checklist

- [ ] Write `CLAUDE.md` with build commands, style, conventions
- [ ] Configure `.claude/settings.json` with permissions and hooks
- [ ] Create skills in `.claude/skills/` for reusable workflows
- [ ] Create subagents in `.claude/agents/` for specialized tasks
- [ ] Set up hooks for automation (lint, format, protect files)
- [ ] Connect MCP servers in `.claude/.mcp.json`
- [ ] Add `.claude/rules/` for modular, path-specific instructions
- [ ] **Commit to git**: CLAUDE.md, settings.json, skills/, agents/, rules/
- [ ] **Gitignore**: settings.local.json, agent-memory-local/

### .gitignore additions

```
.claude/settings.local.json
.claude/agent-memory-local/
```

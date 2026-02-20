# External Subagents Skill Design

## Overview

A universal agentskills.io skill that teaches coding agents how to invoke other coding agent CLIs (Claude Code, Codex CLI, Gemini CLI) via shell to perform tasks like code review, brainstorming, and research from a different model's perspective.

## Key Constraints

- Agents only invoke external subagents when **explicitly instructed by the user**
- Subagents run in **sandboxed mode with full permissions**
- Output captured as **plain text via stdout**
- Agent **detects available CLIs** and suggests the best option if user didn't specify
- **No model flag** - let each CLI use its default model
- When relevant skills/context exist, pass **full file paths** in the prompt and tell the subagent to read them
- Run subagent commands **in background** if the agent's tooling supports it
- If the parent agent is sandboxed, the subagent invocation must happen **outside the sandbox**

## Format

Single `SKILL.md` following the agentskills.io specification. Works across all compatible agents (Claude Code, Codex CLI, Gemini CLI, Cursor, etc.).

## Directory Structure

```
external-subagents/
├── SKILL.md
└── references/
    ├── claude-code.md
    ├── codex-cli.md
    └── gemini-cli.md
```

## Core Flow

1. **Trigger** - User explicitly asks to delegate a task to another agent
2. **Detect** - Run `which claude codex gemini` to see what's available
3. **Suggest** - If user didn't specify a tool, recommend one from what's available
4. **Sandbox check** - Detect if currently running in a sandbox; if so, execute the subagent command outside the sandbox (e.g., `dangerouslyDisableSandbox` in Claude Code)
5. **Build command** - Construct the non-interactive CLI invocation with:
   - The user's task as the prompt
   - Sandboxed mode with full permissions for the subagent
   - Plain text output
   - Any relevant skill/context file paths mentioned in the prompt with instructions to read them
6. **Execute** - Run the command in the background if the agent's tooling supports it; otherwise run in foreground
7. **Report** - Present the subagent's output to the user

## Command Templates

### Claude Code

```bash
claude -p "Your task here. Read /path/to/SKILL.md first." \
  --dangerously-skip-permissions
```

### Codex CLI

```bash
codex exec --full-auto "Your task here. Read /path/to/SKILL.md first."
```

### Gemini CLI

```bash
gemini -p "Your task here. Read /path/to/SKILL.md first." \
  --approval-mode yolo --sandbox
```

## Sandbox Escape

When the parent agent is sandboxed, the subagent CLI won't be accessible inside the sandbox. The skill teaches agents to:

- **Detect sandbox** - Check if the current shell is sandboxed (inside Docker, macOS Seatbelt, restricted filesystem)
- **Escape for invocation** - Use the agent's mechanism to run commands outside the sandbox:
  - Claude Code: `dangerouslyDisableSandbox: true` on the Bash tool
  - Codex CLI: Run subagent command on the host, not inside the workspace sandbox
  - Gemini CLI: Invoke outside the Seatbelt/Docker sandbox

The subagent itself still runs sandboxed with full permissions. Only the *invocation* escapes the parent's sandbox.

## Context & Skill Passing

When invoking a subagent, enrich the prompt with relevant context:

- **Skill files** - Tell the subagent to read the skill file at a given absolute path in the prompt text (not via system prompt flags)
- **Working directory** - Subagent runs in the same project directory as the parent agent
- **Relevant files** - Mention specific files explicitly in the prompt

Example prompt construction:

> "Review the code in src/auth.py for security issues. Before starting, read the skill file at /home/user/.claude/skills/code-review/SKILL.md and follow its instructions."

## Reference Files

Each `references/*.md` file contains a focused quick-reference (~100-150 lines) for that CLI tool:

- Non-interactive invocation syntax
- Key flags (model, permissions, output format, working directory, turn/budget limits)
- Stdin piping pattern
- Environment variables for authentication
- How to pass additional context files

These are loaded only when the agent needs to invoke that specific tool.

## Decisions Summary

| Aspect | Decision |
|--------|----------|
| Format | Single agentskills.io `SKILL.md` |
| Trigger | User explicitly requests external subagent |
| Detection | `which claude codex gemini`, suggest if user didn't specify |
| Permissions | Subagent runs sandboxed with full permissions |
| Sandbox escape | Parent invokes subagent outside its own sandbox |
| Output | Plain text via stdout |
| Model | No model flag, use CLI defaults |
| Background | Run in background if agent supports it |
| Context passing | Include file paths in prompt, tell subagent to read them |
| Skill passing | Tell subagent to read skill file at given path in the prompt |
| Reference files | 3 separate files per CLI tool |

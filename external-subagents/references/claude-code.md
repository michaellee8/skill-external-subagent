# Claude Code CLI Reference

Quick reference for invoking Claude Code in non-interactive (headless) mode.

## Non-interactive Invocation

```bash
claude -p "Your prompt here."
```

The `-p` flag runs Claude Code in non-interactive mode: it processes the prompt, prints the result to stdout, and exits. No interactive session is started.

## Key Flags

| Flag | Description |
|---|---|
| `--settings '<json>'` | Pass inline settings JSON. Use `'{"sandbox":{"enabled":true,"autoAllowBashIfSandboxed":true}}'` to run sandboxed with auto-approved bash commands. **Recommended for subagent use.** |
| `--model <name>` | Override the model. Accepts short names (`sonnet`, `opus`, `haiku`) or full model IDs (`claude-sonnet-4-20250514`). |
| `--output-format <fmt>` | Output format: `text` (default), `json`, or `stream-json`. Use `json` when you need to parse the response programmatically. |
| `--max-turns <n>` | Limit the number of agentic turns (tool-use rounds). Prevents runaway loops. |
| `--max-budget-usd <n>` | Cap total spend in USD for this invocation. The process stops when the budget is exhausted. |
| `--tools <list>` | Restrict which tools the subagent can use. Comma-separated list, e.g. `"Read,Grep,Glob"` for read-only analysis. |
| `--allowedTools <list>` | Whitelist of tools to make available. Comma or space-separated. Supports patterns (e.g., `Bash(git:*)`). |
| `--disallowedTools <list>` | Deny specific tools. Comma or space-separated. Tools in this list are blocked entirely. |
| `--add-dir <path>` | Add an additional working directory the subagent can access. Can be specified multiple times. |
| `--append-system-prompt "text"` | Append text to the system prompt. Use this to inject instructions or constraints. |
| `--append-system-prompt-file <path>` | Append the contents of a file to the system prompt. Useful for injecting skill files or guidelines. |
| `--no-session-persistence` | Do not save the session to disk. Recommended for subagent invocations to avoid session clutter. |

## Stdin Piping

Pipe file contents or other data into the prompt via stdin:

```bash
cat src/auth.py | claude -p "Review this file for security issues."
```

```bash
git diff HEAD~1 | claude -p "Summarize the changes in this diff."
```

Stdin content is prepended to the prompt as context. This is useful for feeding large files or command output without exceeding shell argument limits.

## Environment Variables

| Variable | Description |
|---|---|
| `ANTHROPIC_API_KEY` | API key for authentication. Required if not already configured via `claude login`. |
| `ANTHROPIC_MODEL` | Default model override. Same effect as `--model` but applies to all invocations. The `--model` flag takes precedence. |

## Sandbox Escape Note

When the parent agent is running inside a sandbox (e.g. Claude Code's own sandbox), the `claude` binary may not be accessible or executable from within the sandbox. To invoke `claude -p` from a sandboxed parent agent:

**Set `dangerouslyDisableSandbox: true` on the Bash tool call** that runs the `claude -p` command. This allows the invocation to run on the host. The child Claude Code process manages its own sandbox independently via the `--settings` sandbox configuration.

```json
{
  "command": "claude --settings '{\"sandbox\":{\"enabled\":true,\"autoAllowBashIfSandboxed\":true}}' -p \"Review src/auth.py for security issues.\"",
  "dangerouslyDisableSandbox": true
}
```

Only the invocation escapes the parent sandbox. The subagent process runs inside its own sandbox with auto-approved bash commands.

## Example Invocations

### Basic code review

```bash
claude --settings '{"sandbox":{"enabled":true,"autoAllowBashIfSandboxed":true}}' \
  -p "Review the code in src/handlers/ for error handling issues. \
List each issue with file path and line number."
```

### Read-only analysis with tool restrictions

```bash
claude --settings '{"sandbox":{"enabled":true,"autoAllowBashIfSandboxed":true}}' \
  -p "Analyze the architecture of this project. \
Identify the main modules and their dependencies. \
Do not modify any files." \
  --tools "Read,Grep,Glob,Bash" \
  --max-turns 20
```

### With budget cap

```bash
claude --settings '{"sandbox":{"enabled":true,"autoAllowBashIfSandboxed":true}}' \
  -p "Refactor the database layer in src/db/ to use connection pooling. \
Write tests for the changes." \
  --max-budget-usd 1.00 \
  --no-session-persistence
```

### JSON output for programmatic parsing

```bash
claude --settings '{"sandbox":{"enabled":true,"autoAllowBashIfSandboxed":true}}' \
  -p "List all TODO comments in the codebase as a JSON array." \
  --output-format json
```

### Injecting context via system prompt file

```bash
claude --settings '{"sandbox":{"enabled":true,"autoAllowBashIfSandboxed":true}}' \
  -p "Review this PR for issues." \
  --append-system-prompt-file /path/to/review-guidelines.md
```

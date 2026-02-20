# Codex CLI Reference (Non-Interactive Mode)

Quick reference for invoking OpenAI's Codex CLI as an external subagent.

## Non-Interactive Invocation

Use `codex exec` to run Codex CLI in non-interactive (headless) mode:

```bash
codex exec "Your prompt here."
```

The `exec` subcommand runs the prompt to completion without user interaction and
prints the final output to stdout.

## Key Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--full-auto` | | Workspace-write sandbox with on-request approval policy. Recommended default for subagent use. |
| `--yolo` | | Alias for `--dangerously-bypass-approvals-and-sandbox`. Bypasses all approvals and sandboxing. Use with caution. |
| `--dangerously-bypass-approvals-and-sandbox` | | Disables all approval prompts and sandbox restrictions. Equivalent to `--yolo`. |
| `--model <model>` | `-m` | Override the model. Examples: `o4-mini`, `o3`, `codex-mini-latest`. |
| `--json` | | Output newline-delimited JSON instead of plain text. Each line is a JSON object representing an event. |
| `--output-last-message <path>` | `-o` | Write the final assistant message to the specified file path. |
| `--cd <path>` | `-C` | Set the working directory for the Codex session. Defaults to the current directory. |
| `--sandbox <mode>` | `-s` | Sandbox mode: `read-only`, `workspace-write`, or `danger-full-access`. |
| `--ask-for-approval <policy>` | `-a` | Approval policy: `untrusted` (ask for everything), `on-request` (auto-approve reads/writes, ask for commands), `never` (auto-approve all). |
| `--skip-git-repo-check` | | Allow running outside a git repository. By default Codex requires a git repo. |
| `--ephemeral` | | Do not persist the session. The conversation is discarded after execution. |

### Convenience Flag Details

`--full-auto` is equivalent to:

```
--sandbox workspace-write --ask-for-approval on-request
```

`--yolo` is equivalent to:

```
--sandbox danger-full-access --ask-for-approval never
```

For subagent use, prefer `--full-auto` unless the task specifically requires
unrestricted host access.

## Stdin Piping

You can pipe a prompt via stdin using `-` as the prompt argument:

```bash
echo "Explain this code." | codex exec -
```

You can also pipe file content as context:

```bash
cat src/auth.py | codex exec --full-auto "Review this code for security issues. The code is piped via stdin."
```

Note: When piping content, the piped data becomes part of the conversation
context. The prompt string still describes the task.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `CODEX_API_KEY` | API key for Codex CLI in exec mode. Takes priority over `OPENAI_API_KEY`. |
| `OPENAI_API_KEY` | OpenAI API key. Used if `CODEX_API_KEY` is not set. |
| `OPENAI_BASE_URL` | Custom API endpoint URL. Use this to point Codex at a proxy or compatible API. |

The exec mode checks `CODEX_API_KEY` first, then falls back to `OPENAI_API_KEY`.

## Sandbox Escape Note

Codex `exec` runs directly on the host by default -- it does not launch its own
outer sandbox around the parent process. However, if the **parent agent** (the
agent invoking Codex) is itself sandboxed, the `codex` binary may be
inaccessible or blocked from execution inside that sandbox.

To invoke Codex from a sandboxed parent agent:

- **Claude Code parent:** Set `dangerouslyDisableSandbox: true` on the Bash
  tool call that runs the `codex exec` command.
- **Other parents:** Use your agent's sandbox-escape mechanism for the
  invocation command only.

The `--full-auto` or `--sandbox` flags on the Codex command itself still control
how Codex sandboxes its own tool execution internally.

## Example Invocations

### Basic code review with full-auto

```bash
codex exec --full-auto \
  "Review the code in src/api/handlers.py for bugs, edge cases, and \
  improvements. Report findings as a numbered list."
```

### Read-only analysis

```bash
codex exec \
  --sandbox read-only \
  --ask-for-approval untrusted \
  "Analyze the architecture of this project. Identify the main modules, \
  their responsibilities, and how they interact. Do not modify any files."
```

### JSON output written to file

```bash
codex exec --full-auto --json \
  -o /tmp/codex-review.txt \
  "Summarize the changes in the last 5 git commits and flag any risky \
  modifications."
```

The `--json` flag outputs structured event data to stdout. The `-o` flag
writes just the final assistant message to the specified path, which is
typically more useful for capturing the subagent's conclusion.

### Setting working directory

```bash
codex exec --full-auto \
  -C /home/user/project \
  "Run the test suite and report any failures with suggested fixes."
```

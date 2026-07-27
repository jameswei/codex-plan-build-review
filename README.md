# Plan, Build, Review

[English](README.md) | [简体中文](README.zh-CN.md)

A role-based workflow for delivering meaningful milestones through
independent review, explicit approval, and task-level checkpoints — works
with both [Codex](https://developers.openai.com/codex/) and
[Claude Code](https://code.claude.com/).

## Why use it

- A planner creates an evidence-grounded plan; a separate reviewer must pass it.
- Nothing is implemented before explicit user approval.
- Work proceeds one task at a time through build, validation, review, and
  checkpoint commit.
- Every review pass uses a fresh reviewer to reduce builder-reviewer drift.
- A final integration review checks cumulative behavior and cross-task
  contracts.
- Publication requires explicit approval, and the workflow never merges
  automatically.

```text
preflight → plan → review → approve → build task → review task → checkpoint
          → repeat → integration review → publication approval
```

## Roles

| Role | Responsibility | Access | Codex model · effort | Claude model · effort |
| --- | --- | --- | --- | --- |
| `milestone-explorer` | Bounded investigation | Read only | Terra · medium | Haiku |
| `milestone-planner` | Decision-complete milestone plan | Read only | Sol · xhigh | Opus · xhigh |
| `milestone-builder` | One approved task | Workspace write | Sol · medium | Sonnet · high |
| `milestone-reviewer` | Plan, task, and integration review | Read only | Sol · high | Opus · high |

The role definitions are bundled and can be tuned for either runtime. Haiku
doesn't support a configurable reasoning-effort level, so the Claude column
is left blank for `milestone-explorer`.

## Install

### Codex

Clone the repository into the Codex skills directory using the internal skill
name:

```sh
git clone https://github.com/jameswei/plan-build-review.git \
  ~/.codex/skills/plan-build-review
```

Install the four custom agent definitions:

```sh
mkdir -p ~/.codex/agents
cp ~/.codex/skills/plan-build-review/assets/agents/*.toml ~/.codex/agents/
```

Review existing files before copying if you already have agents with the same
names. Start a new Codex session after installation so the skill and agents are
discovered. (Codex also supports a project-scoped `.agents/skills` location
alongside the user-scoped `~/.codex/skills/` path used above, if you'd rather
install per-repository.)

### Claude Code

Add this repository as a plugin marketplace, then install the plugin:

```text
/plugin marketplace add jameswei/plan-build-review
/plugin install plan-build-review@plan-build-review
/reload-plugins
```

This installs the skill and all four subagent definitions in one step, with
no manual file copying. Once installed, the skill and roles are namespaced
under the plugin name, e.g. `milestone-planner` becomes
`plan-build-review:milestone-planner`.

## Use

Invoke the skill explicitly:

```text
Use $plan-build-review to deliver milestone v1.0 for this repository.
```

in Codex, or in Claude Code:

```text
Use /plan-build-review:plan-build-review to deliver milestone v1.0 for this repository.
```

The workflow stops after the reviewed plan and waits for approval before it
creates a branch or edits implementation files. It also asks again before
pushing or opening a draft pull request.

Use it for milestone work where scope, correctness, and review traceability
matter. It intentionally does not trigger for ordinary coding or documentation
tasks.

## Compatibility

This package supports both Codex and Claude Code; no other runtime is
targeted. Installation isn't equally frictionless on both: Claude Code's
plugin marketplace installs the skill and all four subagents in one step,
with zero manual file copying. Codex has no mechanism to bundle custom agent
definitions with a skill install, so its custom agents still require the
one-time manual copy described above.

Read-only enforcement also isn't equivalent. Codex's `sandbox_mode:
read-only` blocks filesystem writes at the OS level even inside a shell
command. Claude Code has no equivalent execution sandbox, so
`milestone-explorer`, `milestone-planner`, and `milestone-reviewer` enforce
read-only access by omitting Bash and MCP tools from their `tools` allowlist
entirely, rather than sandboxing them — they simply aren't granted the
tools that could write.

## License

[MIT](LICENSE)

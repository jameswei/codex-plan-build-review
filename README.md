# Plan, Build, Review

[English](README.md) | [简体中文](README.zh-CN.md)

A role-based workflow for delivering meaningful scoped work, from features to
milestones, through
independent review, explicit approval, and task-level checkpoints — works
with both [Codex](https://developers.openai.com/codex/) and
[Claude Code](https://code.claude.com/).

## Why use it

- A planner creates an evidence-grounded, minimal plan; a separate reviewer
  must pass both its premises and specification.
- Nothing is implemented before explicit user approval.
- Work proceeds one task at a time through build, validation, review, and
  checkpoint commit.
- Every review pass uses a fresh reviewer and a risk-based charter, not a
  generic checklist.
- A final integration review checks cumulative behavior and cross-task
  contracts.
- Publication requires explicit approval, and the workflow never merges
  automatically.

```text
preflight → plan → review → approve → build task → review task → checkpoint
          → repeat → integration review → publication approval
```

Each role receives a short decision summary plus the plan and evidence needed
for its assignment. The workflow does not use a whole plan as default
context for every handoff.

## What it prevents, and where it helps

This workflow prevents a plausible plan from being treated as a sufficient
reason to build. It exposes unchallenged product assumptions, scope expansion,
unjustified compatibility or security work, incomplete merge units, and tests
that accidentally depend on their development environment.

It is most useful when a feature or milestone crosses several components,
needs an owner decision before implementation, or must leave a trustworthy
review and release trail. Its value is not more agents or more ceremony: it
gives each role a distinct challenge, a small authoritative context, and a
clear escalation path while the main agent maintains the whole outcome.

## Roles

| Role | Responsibility | Access | Codex model · effort | Claude model · effort |
| --- | --- | --- | --- | --- |
| `pbr-explorer` | Bounded investigation | Read only | Terra · medium | Haiku |
| `pbr-planner` | Minimal, decision-complete plan | Read only | Sol · xhigh | Opus · xhigh |
| `pbr-builder` | One approved task | Workspace write | Sol · medium | Sonnet · high |
| `pbr-reviewer` | Premise, plan, task, and integration review | Read only | Sol · high | Opus · high |

The role definitions are bundled and can be tuned for either runtime. Haiku
doesn't support a configurable reasoning-effort level, so the Claude column
is left blank for `pbr-explorer`.

## Review model

Plan review has two required judgments. The premise review challenges the
purpose, target user, architecture, deployment model, compatibility posture,
security posture, and roadmap delta. The specification review checks whether
the chosen plan is complete and executable. A plan passes only when both pass.

For implementation work, the main agent selects the smallest sufficient review
charter: for example, implementation behavior, a final public surface, or
cross-task integration. A task must be a semantically complete unit that can be
merged and verified on its own. The workflow does not split work mechanically
by file type or code layer.

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
names. The `pbr-` prefix avoids collisions because Codex installs agents into a
flat global directory. Start a new Codex session after installation so the skill and agents are
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
under the plugin name, e.g. `pbr-planner` becomes
`plan-build-review:pbr-planner`.

## Use

Invoke the skill explicitly:

```text
Use $plan-build-review to deliver feature export-history for this repository.
```

in Codex, or in Claude Code:

```text
Use /plan-build-review:plan-build-review to deliver feature export-history for this repository.
```

The workflow stops after the reviewed plan and waits for approval before it
creates a branch or edits implementation files. It also asks again before
pushing or opening a draft pull request.

Use it for scoped feature or milestone work where scope, correctness, and review traceability
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
`pbr-explorer`, `pbr-planner`, and `pbr-reviewer` enforce
read-only access by omitting Bash and MCP tools from their `tools` allowlist
entirely, rather than sandboxing them — they simply aren't granted the
tools that could write.

## License

[MIT](LICENSE)

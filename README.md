# Codex Plan, Build, Review

A role-based Codex skill for delivering meaningful milestones through
independent review, explicit approval, and task-level checkpoints.

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

| Role | Responsibility | Access | Model and effort |
| --- | --- | --- | --- |
| `milestone-explorer` | Bounded investigation | Read only | Terra, medium |
| `milestone-planner` | Decision-complete milestone plan | Read only | Sol, xhigh |
| `milestone-builder` | One approved task | Workspace write | Sol, medium |
| `milestone-reviewer` | Plan, task, and integration review | Read only | Sol, high |

The role definitions are bundled and can be tuned for your Codex environment.

## Install

Clone the repository into the Codex skills directory using the internal skill
name:

```sh
git clone https://github.com/jameswei/codex-plan-build-review.git \
  ~/.codex/skills/plan-build-review
```

Install the four custom agent definitions:

```sh
mkdir -p ~/.codex/agents
cp ~/.codex/skills/plan-build-review/assets/agents/*.toml ~/.codex/agents/
```

Review existing files before copying if you already have agents with the same
names. Start a new Codex session after installation so the skill and agents are
discovered.

## Use

Invoke the skill explicitly:

```text
Use $plan-build-review to deliver milestone v1.0 for this repository.
```

The workflow stops after the reviewed plan and waits for approval before it
creates a branch or edits implementation files. It also asks again before
pushing or opening a draft pull request.

Use it for milestone work where scope, correctness, and review traceability
matter. It intentionally does not trigger for ordinary coding or documentation
tasks.

## Compatibility

This package is Codex-specific. Its workflow principles are portable, but its
skill metadata, custom agents, sandbox settings, reasoning controls, and
subagent context handling rely on Codex.

## License

[MIT](LICENSE)

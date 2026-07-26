# Codex Plan, Build, Review

A gated, role-based Codex workflow for independently reviewed milestone
delivery.

This skill separates planning, implementation, and review into focused Codex
agents while keeping the user and main agent in control of scope and approval
gates.

## What it does

The workflow moves a meaningful milestone through:

1. repository preflight and evidence gathering;
2. independent milestone planning and plan review;
3. explicit user approval;
4. task-by-task implementation and review checkpoints;
5. final cross-task integration review; and
6. explicit approval before publication.

Later tasks cannot begin until the current task has passed validation, received
an independent review, and been committed as a stable checkpoint.

## Codex roles

| Role | Purpose | Default model | Reasoning effort |
| --- | --- | --- | --- |
| `milestone-explorer` | Bounded, read-heavy investigation | `gpt-5.6-terra` | `medium` |
| `milestone-planner` | Evidence-grounded milestone planning | `gpt-5.6-sol` | `xhigh` |
| `milestone-builder` | One approved implementation task | `gpt-5.6-sol` | `medium` |
| `milestone-reviewer` | Independent plan, task, and integration review | `gpt-5.6-sol` | `high` |

The builder deliberately uses medium reasoning because it receives one bounded,
reviewed task at a time. Higher-effort reviewers and validation gates provide
independent checks around implementation.

## Compatibility

This repository is designed specifically for Codex. The workflow principles
can be adapted elsewhere, but the packaged skill relies on Codex skills, named
custom agents, sandbox settings, reasoning-effort configuration, and subagent
context controls.

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

## Repository structure

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
└── assets/
    └── agents/
        ├── milestone-builder.toml
        ├── milestone-explorer.toml
        ├── milestone-planner.toml
        └── milestone-reviewer.toml
```

`SKILL.md` defines the workflow. `agents/openai.yaml` provides skill UI
metadata. The TOML files under `assets/agents/` are installed separately as
Codex custom agents.

## License

[MIT](LICENSE)

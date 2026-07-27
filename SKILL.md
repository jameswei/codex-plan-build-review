---
name: plan-build-review
description: Plan and execute explicitly requested milestone work through independent plan review, user approval, task-level build and review checkpoints, integration review, and controlled publication. Use only when the user explicitly invokes this skill by name (for example, $plan-build-review in Codex or /plan-build-review:plan-build-review in Claude Code) or clearly asks for this workflow in other words; do not invoke it for ordinary coding, review, or documentation tasks.
---

# Plan, Build, Review

Coordinate milestone work without mixing planning, implementation, and review.
The main agent owns scope, sequencing, user communication, and approval gates.

Spawn every named role with a fresh, isolated context and a self-contained
task packet.

- In Codex, always call `spawn_agent` with `fork_turns="none"` — the default
  is `fork_turns="all"`, which inherits the entire parent conversation.
- In Claude Code, use a normal non-fork custom subagent invocation.

Do not rely on or inherit parent conversation history.

## 1. Preflight

1. Read applicable repository instructions and status documents.
2. Inspect the base branch, working tree, relevant code, tests, and public
   contracts.
3. Preserve unrelated changes. Stop if they prevent safe work.
4. Identify the milestone version and a lowercase hyphenated slug.

## 2. Plan and Review

1. Spawn `milestone-explorer` only for a bounded investigation that materially
   improves planning.
2. Spawn `milestone-planner` with the request, repository instructions,
   preflight evidence, and explorer findings. Require a decision-complete plan
   with ordered, reviewable tasks.
3. Reconcile the draft with repository evidence in the main thread.
4. Spawn a fresh `milestone-reviewer` with the request, evidence, and draft.
   Require one verdict: `PASS`, `CHANGES_REQUIRED`, or `BLOCKED`.
5. Return accepted blocking findings to the same planner, then use a fresh
   reviewer. Ask the user when a material decision or missing evidence blocks
   planning, or the same blocker survives two review cycles.
6. Present only a passed plan, its assumptions, and non-blocking notes. Stop for
   explicit user approval.

Do not create a branch, save the plan, or implement before approval. Revisions
return to planning and require another plan review and approval.

## 3. Establish the Milestone

1. Recheck the branch and working tree.
2. Create `milestone/<version>-<slug>` from the agreed base.
3. Save the approved plan as `docs/plans/<version>-<slug>.md` and commit it
   before implementation.
4. Create a task ledger from the approved tasks. Track `pending`, `building`,
   `reviewing`, `accepted`, or `blocked`, plus checkpoint and validation
   evidence.

Keep the approved plan immutable. Record progress in the ledger and commits.

## 4. Execute One Task at a Time

Do not begin a later task until the current task passes review, validation, and
has a stable checkpoint commit.

For each task:

1. Mark it `building` and record its review base.
2. Spawn a fresh `milestone-builder` with the approved plan, one task,
   repository instructions, review base, accepted-task summaries, working-tree
   state, and validation commands.
3. Inspect the returned delta and evidence. Split an oversized task only when
   its outcome, scope, dependencies, and contracts remain unchanged; otherwise
   return to planning and approval.
4. Mark it `reviewing`. Spawn a fresh `milestone-reviewer` with the request,
   approved plan, task, review-base delta, repository instructions,
   accepted-task summaries, and validation evidence.
5. On `CHANGES_REQUIRED`, send accepted blocking findings to the same builder,
   rerun validation, and use a fresh reviewer. On `BLOCKED`, obtain the missing
   evidence, capability, or user decision. Stop when the same blocker survives
   two review cycles.
6. After `PASS`, rerun required validation and confirm the reviewed delta is
   unchanged. Review any tracked output changed by validation.
7. Commit the accepted delta with its task ID, mark it `accepted`, and report
   the checkpoint and evidence.

## 5. Integration Review

1. After all tasks are accepted, run the complete validation suite.
2. Spawn a fresh `milestone-reviewer` with the request, approved plan, ledger,
   checkpoints, verdicts, validation evidence, repository instructions, and
   full branch diff.
3. Review cross-task behavior, cumulative acceptance criteria, scope, public
   contracts, regressions, and omitted requirements.
4. Treat required fixes as a narrow task: build, validate, review, and
   checkpoint it before repeating integration review.

The milestone is complete only after `PASS`.

## 6. Visibility

Use the native plan display when available. Show setup, each task, integration
review, and publication; keep it synchronized with the ledger.

At each state change, report the stage, active task and review pass, blocking
gate, and next expected event. Include task ID, role, and review pass in agent
task names. During long operations, post a brief heartbeat at least once per
minute.

## 7. Publication Gate

Before requesting publication approval, confirm that all tasks and integration
review passed, required validation is green, the branch contains only accepted
work, and PR or release prerequisites are available.

Summarize the outcome, checkpoints, tests, final verdict, non-blocking notes,
and working-tree state. Ask before pushing or opening a draft pull request.
Never merge automatically.

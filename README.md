# shiplog-codex

**A Codex-native captain's log for software work.** Plans, discoveries, commits, reviews, and closures become durable GitHub artifacts instead of chat-only memory.

This package adapts the original [shiplog](https://github.com/devallibus/shiplog) workflow to how Codex actually operates:

- explicit `$shiplog-codex` invocation instead of `/shiplog`
- direct `git` and `gh` commands as the default path
- `update_plan` posture instead of relying on a separate plan-mode switch
- no implicit sub-agent spawning
- review contracts and durable handoffs when independent review is needed

## Quick Install

If you already have this folder locally, install it into Codex by copying it into your skills directory.

### PowerShell

```powershell
Copy-Item -Recurse -Force .\shiplog-codex $HOME\.codex\skills\shiplog-codex
```

### Bash

```bash
cp -r ./shiplog-codex ~/.codex/skills/shiplog-codex
```

Then invoke it explicitly in Codex:

```text
Use $shiplog-codex to brainstorm this feature and capture it as an issue.
Use $shiplog-codex to resume issue #42.
Use $shiplog-codex to prepare the PR timeline for this branch.
```

## Why This Exists

AI coding sessions reset constantly. The code survives, but the reasoning usually does not.

You may remember the diff, but not:

- which alternatives were rejected
- why a branch was split
- what surprising dependency appeared mid-work
- whether a PR fully closed the issue or only shipped part of it
- which model wrote or reviewed which artifact

**shiplog-codex** turns GitHub and Git into a durable workflow history so another session, another model, or another person can recover the state of the work without guessing.

```text
Brainstorm -> Issue -> Branch -> Commits -> PR
    ^                          ^
    |        Discoveries -> New Issues / Stacked Work
    |
    +-> Search later across issues, PRs, commits, and shiplog artifacts
```

## What You Get

**Plans become issues.** Brainstorms turn into structured GitHub issues with tasks, acceptance criteria, and verification expectations.

**Branches stay tied to issue IDs.** The workflow uses `issue/<id>-<slug>` so retrieval stays cheap and predictable.

**Commits carry context.** Significant commits can be paired with reasoning and verification notes instead of leaving intent buried in chat.

**PRs tell the full story.** The PR timeline captures the original plan, discoveries, implementation issues, decisions, testing, and future-facing notes.

**Discoveries get routed, not forgotten.** Mid-work findings can be fixed inline, split into a prerequisite, or recorded as a new issue with explicit cross-links.

**Review and closure stay honest.** No issue closes without evidence, and no PR should merge without independent review.

## How It Works

| Step | Result |
|------|--------|
| **Brainstorm** | A plan is captured as an issue with tasks and verification expectations |
| **Branch** | Work starts from an issue-linked branch, ideally in its own worktree |
| **Discover** | Mid-work findings are classified and routed |
| **Commit** | Conventional commits reference the issue and, when useful, a task |
| **PR** | A timeline-style PR body records the journey |
| **Lookup** | Later sessions can search issues, PRs, commits, and envelopes |

## Full Mode and Quiet Mode

**Full Mode** writes the history directly into issues and PRs. This is the default and usually the best fit for personal projects and OSS.

**Quiet Mode** keeps the clean feature PR separate from the full reasoning trail by using a `--log` branch and PR.

```text
feature/my-change
  -> clean feature PR

feature/my-change--log
  -> shiplog knowledge trail PR
```

## Codex-Specific Behavior

This adaptation stays close to the original shiplog workflow, but it changes a few operational assumptions so the skill feels native in Codex.

### Invocation

Use explicit prompts such as:

- `Use $shiplog-codex to brainstorm this idea`
- `Use $shiplog-codex to resume issue #239`
- `Use $shiplog-codex models for this repo`

Do not expect `/shiplog` slash commands to exist.

### Planning posture

When a phase recommends planning, the skill uses `update_plan` and a read-first posture. It does not assume a separate mode switch exists in the runtime.

### Execution path

The default implementation path is direct `git` and `gh`, not optional helper skills. Helper skills can still be treated as extras if they are available, but they are not required for the workflow to function.

### Delegation

Sub-agents are not assumed. Bounded handoff contracts and review contracts are first-class so work can still move between tools or people cleanly.

## Key Features

### Cross-model review gate

The workflow treats independent review as part of the safety model. Same-model self-review can be logged as an audit artifact, but it does not satisfy the merge gate.

### Provenance signing

Artifacts use `Authored-by:`, `Updated-by:`, and `Reviewed-by:` lines so authorship and review history remain searchable.

### Tier and posture routing

The routing guide helps decide when work is best done as high-judgment planning versus straightforward execution, without requiring Codex to expose a dedicated mode toggle.

### Discovery protocol

Unexpected findings are classified before action:

```text
Discovery made during work
  +- Small and local?         -> Fix inline
  +- Blocks current work?     -> Create prerequisite issue/branch
  +- Important but separate?  -> New issue, continue current task
  +- Cleanup opportunity?     -> Refactor issue
```

### Task-level delivery

Issues can be split into task IDs like `T1`, `T2`, and commits can reference them directly, for example `feat(#42/T1): add JWT validation`.

### Verification profiles

The workflow supports repo, issue, and task-level verification policy in `.shiplog/verification.md`, including behavior specs, red-green testing, self-audit, structural checks, and mutation-oriented workflows.

### Artifact envelopes

Machine-readable HTML comment envelopes make it possible to retrieve current state and history without reading every long comment thread end to end.

### Evidence-linked closure

Issues should only close with proof: merged PRs, commits on the default branch, or decision artifacts that clearly satisfy the work.

### Label bootstrapping

The skill maintains a compact `shiplog/*` label vocabulary so issue and PR state is visible at a glance.

## Installation in Codex

### Install from this local export

The current export is intended to live as:

```text
~/.codex/skills/shiplog-codex
```

On this machine that resolves to:

```text
C:\Users\cheap\.codex\skills\shiplog-codex
```

### Install manually from a working folder

### PowerShell

```powershell
Copy-Item -Recurse -Force <path-to-shiplog-codex> $HOME\.codex\skills\shiplog-codex
```

### Bash

```bash
cp -r <path-to-shiplog-codex> ~/.codex/skills/shiplog-codex
```

### Local development loop

If you are refining the skill locally, keep editing the project copy and then copy it back into `~/.codex/skills/shiplog-codex` after validation.

## Files

| Path | Purpose |
|------|---------|
| `SKILL.md` | Main activation and orchestration rules |
| `agents/openai.yaml` | Codex interface metadata |
| `references/brainstorm.md` | Plan capture guidance |
| `references/branch.md` | Branch, worktree, and session-start workflow |
| `references/discovery.md` | Discovery routing protocol |
| `references/commit.md` | Commit-context workflow |
| `references/pr.md` | PR timeline workflow |
| `references/lookup.md` | Decision and history retrieval |
| `references/timeline.md` | Ongoing timeline artifact patterns |
| `references/artifact-envelopes.md` | Machine-readable metadata conventions |
| `references/closure-and-review.md` | Review and closure policy |
| `references/model-routing.md` | Tier and posture guidance for Codex |
| `references/verification-profiles.md` | Verification policy |

## Configuration

The skill can run without config, but these repo-local files make behavior more consistent:

| File | Purpose |
|------|---------|
| `.shiplog/mode.md` | `full` or `quiet` mode selection |
| `.shiplog/routing.md` | routing prompt behavior |
| `.shiplog/verification.md` | default verification profiles |

## Example Prompts

```text
Use $shiplog-codex to brainstorm a new onboarding flow and turn it into an issue.
Use $shiplog-codex to resume work on issue #118 from the current branch.
Use $shiplog-codex to record this implementation issue before we keep coding.
Use $shiplog-codex to create the PR timeline for this branch.
Use $shiplog-codex to find where we decided to keep the background job synchronous.
```

## Relationship to Upstream shiplog

This package is an adaptation of the upstream [shiplog](https://github.com/devallibus/shiplog) project for Codex usage. The goal is fidelity to the original workflow shape, with the runtime-specific changes concentrated around:

- activation style
- planning posture
- delegation assumptions
- review-contract fallback behavior
- install instructions for the Codex skill layout

## Requirements

- `gh` CLI authenticated against GitHub
- `git`
- a repository with a GitHub remote

Everything else is optional.

## License

This adaptation follows the workflow and structure of the upstream project. Check the upstream repository for the authoritative license and project metadata.

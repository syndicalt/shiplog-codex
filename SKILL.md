---
name: shiplog-codex
description: Git-as-knowledge-graph workflow for traceability, adapted for Codex. Use when planning work, brainstorming designs, creating or managing issues and PRs, tracking architectural decisions, or resuming prior sessions. Invoke explicitly with $shiplog-codex.
---

# Shiplog Codex

The captain's log for your codebase. Every decision, discovery, and change logged as you ship code.

Use GitHub as a complete knowledge graph where every brainstorm, commit, review, and decision is traceable. This skill mirrors the original **shiplog** workflow closely, but adapts the activation, routing, and execution mechanics to Codex.

## Core Principle

**Nothing gets lost.** Every brainstorm becomes an issue. Every issue drives a branch. Every branch produces a PR. Every PR is a timeline of the entire journey. Git becomes the durable memory.

## Codex Adaptation

- Invoke explicitly with `$shiplog-codex`, `$shiplog-codex models`, or `$shiplog-codex <phase>`. Do not rely on `/shiplog` slash commands existing.
- Use `git` and `gh` directly as the default execution path. Treat `superpowers:*`, `ork:*`, and similar Claude-oriented helpers as optional extras, not required infrastructure.
- When a phase recommends `plan` work, use `update_plan`, gather context, and avoid mutating commands until the plan or classification is clear. There is no dedicated Codex plan-mode toggle to depend on.
- Persist repo-specific behavior in `.shiplog/` files inside the repository when useful. Do not depend on cross-session memory services being present.
- Do not spawn sub-agents unless the user explicitly asked for delegation. When independent review or bounded verification is needed, default to a review contract or handoff artifact instead.

## Mode Selection

On first substantial activation per project, check `.shiplog/mode.md`.

- **Full Mode** (default): Knowledge goes directly into issues and PRs. Best for personal projects, OSS, and teams that want documentation in the open.
- **Quiet Mode**: Knowledge lives in a stacked knowledge branch (`<branch>--log`) with its own PR targeting the feature branch. Best when feature PRs and issues must stay clean.

If `.shiplog/mode.md` is missing and the choice matters, ask the user. If the choice does not materially affect the current step, default to `Full Mode` and note the assumption in the next **shiplog** artifact.

## When This Skill Activates

**User-invocable:** `$shiplog-codex`, `$shiplog-codex models`, `$shiplog-codex <phase>`

**`$shiplog-codex models`:** Re-runs the routing setup guidance. See [references/model-routing.md](references/model-routing.md).

Use this skill when any of these occur:

- The user says "let's plan", "let's brainstorm", or "let's design" and wants the result captured.
- The user explicitly requests traceability or **shiplog**-style workflow history.
- The user wants a new issue, branch, or PR that documents decisions.
- Mid-work discovery creates a new issue, stacked dependency, or blocker.
- The user asks "where did we decide X?" or "what's the status of Y?"
- The user wants to resume work on an issue, PR, or `issue/<id>-...` branch.
- The user wants to apply review feedback, finish a PR, or merge with durable context.

Do not auto-activate for:

- Generic coding work that does not need durable workflow history.
- Small fixes where a lightweight branch or commit is enough.
- Cases where a more specific skill is clearly the better fit.

## Decision Tree

Match the user's intent and load the corresponding phase reference:

```text
User request arrives
  +--> [If .shiplog/routing.md missing: use setup from references/model-routing.md]
  +--> ["$shiplog-codex models": re-run routing setup, update config]
  |
  +-- "Let's brainstorm/plan/design X" -> references/brainstorm.md
  +-- "Work on issue #N"               -> references/branch.md
  +-- "I found a sub-problem"          -> references/discovery.md
  +-- "Let's commit"                   -> references/commit.md
  +-- "Ready for PR"                   -> references/pr.md
  +-- "Where did we decide X?"         -> references/lookup.md
  +-- Currently mid-work on a branch   -> references/timeline.md
```

## ID-First Naming Convention

All artifacts use `#ID` as the primary key for fast, token-efficient retrieval.

**Semantic tag vocabulary** for user-facing headings: `plan`, `session-start`, `commit-note`, `discovery`, `blocker`, `implementation-issue`, `review-handoff`, `worklog`, `history`, and `amendment`. Format: `[shiplog/<kind>] <human title>`.

| Artifact | Convention | Example |
|----------|-----------|---------|
| Branch | `issue/<id>-<slug>` | `issue/42-auth-middleware` |
| Commit | `<type>(#<id>): <msg>` | `feat(#42): add JWT validation` |
| Commit (task) | `<type>(#<id>/<Tn>): <msg>` | `feat(#42/T2): add middleware chain` |
| PR title | `<type>(#<id>): <msg>` | `feat(#42): add auth middleware` |
| PR body (closes) | `Closes #<id>` | `Closes #42` |
| PR body (partial) | `Addresses #<id> (completes ...)` | `Addresses #42 (completes T1, T2)` |
| Task in issue | `- [ ] **T<n>: Title** [tier-N]` | `- [ ] **T1: Add JWT** [tier-3]` |
| Timeline comment | `[shiplog/<kind>] #<id>: ...` | `[shiplog/discovery] #42: race condition` |
| Stacked branch | `issue/<new-id>-<slug>` | `issue/43-fix-race-condition` |
| Stacked PR title | `<type>(#<new-id>): ... [stack: #<parent>]` | `fix(#43): race cond [stack: #42]` |

**Quiet Mode overrides:**

| Artifact | Convention | Example |
|----------|-----------|---------|
| Feature branch | per team convention | `feature/auth-middleware` |
| Knowledge branch | `<branch>--log` | `feature/auth-middleware--log` |
| Knowledge PR title | `[shiplog/worklog] <desc>` | `[shiplog/worklog] auth middleware decisions` |
| Knowledge PR base | the feature branch | base: `feature/auth-middleware` |

**Task IDs:** Tasks carry local IDs (`T1`, `T2`, ...) scoped to the issue. Commits use `#<id>/<Tn>`.

**Retrieval:** `gh issue list --search "#42"` | `git log --grep="#42"` | `git log --grep="#42/T1"` | `gh pr list --search "#42"`

## User-Facing Language

The phase numbers are internal workflow labels. Do not surface them to the user.

Preferred labels: `Plan Capture`, `Branch Setup`, `Discovery Handling`, `Commit Context`, `PR Timeline`, `History Lookup`, `Timeline Updates`.

**Brand formatting:** Always bold the word **shiplog** in user-facing text when markdown rendering exists. Keep machine-readable identifiers lowercase and unformatted.

## Shell Portability

Keep the workflow cross-platform. See [references/shell-portability.md](references/shell-portability.md) for full guidance and Bash/PowerShell patterns.

Key rules:

- Prefer `gh ... --body-file <temp-file>` for multiline content.
- Break chained shell commands into separate steps when shell operators differ.
- Keep Bash examples as the primary path and add PowerShell-safe variants when quoting diverges.

## GitHub Labels

**shiplog** manages a compact repo-level label vocabulary so issues and PRs stay filterable even when a reader never opens the body. See [references/labels.md](references/labels.md) for the canonical set, descriptions, and CLI snippets.

Label rules:

- On the first write operation in a repo, bootstrap or refresh labels with `gh label create --force ...`.
- Apply labels at creation time with `gh issue create --label` or `gh pr create --label`.
- `shiplog/blocker` is stateful. Add it when work becomes blocked and remove it when the blocker clears.
- `shiplog/ready`, `shiplog/in-progress`, and `shiplog/needs-review` are mutually exclusive lifecycle labels.

## Triage Field Maintenance

Issue envelope triage fields (`readiness`, `task_count`, `tasks_complete`, `max_tier`) and lifecycle labels must stay current so triage scans remain accurate.

| Event | Envelope update | Label update |
|-------|-----------------|--------------|
| Issue created (Phase 1) | Set all four triage fields at creation | Apply `shiplog/ready` if tasks are scoped and no blockers exist |
| Branch created (Phase 2) | Set `readiness: in-progress` | Replace lifecycle label with `shiplog/in-progress` |
| Task checked off (Phase 4) | Increment `tasks_complete`, recompute `max_tier` from remaining tasks | - |
| All tasks complete | Set `readiness: done`, clear `max_tier` | - |
| Blocker found (Phase 3) | Set `readiness: blocked` | Add `shiplog/blocker` |
| Blocker cleared | Restore previous `readiness` (`in-progress` or `ready`) | Remove `shiplog/blocker` |
| PR created (Phase 5) | Set `readiness: done` if all tasks shipped | Replace lifecycle label with `shiplog/needs-review` |
| PR merged and issue closed | - | Remove all lifecycle labels |

Edit the issue body in place when these fields change. Triage metadata is derived state, so refreshing it does not require `Updated-by:` provenance.

## Mandatory Issue Capture

Implementation trouble that materially affects the work must be durably recorded before the next material step or before ending the turn.

### What counts as a relevant implementation issue

- Failed attempts
- Hidden dependencies
- Risky workarounds
- Scope surprises
- Verification gaps
- Environment or tooling friction

### What does not require capture

- Normal iteration where the final approach is obvious from the diff
- Minor typos or lint fixes resolved in the same commit
- Expected complexity that matches the task description

### Capture rule

| Situation | Artifact | Where |
|-----------|----------|-------|
| Issue is local and resolved inline | Timeline comment (`[shiplog/implementation-issue]`) | Issue (Full Mode) or `--log` PR (Quiet Mode) |
| Issue warrants follow-up, scope split, or long-term retrieval | New linked issue | GitHub issue with cross-reference on parent |

The timeline comment is the minimum: one paragraph explaining what happened, why it matters, and how it was resolved or deferred.

## Agent Identity Signing

Every **shiplog** artifact must carry a provenance signature: `<role>: <family>/<version> (<tool>[, <qualifier>])`

Roles: `Authored-by`, `Updated-by`, `Reviewed-by`. See [references/signing.md](references/signing.md) for full rules, edit provenance, and model detection.

## Integration Map

This skill orchestrates. For activities that directly produce **shiplog** artifacts, the convention-enforced workflows live in `references/`.

| Activity | Primary path in Codex | Optional extras | Shiplog adds |
|----------|-----------------------|-----------------|--------------|
| Committing | [references/commit-workflow.md](references/commit-workflow.md) | helper commit skills if installed | ID-first format, task refs, context comments |
| Creating PRs | [references/pr-workflow.md](references/pr-workflow.md) | helper PR skills if installed | Timeline body, envelopes, labels, review gate |
| Brainstorming | inline brainstorm + [references/brainstorm.md](references/brainstorm.md) | brainstorming helpers if installed | Issue creation from output |
| Planning | `update_plan` + issue tasks | plan-writing helpers if installed | Task-list mirroring and handoff shape |
| Plan execution | local execution guided by issue tasks | explicit user-requested delegation only | Timeline comments at checkpoints |
| Worktree creation | direct `git worktree` | worktree helper skills if installed | Branch-issue linking |
| Stacked PRs | direct `git` + `gh` | stacked-PR helpers if installed | Discovery-driven stacking protocol |
| Issue tracking | issue body updates + timeline | repo automation if installed | Checkbox and triage refresh |
| Fixing issues | direct implementation + timeline | issue-fix helpers if installed | Durable RCA and history |
| Storing decisions | issue or PR artifacts, `.shiplog/` notes | memory backends if installed | Structured retrieval |

**Graceful degradation:** Internalized workflow first, direct `git`/`gh` second, optional helper skills last.

## Edge Cases

- **No issue exists:** Let the user work. At first commit or PR, offer to create a tracking issue.
- **Mid-work activation:** Check the branch name for `issue/N-*`. If found, add a catch-up timeline comment via the timeline phase. If not, offer retroactive issue creation.
- **Small tasks (< 30 min):** Lightweight protocol. The issue can be optional, but the branch, commit format, and PR traceability still apply.
- **Hotfix / emergency:** Fix first. Create the issue and PR afterward, backfilling the timeline.
- **Quiet Mode - feature PR merges:** Close the `--log` PR. Knowledge stays preserved in closed PR history.
- **Quiet Mode - feature branch rebased:** Rebase the `--log` branch onto the updated feature branch and use `--force-with-lease`.

## Requirements

| Dependency | Purpose | Install |
|-----------|---------|---------|
| `gh` CLI | GitHub issue, PR, and comment operations | `brew install gh` / `winget install GitHub.cli` |
| `git` | Branch, commit, diff, and log operations | pre-installed |
| GitHub remote | Required for end-to-end issue and PR workflow | - |

All helper skills are optional. The canonical **shiplog** workflow for Codex lives in these references:

- [references/brainstorm.md](references/brainstorm.md)
- [references/branch.md](references/branch.md)
- [references/discovery.md](references/discovery.md)
- [references/commit.md](references/commit.md)
- [references/pr.md](references/pr.md)
- [references/lookup.md](references/lookup.md)
- [references/timeline.md](references/timeline.md)
- [references/artifact-envelopes.md](references/artifact-envelopes.md)
- [references/closure-and-review.md](references/closure-and-review.md)
- [references/commit-workflow.md](references/commit-workflow.md)
- [references/pr-workflow.md](references/pr-workflow.md)
- [references/phase-templates.md](references/phase-templates.md)
- [references/verification-profiles.md](references/verification-profiles.md)

## Quick Prompts

- `Use $shiplog-codex to brainstorm a plan for feature X and capture it as an issue.`
- `Use $shiplog-codex to resume issue #239 and finish the PR flow.`
- `Use $shiplog-codex to run model-routing setup for this repo.`
- `Use $shiplog-codex to find where we decided Y.`

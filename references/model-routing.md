# Model-Tier Routing Reference

Route AI models to **shiplog** phases based on cognitive demand. The skill cannot switch models by itself - only the user can. In Codex, "plan mode" is an execution posture, not a UI toggle: use `update_plan`, gather context first, and avoid mutating actions until the phase is understood.

---

## Phase Entry Check (Step 0)

Every phase begins with this check:

1. Read routing mode from per-issue override > `.shiplog/routing.md` > default (`confirm`).
2. If work is transferring to another model or tool, write a handoff comment per the Context Handoff Protocol below.
3. If mode is `off`, skip prompts and continue, but still respect the phase's recommended execution posture.
4. Compare the entering phase's tier and recommended posture to the previous phase's tier and posture. If both are the same, skip to Step 1.
5. If mode is `confirm`, emit the routing prompt and wait for user acknowledgment when a tier or posture change materially matters.
6. If mode is `warn`, emit the routing banner and continue immediately.

**Routing mismatch:** If the user continues without switching models, proceed normally. Never block or repeat the prompt.

---

## Routing Behavior

Configure how the skill communicates tier transitions. Set this in `.shiplog/routing.md`.

| Mode | Behavior | Default? |
|------|----------|----------|
| `confirm` | Pause at tier transitions and ask the user before proceeding | Yes |
| `warn` | Show a one-line banner at tier transitions and keep going | No |
| `off` | Silent. No routing prompts or banners; actual handoffs still apply when work transfers | No |

### Config file format

Create `.shiplog/routing.md` in your project root:

```markdown
# Model Routing

routing: confirm
```

If the file does not exist, the skill runs the setup prompt on first activation.

### `$shiplog-codex models`

Re-runs the setup prompt at any time and updates `.shiplog/routing.md`.

### Setup prompt

When no `.shiplog/routing.md` exists, or when the user runs `$shiplog-codex models`:

> How should I handle model-tier suggestions at phase transitions?
> - **confirm** (default) - I'll pause and ask before proceeding, so you can switch models if you want.
> - **warn** - I'll show a banner but keep going.
> - **off** - No routing prompts. Use this if you run the same model for everything.

---

## Tier Definitions

| Tier | Cognitive Profile | Use For |
|------|-------------------|---------|
| **tier-1** (reasoning) | Creative synthesis, architectural judgment, narrative construction | Brainstorming, PR timeline synthesis, complex discovery triage |
| **tier-2** (capable) | Context loading, scope judgment, structured documentation | Issue-to-branch, discovery protocol, knowledge retrieval |
| **tier-3** (fast) | Execution speed, template filling, routine operations | Commits, simple implementations, timeline checkpoints |

## Default Phase-to-Tier Mapping

| Phase | Default Tier | Recommended Posture | Rationale |
|-------|--------------|---------------------|-----------|
| 1. Brainstorm-to-Issue | tier-1 | plan | Analysis and design; no code changes expected |
| 2. Issue-to-Branch | tier-2 | plan then agent | Load context first, then create the branch and artifacts |
| 3. Discovery Protocol | tier-2 | plan then agent | Classify the discovery before acting on it |
| 4. Commit-with-Context | tier-3 | agent | Pure execution: staging, committing, posting |
| 5. PR-as-Timeline | tier-1 | plan | Narrative synthesis; compose the PR body before mutating GitHub |
| 6. Knowledge Retrieval | tier-2 | plan | Read-only search and synthesis |
| 7. Timeline Maintenance | tier-3 | agent | Posting comments and updates |
| Review execution | tier-2 | plan | Reviewing a diff should not modify code |

---

## Routing Prompt Format

At phase transitions, when the entering phase's tier differs from the previous phase's tier:

### `confirm` mode

```text
---
[shiplog routing] Entering <Phase Name> - recommends tier-X (<profile>) and <posture> work.
This is advisory - switch models if you want, or continue as-is.
Continue? (y / or switch models first)
---
```

### `warn` mode

```text
---
[shiplog routing] Entering <Phase Name> - recommends tier-X (<profile>) and <posture> work.
---
```

### `off` mode

No routing prompt or banner. Proceed directly unless work is actually transferring to another model or tool, in which case write the handoff artifact and continue.

### When to prompt

Only prompt when the tier or posture changes between consecutive phases. Same-tier, same-posture transitions produce no prompt.

---

## Per-Issue Override

Add a `## Model Routing` section to any GitHub issue body to override the project routing mode for that issue:

```markdown
## Model Routing

routing: off
```

This silences routing prompts for that specific issue only. It is useful for simple tasks where tier switching adds no value.

---

## Execution Posture

In addition to tier routing, **shiplog** recommends an execution posture for each phase.

| Posture | Purpose | What the agent may do |
|---------|---------|----------------------|
| **plan** | Analysis, design, classification, synthesis | Read files, search, compose text. Avoid mutations until the plan is clear. |
| **agent** | Execution, creation, modification | Full tool access - write files, run commands, post artifacts. |
| **plan then agent** | Two-step phases | Start with analysis, then move into execution within the same phase. |

### Tool notes

| Tool | Practical behavior |
|------|--------------------|
| **Codex** | Use `update_plan` for substantial planning. There is no dedicated plan-mode toggle to depend on. |
| **Claude Code** | If a true plan mode exists, it can map naturally to the `plan` posture. |
| **Cursor** | User-controlled mode switching can mirror the posture if desired. |

### Posture discipline in Codex

When the recommended posture is `plan`:

1. Call `update_plan` if the work is substantial.
2. Gather context without mutating files or external state.
3. Transition into execution only after the classification or plan is stable.

When the recommended posture is `plan then agent`, the planning portion covers context loading and classification; execution starts when the phase reaches its mutating step.

---

## Context Handoff Protocol

When work transfers between models or tools, a self-contained handoff ensures the receiving model has everything it needs.

### When to write a handoff

- Transitioning from a higher tier to a lower tier
- Switching tools
- Delegating bounded work to another model or reviewer
- Any time the next model will not have the current conversation context

In Codex, only delegate to sub-agents when the user explicitly asked for delegation. Otherwise, use the handoff format as a durable artifact for humans or other tools.

### Handoff template

```markdown
## [#<ID>] handoff: Phase N -> Phase M

**Tier transition:** tier-X -> tier-Y
**Current model:** <family>/<version> (<tool>)

### What was decided
- [Decision 1 - concrete, not conceptual]
- [Decision 2]

### What to do next
1. [Concrete action with file path]
2. [Concrete action with file path]

### Contract
- **Allowed files:** `path/to/file.ts`, `path/to/other.ts`
- **Must not change:** [files, APIs, behavior, or decisions out of scope]
- **Stop and ask if:** [condition requiring widening scope or judgment]
- **Verification required:** [tests, checks, or evidence required before claiming done]
- **Return artifact:** [diff summary, verification note, changed file list, blocker report]
- **Decision budget:** `none` | `narrow` | `full`

### Files to touch
- `path/to/file.ts` (create) - [what goes in it]
- `path/to/other.ts` (modify line N) - [what to change]

### Gotchas
- [Anything a cheaper model would get wrong without this warning]
```

### The golden rule

> If a tier-3 model reading this handoff would need to make a judgment call, the handoff is not specific enough. Rewrite it until every decision is pre-made.

For delegated execution, `Decision budget: none` should be the default.

### Using the handoff as a verifier contract

When closure or review work is delegated to another model, use this same handoff template as the bounded verifier contract. The supervising model keeps the closure or merge decision; the delegated verifier only inspects the named evidence, current file state, or PR diff and returns the required verification artifact.

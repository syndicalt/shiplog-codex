# Closure and Review

Evidence-linked issue closure and multi-model review protocol for **shiplog**. No issue closes without evidence. No PR merges without independent review.

---

## 1. Evidence-Linked Closure

### Core rule

Do not close an issue without linked evidence and a verification note.

### Evidence requirements

| Evidence type | When to use |
|---------------|-------------|
| Commit URL on default branch | The fix is a code change that has been merged |
| Merged PR URL | The fix is better represented by the full PR |
| Discussion or decision artifact | No code change; the issue was resolved by a decision, policy change, or external action |

**Preference order:** Commit on default branch > merged PR > discussion artifact. Use the most specific link available.

### Closure comment format

Every closure must include a comment, unless the issue is closed via a PR body `Closes #N` and that PR already contains the needed evidence trail. When closing manually:

```markdown
## [shiplog/history] #<ID>: Closure

**Evidence:** [URL to commit, PR, or decision artifact]
**Merged to default branch:** yes | no | n/a
**Verification:** [1-3 sentences - why this evidence satisfies the issue]
**Disposition:** fully resolved | superseded by #<N> | won't fix (reason)
```

### What counts as evidence

- A commit SHA on the default branch that addresses the issue
- A merged PR whose diff addresses the issue
- A comment, ADR, or external decision that makes the issue moot
- For umbrella issues: links to the child PRs or issues that collectively satisfy the parent

### What does not count

- Memory or assumption that the work was done
- A branch that exists but was never merged
- A PR that is still open or was closed without merging
- Partial resolution without acknowledging the remaining gap

### Escalation on ambiguity

If the match between the issue and the evidence is ambiguous:

1. **Do not close the issue.** Leave it open.
2. Post a comment explaining the ambiguity.
3. Tag the issue for human review or escalate to a higher-tier model.
4. If the user explicitly asked for delegation, you may prepare a bounded verifier contract using `model-routing.md`.

### Optional verifier workflow

Use this workflow when closure evidence is specific enough to audit but repetitive enough to hand off. In Codex, do not spawn a verifier automatically; either keep the audit local or generate the bounded verifier contract for another model or human.

**Supervisor responsibilities:**

- choose the candidate evidence links to inspect
- assemble a bounded verifier contract using the handoff template from `model-routing.md`
- keep closure judgment for ambiguous issues, umbrella issues, and any case where the verifier reports mismatch or low confidence

**Verifier may:**

- read the issue body and linked discussion
- inspect candidate commits and merged PRs
- inspect current file state on the default branch
- produce a signed verification note with evidence, confidence, and a recommended action

**Verifier may not:**

- reinterpret vague issue intent
- decide that a partial fix is "good enough"
- close umbrella issues or mixed-status roadmap issues on its own
- close any issue directly
- resolve ambiguous evidence by inference

**Required verifier output:**

- candidate fix artifact links
- whether the fix is merged to the default branch
- which parts of the issue are satisfied by the diff or current file state
- any unresolved mismatch or ambiguity
- confidence: `high` | `medium` | `low`
- recommended action: `close` | `keep open` | `escalate`

### Umbrella issues

Umbrella issues require:

- all child issues closed with their own evidence
- a summary comment on the umbrella linking each child's resolution
- the umbrella to remain open if any child is unresolved

### Partial delivery

When a PR ships some tasks from an issue but other tasks remain:

1. **Do not use `Closes #N`.** Use `Addresses #N (completes T1, T2, ...)`.
2. **The issue stays open.** Completed tasks are checked off; remaining tasks stay unchecked.
3. **Post a milestone comment** on the issue after merge, listing what shipped and what remains.
4. **Post a blocker comment** if remaining tasks are blocked on an external dependency.
5. **Final closure** happens when the last remaining task ships or when all remaining tasks are explicitly cancelled with a rationale.

---

## 2. Closure Scope

This policy applies to:

- backlog hygiene and issue triage
- recovery or history-reconciliation flows that close stale issues
- automated or semi-automated closure by **shiplog** workflows
- manual closure during PR merges via `Closes #N`

This policy does not normalize:

- silent closure of ambiguous issues
- closing issues "for housekeeping" without evidence
- closing issues because the branch exists

---

## 3. Multi-Model Review Protocol

### Core rule

Every PR merge requires a positive review from a model different from the author. Same-model self-review does not count as independent review.

### Why this matters

The review loop is part of the safety model. If **shiplog** captures the workflow but omits signed review, it misses one of the core mechanisms that makes the process trustworthy.

### Review artifacts

A review produces one of three artifacts:

| Artifact | Meaning |
|----------|---------|
| **Approve** | No issues found; merge is authorized |
| **Request changes** | Issues found; author must address before re-review |
| **Comment** | Observations that do not block merge |

For AI-operated **shiplog** reviews, these outcomes are recorded as signed comment artifacts. The `Disposition:` line is authoritative; GitHub review badges are advisory.

### Sign-off format

Every review comment must include:

```text
Reviewed-by: <family>/<version> (<tool>)
Disposition: approve | request-changes
Scope: <what was reviewed>
```

### Review sign-off comment template

```markdown
<!-- shiplog:
kind: verification
issue: <ISSUE_NUMBER>
pr: <PR_NUMBER>
updated_at: <ISO_TIMESTAMP>
-->

Reviewed-by: <family>/<version> (<tool>)
Disposition: approve | request-changes
Scope: <what was reviewed>
```

See `signing.md` for the full signing protocol.

### What constitutes "different model"

- Different model family counts
- Different version within the same family counts only if explicitly documented
- Same model, same version, different session does not count as independent
- Human review always counts as independent

### Review target selection

When asked to review PRs:

1. List open PRs in the repository.
2. Inspect the newest signed author-side artifact and any existing `Reviewed-by:` sign-offs.
3. Skip PRs whose newest verifiable author-side artifact was authored by the same model and version.
4. Review PRs where the latest signed activity is from a different model.
5. If all open PRs were last touched by the current model, tell the user that cross-model review needs a different reviewer and offer an audit-trail-only self-review if they still want one.
6. Only proceed with self-authored review if the user explicitly confirms.

Review artifacts normally live in PR bodies and issue or PR comments, not in formal GitHub review events.

---

## 4. Review Execution Ladder

Ordered from most to least desirable:

### Best: independent reviewer

Use another model or a human reviewer. In Codex, this is the default interpretation of the review gate.

### If delegation was explicitly requested: bounded review contract

When the user explicitly asks for delegation or sub-agents, prepare a bounded review contract and hand it to the reviewer:

```markdown
## Review Contract

**PR:** #<N> - <title>
**Author:** <model name> (<tool>)
**Branch:** <branch> -> <base>
**Diff command:** `gh pr diff <N>`

### What to review
- [Specific files or sections to focus on]
- [Key decisions to validate]

### Review checklist
- [ ] Changes match the issue requirements
- [ ] No unintended side effects or regressions
- [ ] Cross-references between files are consistent
- [ ] Templates and examples are correct
- [ ] Relevant implementation issues are durably captured

### Output required
Sign-off comment with:
- Reviewed-by line
- Disposition (approve / request-changes)
- Scope of review
- Any findings
```

### Fallback: generate a review contract for the user

If independent review is not directly available, generate the contract above so the user can hand it to another model or a human reviewer.

### Audit trail only

When independent review is genuinely unavailable and the user still wants an audit artifact:

```text
Reviewed-by: <family>/<version> (<tool>)
Disposition: self-review (does NOT satisfy gate - independent review required)
Scope: full diff
Note: Self-review recorded as audit trail. This PR must not merge until an independent cross-model review is completed.
```

Self-review is an audit artifact, not a gate-satisfying event. It must be paired with a review contract so the PR is not left in a dead-end state.

### Review completion: publication

A PR review is not complete until the signed review artifact is posted on the PR as a GitHub comment, unless the user explicitly asked for a dry run or local-only review.

If GitHub posting is blocked:

1. Report the blocker immediately.
2. Provide the exact signed review artifact text in chat.
3. Mark the review as pending publication, not complete.
4. Retry posting later or let the user post it manually.

---

## 5. Merge Authorization

A PR may be merged when:

1. At least one cross-model review with `Disposition: approve` exists.
2. All `request-changes` reviews have been addressed.
3. The PR body includes `Closes #<N>` for full closure, or the partial-delivery protocol is followed.
4. The issue closure has linked evidence.

### Implementation issue capture check

Reviewers must check whether failed attempts, hidden dependencies, risky workarounds, scope surprises, and verification gaps were captured as durable artifacts instead of remaining only in chat memory.

### Risk-based review requirements

| Change type | Minimum review |
|-------------|----------------|
| Documentation only | 1 cross-model approve |
| Code changes | 1 cross-model approve |
| Gate or policy changes | 1 cross-model approve + explicit acknowledgment of policy impact |
| Security-sensitive | 1 cross-model approve + human confirmation recommended |

### After merge

1. Verify the linked issue or issues are closed.
2. If manual closure is needed, use the closure comment format above.
3. Post a verification note if the auto-close path does not carry enough evidence context.

---

## Integration with Shiplog Phases

### Phase 5 (PR-as-Timeline)

Before creating a PR, check:

- Is the author identity known?
- Who will review?

After creating a PR:

- If cross-model review is available, request it immediately.
- If not, generate the review contract and inform the user.

### Phase 4 (Commit-with-Context)

Commit context comments do not require cross-model review. The review gate applies at the PR level.

### Issue closure

When a PR merges and auto-closes issues via `Closes #N`:

- the merged PR is the evidence
- no extra closure comment is needed if the PR body already makes the evidence trail clear
- if the relationship between the PR and the issue is non-obvious, add a closure comment

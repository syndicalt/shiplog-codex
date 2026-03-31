---
name: brainstorm
description: "Phase 1: Capture brainstorming output as a GitHub issue with structured tasks, alternatives, and verification status."
---

# Plan Capture (Phase 1)

<!-- routing: tier-1, plan -->
<!-- cross-cutting: model-routing.md (Step 0), signing.md, labels.md -->

0. **Routing check.** Run the phase entry check from `model-routing.md`. For substantial planning work in Codex, call `update_plan` before creating artifacts.

1. **Run the brainstorm.** Brainstorm inline by default. If equivalent helper skills are installed, they are optional conveniences, not required dependencies.

2. **Capture as GitHub Issue (Full Mode).** Before the first labeled create in a repo, bootstrap the **shiplog** labels per `labels.md`. Create the issue with `shiplog/plan` already applied. Sign the issue body per `signing.md`.

   Before writing the final issue body, classify factual claims:
   - **Internal claims** about this repository's code, tests, configuration, or committed docs can be verified from the repo itself.
   - **External claims** about third-party tools, URLs, APIs, platform capabilities, or hosted services must be verified against primary sources before they are stated as facts.
   - If an external claim cannot be verified yet, keep it marked as `[unverified]` and treat it as a hypothesis, not settled input.

3. **Quiet Mode: defer capture.** Do not create the `--log` PR yet. The feature branch does not exist until branch setup. Save the brainstorm content locally and use it as the opening entry when the `--log` PR is created.

4. **Store in the knowledge graph.** If no dedicated memory backend exists, the issue itself is the durable record. Optional repo-local notes in `.shiplog/` are fine when they help recovery.

5. **Transition.** Proceed to the branch phase if the user wants to start implementation.

---

## Issue Template (Full Mode)

```bash
gh issue create \
  --label "shiplog/plan" \
  --title "[shiplog/plan] Brief title describing the work" \
  --body-file <temp-file>
```

Issue body content:

```markdown
<!-- shiplog:
kind: state
status: open
phase: 1
updated_at: <ISO_TIMESTAMP>
-->

## Context

[1-3 sentences: what problem are we solving and why now]

## Design Summary

[Key decisions from the brainstorm - 3-5 bullet points]

## Approach

[The chosen approach with brief rationale]

## Alternatives Considered

- **Alternative A**: [why not chosen]
- **Alternative B**: [why not chosen]

## Sources and Verification Status

Use this section when the issue includes external factual claims. If the issue is only
about this repository's own code or docs, cite repo paths inline where useful and omit
this section if it adds no value.

- `[verified]` Claim: [external factual statement that affects the issue]
  - **Source:** [primary source URL or official repository link]
  - **Checked:** [what the source confirms]
- `[unverified]` Claim or hypothesis: [idea worth preserving, but not yet confirmed]
  - **Why unverified:** [missing source, conflicting reports, or pending experiment]
  - **Next step:** [what must be checked before this can become a requirement]

## Tasks

Each task is self-contained. A fast execution model should be able to implement any
`[tier-3]` task using only the information in that block, without reading the rest
of the issue.

- [ ] **T1: [Short title]** `[tier-3]`
  - **What:** [1-2 sentences, exactly what to do]
  - **Files:** `path/to/file.ts` (create|modify|delete)
  - **Allowed to change:** [`path/to/file.ts`; specific symbols or sections if needed]
  - **Must not change:** [files, APIs, behavior, or decisions that are out of scope]
  - **Forbidden judgment calls:** [choices the implementer must not make locally]
  - **Stop and ask if:** [condition that would require widening scope]
  - **Verification:** [command, check, or evidence required before claiming completion]
  - **Return artifact:** [diff, comment, checklist update, verification note]
  - **Decision budget:** `none`
  - **Accept when:** [concrete, testable acceptance criteria]
  - **Context:** [any non-obvious background the implementer needs]

- [ ] **T2: [Short title]** `[tier-1]`
  - **What:** [1-2 sentences]
  - **Files:** `path/to/file.ts`
  - **Decision budget:** [what judgment this task is allowed to exercise]
  - **Accept when:** [criteria]
  - **Why tier-1:** [why this needs reasoning]

Task ID rules:
- Every task carries a local ID: `T1`, `T2`, `T3`, etc.
- IDs are scoped to the issue. `T1` in issue #42 becomes `#42/T1` globally.
- Commits referencing a task use: `<type>(#<id>/<Tn>): <msg>`.
- `[tier-3]` tasks must be executable without creative judgment.
- `[tier-1]` tasks require reasoning. Include **Why tier-1**.

## Open Questions

- [Any unresolved questions]

---
Authored-by: <family>/<version> (<tool>)
*Captain's log entry created by [shiplog](https://github.com/devallibus/shiplog)*
```

On PowerShell, prefer the `--body-file` temp-file pattern from `shell-portability.md`.
Before the first labeled create in a repository, run the bootstrap commands from `labels.md`.

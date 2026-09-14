---
name: simplify
description: Simplify a recent code change with three parallel read-only reviews for code quality, performance, and reuse/repository patterns, then apply only high-confidence behavior-preserving cleanup. Use after implementing or fixing code when the user wants a diff, commit, branch, or working tree simplified without broad redesign.
---

# Simplify

Reduce accidental complexity in the requested change without changing intended behavior.

This skill is an orchestrator. When collaboration/subagent tools are available, you MUST run exactly three independent read-only reviews in parallel:

1. **Code quality** — unnecessary complexity, redundant comments/state/abstractions, dead compatibility code, avoidable control-flow noise.
2. **Performance** — meaningful waste that can be removed by simplification; it is valid to return no findings.
3. **Reuse / repository patterns** — existing helpers, components, conventions, tests, docs ownership, architecture/ADR constraints, or sibling implementations that the change should reuse or follow.

Read [references/reviewers.md](references/reviewers.md) before delegating. Read [references/reconciliation.md](references/reconciliation.md) before applying any finding.

## Workflow

### 1. Resolve scope before reviewing

- Inspect repository instructions and relevant `AGENTS.md` files first.
- If the user supplied an explicit commit/range/branch/path scope, inspect it and determine the intended implementation surface.
- Otherwise, inspect the current working-tree/index diff and use the relevant local change as the default scope.
- Inspect `git status` and explicitly protect unrelated uncommitted changes. Do not revert, format, rewrite, or include them accidentally.
- Scope is semantic, not merely textual: a docs-only commit may describe implementation still present in the working tree, while unrelated local edits must remain excluded.
- Preserve every explicit user constraint, including requests such as “do not commit” or limits on tests.

### 2. Understand the change

Before spawning reviewers, form a compact change model:

- what changed and why;
- relevant files, consumers, call paths, hot paths, or runtime surfaces;
- nearby repository patterns worth comparing;
- known exclusions and unrelated worktree changes;
- product/design decisions that must not be silently made.

Do not give all three reviewers the same generic dump. Build a role-specific brief for each reviewer from this model.

### 3. Run three read-only reviewers in parallel

When subagents are available, explicitly delegate all three roles in parallel and wait for all results. The skill itself authorizes this delegation.

Each reviewer must be told that it is read-only and must not edit files, run formatters, create worktrees, commit, or otherwise mutate the repository. It may inspect/search/read as needed.

If subagents are unavailable, perform the same three review passes sequentially and state that fallback in the final summary.

A reviewer may return **no material findings**. Never require busywork so that every reviewer contributes a change.

### 4. Reconcile evidence, do not vote

Aggregate and deduplicate findings. When reviewers disagree, or when a suggestion could regress behavior, inspect the actual code and repository precedent before deciding.

Do not accept a finding merely because it is high-confidence or repeated. Prefer evidence from:

- current behavior and the reason for the original change;
- repository-local conventions and analogous implementations;
- architecture/ADR/test constraints;
- direct verification of the affected path.

### 5. Apply only surgical simplifications

A finding is a good candidate to apply when it is:

- correct and supported by evidence;
- inside the resolved scope;
- behavior-preserving;
- small and targeted;
- a net reduction in total complexity;
- consistent with established repository patterns;
- safe without an unresolved product/design decision.

Do **not** optimize for the number of findings fixed. Do not add a helper, abstraction, component, branch, cache, memoization layer, or design token merely to eliminate tiny duplication or theoretical cost. Small duplication can be simpler than a new abstraction.

Skip or defer findings that are valid but larger than the simplify pass, speculative, product-sensitive, novel relative to the repository, or low-payoff.

### 6. Verify proportionally

Run the smallest meaningful checks appropriate to the touched code and repository rules: focused tests, typecheck, lint, build, or another relevant check. Do not run forbidden or maintainer-owned validation.

Re-inspect the final diff to ensure unrelated changes were not touched and the simplification did not undo the original fix/feature.

### 7. Report

Summarize:

- **Simplified / Fixed** — what changed and why it is simpler;
- **Skipped / Deferred** — worthwhile findings not applied, with the reason;
- **Verification** — checks run, or why they were not run;
- **Workspace safety** — note preserved unrelated changes when relevant.

Keep the report concise and distinguish implemented cleanup from recommendations.
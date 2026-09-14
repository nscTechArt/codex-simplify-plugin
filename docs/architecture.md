# Architecture

`codex-simplify-plugin` is implemented as a Codex Skill whose entrypoint orchestrates three specialized read-only reviews and keeps all code mutation in the parent agent.

The design goal is not “make code shorter.” It is **reduce accidental complexity while preserving the intent and behavior of the target change**.

## Pipeline

```text
Invocation / user constraints
        |
        v
Scope discovery
(git diff / commit / branch / status)
        |
        v
Semantic change understanding
        |
        v
Reviewer-specific context builder
        |
   +----+----------------+----------------+
   |                     |                |
   v                     v                v
Quality reviewer     Performance       Reuse / repo pattern
READ ONLY            reviewer          reviewer
                     READ ONLY         READ ONLY
   |                     |                |
   +---------------------+----------------+
                         |
                         v
Findings aggregation + deduplication
                         |
                         v
Evidence reconciliation
(conflicts -> inspect code/repo precedent)
                         |
                         v
Surgical parent edits
                         |
                         v
Adaptive verification
                         |
                         v
Fixed / Skipped / Verification report
```

## Why the parent owns editing

The three reviewers intentionally optimize for different signals and can disagree. For example, a quality reviewer may prefer removing a helper, a performance reviewer may prefer a specialized fast path, and a reuse reviewer may find an existing repository pattern with different semantics.

If reviewers edit independently, those recommendations can collide or create churn. The parent instead receives read-only evidence, resolves contradictions, then makes one coherent patch.

## Scope is semantic

The initial git object is only the starting point for scope resolution.

Examples:

- With no explicit argument, the relevant working-tree/index diff is usually the default target.
- With a commit, inspect the commit and surrounding working-tree state.
- A docs-only commit may describe implementation that is still uncommitted and clearly belongs to the same requested change.
- Unrelated local edits must remain excluded even when they are physically near the target files.

The parent should state exclusions in reviewer briefs so subagents do not “clean up” unrelated work.

## Reviewer-specific context

The parent should not blindly send the same prompt to every reviewer.

### Quality context

Usually benefits from the concrete diff, surrounding code, and a precise explanation of why the change exists.

### Performance context

Usually benefits from hot/cold path identification, runtime scale, consumers/callers, and domain-specific costs. Tiny presentational UI changes may need almost no performance review; rendering code may need pass/sample/variant reasoning.

### Reuse context

Usually benefits from explicit repository search targets: sibling implementations, helpers, style constants, test patterns, architecture docs, ADRs, lifecycle ownership, or documentation ownership.

## Decision objective

The reconciliation objective is approximately:

```text
minimize(total accidental complexity)
subject to:
  preserve intended behavior
  preserve user constraints
  stay inside resolved scope
  preserve unrelated workspace changes
  prefer repository-local precedent
  avoid unresolved product/design decisions
```

This is deliberately different from maximizing DRYness, maximizing performance, or maximizing the number of findings fixed.

## Verification

Verification is adaptive rather than hard-coded.

Use the smallest checks that materially validate the edited surface and are permitted by repository instructions. Examples include a focused unit test, typecheck, lint on touched files, build target, or a visual/runtime check. Some repositories explicitly reserve compilation/testing for maintainers; in that case report that the check was not run rather than violating the repository convention.

## Codex implementation notes

Current Codex Skills use a `SKILL.md` entrypoint with optional supporting `references/`. Codex also supports subagent delegation when the user or selected skill explicitly requests it. The orchestrator therefore makes the three-reviewer delegation an explicit requirement when collaboration tools are available, while retaining a sequential fallback for environments without subagents.

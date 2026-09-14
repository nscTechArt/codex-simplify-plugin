# Cursor `/simplify` behavioral observations

This document records black-box behavior observed from user-supplied Cursor sessions. It does **not** claim access to Cursor source code, private implementation details, or exact internal prompt templates.

The purpose is to separate what was directly observed from what this project infers for an independent Codex implementation.

## Observed directly

Across multiple unrelated projects (Unity shaders/assets, C# RenderGraph ownership, React/Tailwind UI), Cursor's `/simplify` showed a stable outer workflow:

1. inspect the requested git scope or local changes;
2. launch three read-only reviewers in parallel:
   - Code Quality;
   - Performance;
   - Reuse / repository patterns;
3. wait for reviewer results;
4. have the parent agent decide which findings to apply;
5. apply targeted cleanup in the parent;
6. run appropriate lightweight verification when repository rules allow it;
7. report applied and skipped/deferred suggestions.

The reviewer instructions consistently prohibited editing, formatting, worktree creation, and committing.

Reviewers were allowed to return no meaningful findings. In a small React/Tailwind change, the performance reviewer explicitly concluded that there was nothing worth optimizing. In a RenderGraph change, performance likewise reported no material issue.

The parent did not blindly apply reviewer output. It re-checked evidence when a recommendation risked regression or when reviewers disagreed.

## Observed role behavior

### Code Quality

Repeated themes included:

- low-information comments;
- one-off helpers / needless wrappers;
- unnecessary abstraction;
- nullable/optional proliferation;
- broad exception handling / duplicated cleanup ownership;
- weak type escape hatches;
- duplicated/derived state;
- dead/compatibility/migration residue;
- duplicated documentation or bookkeeping;
- unrelated drive-by changes;
- stale references left behind by consolidation/retirement.

The role focused on accidental complexity rather than turning into a general bug/security review.

### Performance

The performance brief changed substantially by domain.

Observed domain-specific examples included:

- ordinary hot-path concerns such as repeated expensive work, allocations, blocking work, logging, and N+1 I/O;
- graphics concerns such as redundant material evaluation, texture samples, shader variants, branches, and stale heavy paths;
- web/UI reasoning that deliberately rejected memoization/caching for tiny cheap components at small scale.

This suggests “no material performance issue” is a first-class valid result.

### Reuse / repository patterns

This role searched beyond changed lines and used repository precedent as evidence. Observed targets included:

- existing helpers and wrappers;
- sibling shaders / renderer patterns;
- class-string constants and `cn()` composition conventions;
- test harness patterns;
- documentation ownership;
- architecture documentation and ADR constraints.

The reviewer also produced anti-reuse findings: cases where a tempting existing helper/component had different semantics and should **not** be unified.

## Observed parent behavior

### Semantic scope resolution

Cursor did not treat an explicit commit hash as a purely mechanical `git show` boundary in every case. One session detected that the target commit mainly contained docs while the implementation described by those docs remained in related uncommitted files, and reviewed the implementation surface. In another session it explicitly excluded an unrelated uncommitted shader change.

The consistent principle appeared to be: resolve the intended change surface while preserving unrelated workspace state.

### Conflict resolution

In one shader session, quality and performance recommendations disagreed about keyword retention. The parent inspected sibling implementations and render-pass behavior before deciding. It did not resolve the conflict by majority vote.

### Anti-abstraction / payoff judgment

Across sessions, Cursor repeatedly rejected plausible but low-payoff abstraction:

- did not invent a graph-reset wrapper when no existing helper existed;
- did not extract a helper for tiny weathering duplication;
- chose a shared Tailwind class constant instead of creating a new base `AccentPill` component;
- kept separate domain components when an ADR said their semantics differ.

The recurring pattern was to minimize **total** complexity, not maximize DRYness.

### Safe targeted edits

Findings were often reported but not applied when they were:

- larger than the simplify pass;
- dependent on product/visual/API decisions;
- speculative or low-payoff;
- a performance optimization that introduced new branching/novelty;
- outside the resolved scope.

## Strongly inferred

The exact reviewer prompts appear to be dynamically composed rather than three giant static strings.

Evidence for this inference:

- all sessions kept the same three reviewer roles and read-only contract;
- reviewer briefs contained highly task-specific summaries, hot paths, consumers, sibling files, ADRs, shader concerns, or UI scale;
- the three reviewers in the same session were not given identical context.

A plausible model is:

```text
stable /simplify orchestration policy
        |
        v
parent understands the change
        |
        +--> quality-specific brief
        +--> performance-specific brief
        +--> reuse/pattern-specific brief
```

This repository adopts that architecture rather than copying captured prompt text.

## Open questions

The observations do not establish:

- the exact Cursor system/orchestrator prompt;
- whether all three roles are hard-coded at the product layer or selected by a higher-level policy;
- whether Cursor applies a numeric confidence threshold;
- the exact mechanism used to sandbox reviewer writes;
- how scope resolution behaves for merges, staged-only changes, or very large branch ranges;
- whether reviewer briefs are generated entirely by the parent model or partly by fixed templates.

Future behavioral samples should update this document only when they demonstrate a repeatable decision rule rather than a one-off project preference.

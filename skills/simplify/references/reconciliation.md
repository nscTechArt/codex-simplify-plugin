# Reconciliation policy

The parent/orchestrator owns all edits. Reviewer output is evidence, not an instruction to mutate code.

## First pass: normalize findings

After all three reviewers finish:

1. group findings that refer to the same underlying issue;
2. merge supporting evidence without inflating severity;
3. separate mutually reinforcing findings from genuinely conflicting recommendations;
4. mark findings that require product/design judgment;
5. mark findings that are outside the resolved simplify scope.

Do not reward a suggestion merely because two reviewers phrased it similarly. Multiple reviewers may be reacting to the same superficial symptom.

## Conflict handling

When reviewers disagree, do **not** choose by majority vote or confidence score alone.

Inspect additional evidence:

- the behavior the original change was intended to preserve/fix;
- the concrete call/render/data path;
- analogous sibling implementations;
- tests and repository-local conventions;
- architecture docs / ADRs;
- rendered/runtime behavior when inexpensive to verify.

Prefer the option that preserves semantics while reducing total complexity and novelty.

Example conflict shape:

- quality reviewer says a flag/helper/comment looks redundant;
- performance reviewer says removing it reduces variants/work;
- reuse reviewer finds an established sibling pattern that retains or drops it only in specific passes.

Resolve from the actual semantics and precedent, not from the number of votes.

## Acceptance filter

Before applying a finding, ask:

1. **Correct?** Is the premise true in the current tree?
2. **In scope?** Does it belong to the resolved change surface rather than nearby cleanup debt?
3. **Behavior-preserving?** Could it undo the original bug fix/feature or change public behavior?
4. **Surgical?** Can it be implemented without broad churn?
5. **Net simpler?** Does it reduce more complexity than it introduces?
6. **Repository-aligned?** Does it follow established local patterns, or is any novelty justified?
7. **Decision-safe?** Does it avoid making an unresolved product/design/visual/API choice?
8. **Worth it?** Is the payoff material enough for a simplify pass?

A finding can be valid and still be deferred.

## Anti-abstraction rule

Do not maximize DRYness.

Reject or defer a new helper/component/wrapper/cache/branch/token/variant when:

- it exists only to remove tiny local duplication;
- it conflates distinct domain concepts;
- there is no repository precedent and the gain is marginal;
- it makes control flow or ownership harder to see;
- it adds configuration or indirection that the current change does not need.

Prefer the smallest reusable unit that already matches repository practice. Sometimes that is a shared constant rather than a base component; sometimes it is leaving two short lines duplicated.

## Performance rule

Do not turn `/simplify` into `/optimize`.

A performance finding should normally be applied only when the optimization is also a simplification or an obvious removal of wasted work. Skip speculative micro-optimizations, new caches, extra branch machinery, or architecture changes unless the evidence and payoff are unusually clear.

## Workspace safety

Before edits and again before reporting:

- compare `git status` / diff with the resolved scope;
- preserve unrelated local modifications;
- do not run broad formatters that rewrite excluded files;
- do not commit unless the user explicitly asked for a commit;
- do not revert user changes to make the simplify patch cleaner.

## Final review

After applying accepted findings:

- inspect the final diff for accidental scope expansion;
- confirm the original behavior/fix remains intact;
- run proportionate checks permitted by repository/user constraints;
- report skipped findings separately from applied cleanup.
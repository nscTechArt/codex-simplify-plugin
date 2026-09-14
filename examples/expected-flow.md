# Expected flow

This example is intentionally generic. It demonstrates the orchestration shape rather than prescribing exact wording.

## User request

```text
$simplify HEAD~1
```

## Parent: scope discovery

```text
Target: HEAD~1
Working tree: one unrelated local modification in docs/notes.md; exclude and preserve it.
Change intent: remove duplicate request validation by moving ownership into RequestParser.
Relevant implementation: RequestParser, ApiHandler, parser tests.
```

## Parallel reviewer briefs

### Code Quality

```text
READ ONLY.
Review the resolved change for accidental complexity.
Focus on duplicated ownership/comments, redundant wrappers, unnecessary nesting,
and compatibility paths made obsolete by the ownership move.
Return only actionable findings inside scope; no edits.
```

### Performance

```text
READ ONLY.
The changed path runs once per request. Check whether the ownership move leaves
repeated parsing, allocation, logging, blocking work, or other material waste.
Do not propose caching/memoization without evidence. “No material issue” is valid.
No edits.
```

### Reuse / repository patterns

```text
READ ONLY.
Search sibling request parsers/handlers, test helpers, and architecture docs for the
established validation-ownership pattern. Identify existing code/patterns to reuse,
and call out tempting abstractions that should not be introduced. No edits.
```

The parent launches all three concurrently when subagent tools are available, then waits for all results.

## Parent: reconciliation

Suppose the reviewers return:

```text
Quality: remove an ApiHandler comment that restates RequestParser's return contract.
Performance: no material findings.
Reuse: parser tests already have a shared invalid-request table; reuse it instead of adding a second fixture helper.
```

The parent verifies both findings against the tree, then applies only those two small changes.

If a reviewer instead suggests a larger parser abstraction, the parent should defer it when the current change does not justify the extra indirection.

## Verification

```text
Run focused parser tests and lint/typecheck for touched files, if repository rules permit.
Re-check git diff/status and confirm docs/notes.md is untouched.
```

## Final report

```text
Simplified
- Removed duplicate ownership comment in ApiHandler.
- Reused the existing invalid-request test table instead of adding a parallel helper.

Skipped
- Broader parser abstraction: larger than this simplify pass and adds indirection.

Verification
- Focused parser tests passed.
- Lint/typecheck passed for touched files.
- Unrelated docs/notes.md change was preserved.
```

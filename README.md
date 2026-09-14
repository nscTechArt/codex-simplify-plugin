# codex-simplify-plugin

An independent Codex implementation of a multi-agent **simplify pass**: inspect a recent change, run three read-only reviews in parallel (code quality, performance, and reuse/repository patterns), reconcile the evidence, then apply only targeted behavior-preserving cleanup.

The repository name says “plugin”; the v0.1 implementation is packaged as a **Codex Skill**.

## Status

**v0.1 / behavioral prototype**

The workflow is based on black-box observations of Cursor `/simplify` sessions across rendering, C#, and React/Tailwind projects. It does not contain Cursor source code or claim exact prompt/source compatibility.

See:

- [`docs/architecture.md`](docs/architecture.md) — implementation model
- [`docs/cursor-observations.md`](docs/cursor-observations.md) — observed vs inferred behavior
- [`examples/expected-flow.md`](examples/expected-flow.md) — generic end-to-end example

## Core behavior

```text
scope discovery + workspace guard
             |
             v
semantic change understanding
             |
             v
reviewer-specific briefs
             |
      +------+-------+
      |      |       |
      v      v       v
  Quality   Perf    Reuse
  READONLY READONLY READONLY
      |      |       |
      +------+-------+
             |
             v
evidence reconciliation
             |
             v
surgical parent edits
             |
             v
adaptive verification
```

The key rule is **minimize total accidental complexity**, not “make the code as DRY/short/fast as possible.” A reviewer may return no findings, and a valid finding may still be deferred when it adds abstraction, expands scope, risks behavior, or requires a product decision.

## Install

Copy or symlink the skill directory into your Codex skills directory:

```text
skills/simplify/
```

Target location:

```text
$CODEX_HOME/skills/simplify
```

or, when `CODEX_HOME` is unset:

```text
~/.codex/skills/simplify
```

Codex Skills use a `SKILL.md` entrypoint with optional supporting resources. This repository keeps detailed reviewer and reconciliation policy in `references/` so the entrypoint stays focused.

## Use

Explicit invocation in Codex:

```text
$simplify
```

or with a scope in the request, for example:

```text
Use $simplify on commit abc1234. Do not commit.
```

```text
Use $simplify on origin/main..HEAD.
```

With no explicit scope, the skill inspects the relevant working-tree/index changes and protects unrelated local modifications.

## Skill layout

```text
skills/simplify/
├── SKILL.md
└── references/
    ├── reviewers.md
    └── reconciliation.md
```

### Reviewer roles

- **Code Quality** — accidental complexity, redundant comments/state/abstractions, stale or compatibility residue.
- **Performance** — material waste removable without turning the pass into a redesign; `none` is a valid result.
- **Reuse / Repository Patterns** — existing helpers, sibling patterns, tests, docs ownership, architecture/ADR constraints, plus anti-reuse findings when unification would be wrong.

All three are instructed to remain read-only. The parent agent owns reconciliation and editing.

## Design references

The packaging follows the current Codex Skill model and its `SKILL.md` + optional `references/` structure. OpenAI's own Codex repository also contains an orchestrator-style code-review skill that explicitly delegates review work to subagents; this project applies that general orchestration pattern to simplification rather than copying its review policy.

## Non-goals

- Reproduce proprietary Cursor source code or hidden prompts verbatim.
- Turn every simplify pass into a broad refactor.
- Force every reviewer to find a problem.
- Introduce abstractions solely to remove tiny duplication.
- Optimize unrelated pre-existing code.
- Touch unrelated working-tree changes.

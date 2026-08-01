---
title: "Org-Wide Governance Adoption"
doc_type: manual
description: "Runbook for the git-governance and docs-governance plugins across licorsy repositories: which repository owns which part of the automation, what a compliant repository looks like, how to bring an existing repository up to standard, and how a new repository inherits it."
status: active
version: "1.0.0"
created: 2026-08-01
updated: 2026-08-01
language: en
id: org-governance-adoption
owner: Alexandre Clemente
tags: [governance, automation, plugins, adoption, runbook]
related: [organizational-blueprint, repository-classification, claude-instructions, engineering-standards]
---

# Org-Wide Governance Adoption

## Why this exists

The same defect recurred across six or more repositories: a normative fact was
restated by hand in several files, nothing checked mechanically that the copies
stayed identical, and each round of LLM auditing rediscovered the same class of
problem. This repository alone needed seven fix-and-verify rounds before one
such audit converged.

The answer is not more auditing. It is making the facts machine-checkable, so
drift fails a check instead of waiting for someone to notice. Two plugins do
that, and this runbook explains which one owns what — so the next person, or
the next agent session, does not have to work it out again.

A live example of exactly the defect this guards against: when this repository
was scaffolded on 2026-08-01, the cached `git-governance` plugin was stale
against its own source and wrote a `CLAUDE.md` claiming `develop` was not
protected server-side. The actual ruleset covered `develop`, `staging`, and
`main`. The stale copy was replaced with the live source. See "Keeping the
plugin cache honest" below.

## Division of responsibilities

Four repositories, four distinct jobs. The rule is that a repository may
consume a capability without embedding its logic — see
[REPOSITORY-CLASSIFICATION.md](../REPOSITORY-CLASSIFICATION.md) for the full
ownership matrix.

**`git-governance`** owns the branch-naming taxonomy, commit-message format,
and merge-permission matrix, plus the slash commands and the
`git-governance-advisor` subagent. Its `scripts/init-governance.sh` scaffolds
`CLAUDE.md`, `.pre-commit-config.yaml`, and `.github/workflows/pr-checks.yml`
into a target repository. **It never overwrites an existing file** — it skips
with a warning. In a repository created from `ai-assisted-sdd-template`, the
template's own `CLAUDE.md` therefore wins; the `git-governance` copy is only
the fallback for repositories that have none.

**`docs-governance`** owns mechanical documentation consistency: a rule engine
driven entirely by the consuming repository's `.docgov.config.js`, plus two
read-only review subagents for the semantic questions a script cannot decide.
The engine holds no organization-specific knowledge — all scoping is data in
the config.

**`ai-assisted-sdd-template`** owns the project delivery method: the SDD
operating model, project artifact structure, and bootstrap. It correctly
references `git-governance-advisor` rather than restating the branch taxonomy.

**`platform-workflows`** owns reusable CI workflows called via `workflow_call`
and pinned to the floating `v1` tag. **Reusable CI belongs there, not in
`.github`** — this repository hosts no reusable workflow of its own.

## What "compliant" means

A compliant repository carries five artifacts:

| Artifact | Written by | Purpose |
| --- | --- | --- |
| `CLAUDE.md` | `init-governance.sh` | Branch, commit, and merge policy in force |
| `.pre-commit-config.yaml` | `init-governance.sh` | The primary local gate |
| `.github/workflows/pr-checks.yml` | `init-governance.sh` | Remote gate at promotion points only |
| `.docgov.config.js` | `docgov init`, then edited | Which documents are governed, and how |
| `.claude/settings.json` | by hand | Declares `enabledPlugins` so plugin availability belongs to the repo |

`enabledPlugins` is an **object with boolean values**, not an array, and each
key is `plugin@marketplace`:

```json
{
  "enabledPlugins": {
    "git-governance@git-governance": true,
    "docs-governance@docs-governance": true
  }
}
```

## Documentation metadata

Every tracked Markdown file carries the frontmatter schema declared in
`.docgov.config.js`, with a fixed set of exceptions. A file is excluded when
its frontmatter is already a functional contract owned by another system, when
another system renders or injects its raw content verbatim, or when the file is
not part of the tracked corpus at all:

| Excluded | Reason |
| --- | --- |
| `README.md` | Rendered as the public repository or organization profile; GitHub renders frontmatter as a visible table |
| `.github/PULL_REQUEST_TEMPLATE.md` | Injected verbatim into every pull request body |
| `.github/ISSUE_TEMPLATE/*.md` | Carries GitHub-mandated template frontmatter |
| `agents/*.md`, `commands/*.md`, `.claude/agents/*.md`, `.claude/commands/*.md` | Claude Code plugin manifests; frontmatter is the routing contract |
| `local-notes/**` | Git-untracked reference material, outside the governed corpus entirely — excluded from `internal-links` by directory name rather than from `frontmatter` by pattern |

Every exclusion is recorded with its reason in the `.docgov.config.js` comment.
A silent exclusion is how scope drift starts.

The parser is deliberately naive — `key: value`, first occurrence wins. Four
consequences are easy to get wrong:

- Quotes are **not** stripped, so `status: "active"` fails the enum check.
  Status and date fields must be unquoted.
- Date fields are regex-checked as `YYYY-MM-DD` against the raw value, so a
  quoted date fails.
- YAML **block lists do not parse**. `related:` must use the inline form
  `[a, b, c]`, and `owner:` must be a scalar.
- `related:` entries must resolve to an `id:` declared by some other in-scope
  document, or the rule fails.

## Bringing an existing repository up to standard

1. Run `/git-check`. It audits and, with confirmation, runs
   `init-governance.sh` for the three files it owns.
2. Run `pre-commit install && pre-commit install --hook-type commit-msg`. Both
   are required — they wire different hook stages.
3. Add frontmatter to every Markdown file outside the exceptions above.
4. Run `docgov init`, then edit `.docgov.config.js` by hand: set the
   frontmatter scope and required fields, point `changelog-retention` at the
   same corpus as `frontmatter` (the rule is a no-op for files without the
   marker, so a wide scope imposes nothing and starts checking automatically
   if a changelog is ever added), and record every exclusion's reason in a
   comment.
5. Add `.claude/settings.json` with `enabledPlugins`.
6. Verify: `/git-check` reports Compliant, and `docgov check` passes.

Configuration declares **data, never logic**. If a check does not exist, it
belongs in the engine, not in a repository's config — otherwise the engine gets
forked by config and duplication returns through the side door.

## How a new repository is born compliant

Create it from `ai-assisted-sdd-template`, which ships the SDD scaffold and
already delegates git operations to `git-governance-advisor`.

**Known gap:** as of 2026-08-01 the template ships no `.claude/settings.json`,
so a repository created from it still needs that file added by hand. The same
gap exists in the other three repositories named above — `git-governance`,
`docs-governance`, and `platform-workflows`. `.github` is currently the only
repository that has closed it. Tracked in
[REPOSITORY-CLASSIFICATION.md](../REPOSITORY-CLASSIFICATION.md) under "Known
gaps".

## Keeping the plugin cache honest

`init-governance.sh` copies from the **installed plugin cache**, not from the
plugin's source repository. If the plugin has merged fixes without a version
bump and re-release, the cache silently scaffolds stale policy — which is
exactly what happened here on 2026-08-01.

Before trusting scaffolded output in a new repository, diff it against the live
plugin source. Where they differ, the live source wins, and the divergence
should be reported upstream so the plugin gets a proper release.

## Related work

Open gaps — in other repositories and in this one — are recorded in
[REPOSITORY-CLASSIFICATION.md](../REPOSITORY-CLASSIFICATION.md) under "Known
gaps", not here. This runbook describes the standard; it does not track the
backlog against it.

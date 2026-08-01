---
title: "Org-Wide Governance Adoption"
doc_type: manual
description: "Runbook for the git-governance and docs-governance plugins across licorsy repositories: which repository owns which part of the automation, what a compliant repository looks like in-repo and on GitHub, how to bring an existing repository up to standard, and how a new repository inherits it."
status: active
version: "1.3.0"
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

Compliance is now measurable rather than asserted:
`platform-workflows`' `governance-compliance.yml` checks all five as a reusable
workflow, advisory by default and failing with `strict: true`. It is a presence
check on purpose — what each file must *say* is enforced by `pre-commit` and
`docgov` against the repository's own config, and re-checking that in a workflow
would fork the rules.

```yaml
jobs:
  governance-compliance:
    uses: licorsy/platform-workflows/.github/workflows/governance-compliance.yml@v1
```

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

### Server-side settings a compliant repository carries

The five artifacts above live in the repository. The settings below live on
GitHub, are applied by `git-governance`'s `scripts/setup-branch-protection.sh`,
and are stated here so a repository can be *verified* compliant rather than
assumed compliant after the script runs.

One ruleset per protected branch, each with `deletion`, `non_fast_forward`, and
a `pull_request` rule at `required_approving_review_count: 0` and
`bypass_actors: []`. Merge methods differ per branch:

| Target | Allowed merge methods |
| --- | --- |
| `develop` | merge commit, squash |
| `staging`, `main` | merge commit only |

Rebase-merge is disabled at the repository level, and `delete_branch_on_merge`
is enabled so work branches are cleaned up automatically. The protected branches
survive that setting: GitHub exempts protected branches from auto-delete, and
the `deletion` rule blocks it independently — which is what keeps `develop`, the
head branch of every `develop -> staging` promotion, from being deleted when a
promotion merges.

Per-branch merge methods are the point, not a detail. A squashed
`develop -> staging` merge rewrites the promoted commits, so `staging` stops
sharing history with `develop` and the next promotion re-conflicts on work
already merged. Restricting promotions to merge commits removes that failure
mode structurally rather than by convention.

**What these settings cannot do** is restrict *who* merges into `staging` or
`main`. GitHub cannot distinguish the owner from an agent using the owner's
token, and requiring an approving review would lock a solo maintainer out of
their own promotion branches, since GitHub forbids self-approval. That gate is
therefore behavioral — see
[AGENTS.md](../AGENTS.md), "Why the `staging`/`main` gate is behavioral, not
server-side". Making it server-side requires giving the agent a separate
identity (GitHub App or machine user); until that exists, raising the approval
count is a regression, not a hardening.

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
| `agents/*.md`, `commands/*.md`, `.claude/agents/*.md`, `.claude/commands/*.md` | Claude Code plugin manifests; frontmatter is the routing contract. Exempt from *this* schema, not from checking — see below |
| `CHANGELOG.md` | Follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), an external format standard that owns the file's structure |
| `local-notes/**` | Git-untracked reference material, outside the governed corpus entirely — excluded from `internal-links` by directory name rather than from `frontmatter` by pattern |

`CLAUDE.md` and `AGENTS.md` are **not** exceptions. They are repository entry
points, but no other system owns their frontmatter and nothing renders them
verbatim, so they carry the schema like any other document.

"Excluded" above means **excluded from the eight-field schema, not unchecked.**
The plugin-manifest row is the case where the distinction bites:
`git-governance` and `docs-governance` both put `agents/` and `commands/` in
`scope_dirs` and enforce `required: ['description']` on them, because
`description` is the one field Claude Code actually routes on. Reading that row
as "these directories are never validated" is wrong, and was wrong in this
register until 2026-08-01. A repository that ships plugin manifests should scope
them with the reduced schema rather than skipping them.

Every exclusion is recorded with its reason in the `.docgov.config.js` comment,
and the two copies of this register are pinned against each other by a `facts`
entry — correcting one alone will now fail `docgov check`. A silent exclusion is
how scope drift starts; an unpinned register is how the correction drifts back.

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
   same corpus as `frontmatter`, and record every exclusion's reason in a
   comment. The retention rule is a no-op for files without the marker, so a
   wide scope imposes nothing — but the `marker` must match the heading line
   **exactly** (the engine compares `line.trim() === marker`), so a document
   using a different wording is silently skipped rather than flagged. Confirm
   the count in `docgov check`'s output matches the number of documents that
   actually keep a changelog.
5. Add `.claude/settings.json` with `enabledPlugins`.
6. Verify: `/git-check` reports Compliant, and `docgov check` passes.

Configuration declares **data, never logic**. If a check does not exist, it
belongs in the engine, not in a repository's config — otherwise the engine gets
forked by config and duplication returns through the side door.

## How a new repository is born compliant

Create it from `ai-assisted-sdd-template`, which ships the SDD scaffold and
already delegates git operations to `git-governance-advisor`.

All five repositories — `.github`, `git-governance`, `docs-governance`,
`ai-assisted-sdd-template`, and `platform-workflows` — now ship
`.claude/settings.json` with both plugins declared, closed on 2026-08-01. A
repository created from the template therefore inherits the declaration rather
than needing it added by hand.

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

---
title: "Repository Classification"
doc_type: governance
description: "The four Licorsy repository categories, what each repository owns and must not own, the single-owner matrix that prevents duplicated policy, and the known coupling gaps currently tracked against that model."
status: active
version: "1.0.0"
created: 2026-08-01
updated: 2026-08-01
language: en
id: repository-classification
owner: Alexandre Clemente
tags: [repositories, ownership, decoupling, platform, governance]
related: [organizational-blueprint, architecture-principles, engineering-standards, security-baseline]
---

# Repository Classification

Which repository owns what, and — just as important — what each one must not
own. This is an entry point:
[`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md)
is the canonical source, and Sections 4 and 5 there are authoritative wherever
this file summarizes.

## Categories

### Organization repositories

**`.github`** (this repository) — organization profile, default community health
files, issue and pull request templates, and shared governance documents.

**`platform-workflows`** — reusable GitHub Actions workflows: CI, security
checks, docs governance orchestration, release automation, and deployment
building blocks.

### Platform capability repositories

**`git-governance`** — branch strategy, merge permissions, commit conventions,
and governance commands for Claude Code. Does not own organization-level
defaults or project architecture.

**`docs-governance`** — deterministic documentation-consistency checks,
doc-specific CI validation, and doc review subagents. It owns its own
composite action, which callers invoke as a step; what it does not own is
generic CI orchestration — deciding *when* checks run across plugins — or
business process content.

### Method repository

**`ai-assisted-sdd-template`** — the golden path for new projects: the SDD
operating model, project bootstrap, and project-level governance structure.

Must **not** own organization-wide issue/PR templates, organization-wide
community health files, reusable CI implementation logic that belongs to
`platform-workflows`, or plugin logic from `git-governance` /
`docs-governance`.

### Product repositories

Examples: `personal-os`, `rag-mcp-server`, and future internal or client
products. They implement business capabilities and consume the platform
capabilities above.

## Ownership matrix

| Concern | Single owner | Consumers |
| --- | --- | --- |
| Org profile and default community files | `.github` | all repositories |
| Reusable CI workflows | `platform-workflows` | all delivery repositories |
| Branch/merge/commit governance | `git-governance` | all repositories using Claude Code |
| Markdown/document consistency | `docs-governance` | repositories with governed docs |
| Project delivery method | `ai-assisted-sdd-template` | all new product repositories |
| Product-specific implementation | product repositories | project team |

## Separation rules

1. A repository may **consume** a capability without embedding its logic.
2. A repository may **reference** another repository in documentation without
   duplicating its policy.
3. Organization defaults belong in `.github`, not in every template.
4. CI workflow logic belongs in `platform-workflows`, not copied inline across
   repositories.
5. The template defines **how projects are structured**, not how every
   organization-level policy is implemented.
6. Plugins should remain portable and usable outside Licorsy.

## Portability status

Rule 6 is the blueprint's wording and is deliberately a "should" — portability
is a goal to be measured, not a gate that blocks a release. Verified state as
of 2026-08-01:

| Repository | Portable standalone? | Notes |
| --- | --- | --- |
| `git-governance` | Yes, with one caveat | The `pr-checks.yml` it scaffolds hardcodes `licorsy/docs-governance/action@v1`, but the step is gated so it stays inert without a local `.docgov.config.js`. A fork that also uses docs-governance would run Licorsy's action until it repoints that line |
| `docs-governance` | Yes | Engine is config-driven; all scoping comes from the consuming repository's `.docgov.config.js` |
| `ai-assisted-sdd-template` | Content yes, CI no | See coupling gaps below |
| `platform-workflows` | N/A | Organization-specific by design; it is the thing others point at |

## Known coupling gaps

Open issues against the portability goal and the ownership matrix. Recording
them here is deliberate: an undocumented gap gets rediscovered by every future
audit, which is the failure mode this repository exists to stop. **Fixing these
belongs to the owning repository, not to `.github`.**

1. **`ai-assisted-sdd-template` CI hard-couples to the Licorsy organization.**
   Its `.github/workflows/pr-checks.yml` calls
   `licorsy/platform-workflows/.github/workflows/ci-docs.yml@v1` and
   `ci-security.yml@v1` unconditionally, with no guard. A fork into another
   organization would run CI against Licorsy's repositories until those `uses:`
   lines are repointed. `git-governance` already demonstrates the fix: gate the
   step so it self-disables when the corresponding config is absent.

2. **Semantic doc-review duplication.**
   `ai-assisted-sdd-template/.claude/agents/doc-consistency-reviewer.md`
   duplicates `docs-governance/agents/doc-consistency-auditor.md` — same
   purpose, tools, model, and method. The template already consolidated the
   *mechanical* checks into the shared engine when it came online, but kept a
   parallel copy of the *semantic* reviewer. This is inconsistent with how the
   same template correctly delegates git operations to
   `git-governance-advisor` rather than restating the taxonomy.

3. **The scaffolded `CLAUDE.md` describes a `.docgov.config.js` this
   repository does not have.** Its "Documentation ownership" section cites
   `facts` and `fragment_sync` entries, and an orphan
   `<!-- fragment:branch-flow -->` marker with no counterpart, because the
   file is copied verbatim from `git-governance`, where those entries do
   exist. Correcting it locally would be undone by the next scaffold, so the
   fix belongs in the plugin source. Same root cause as the stale-cache
   incident recorded in
   [`docs/org-governance-adoption.md`](docs/org-governance-adoption.md).

4. **`.github/PULL_REQUEST_TEMPLATE.md` cites tooling absent here.** Its
   checklist points at `.github/scripts/doc-scope.js` and
   `documentation-metadata-standard.md`, neither of which exists in this
   repository, and omits the `version-bump` rule that will actually fail a
   PR. The file is in the frontmatter exceptions register and injected
   verbatim into every pull request, so nothing mechanical will catch it.

5. **`CONTRIBUTING.md` and `GOVERNANCE.md` still route document scope through
   `CATEGORY_DIRS`.** They describe the `docs-governance` scope mechanism as
   something that "may eventually" supersede it; in this repository
   `.docgov.config.js` already governs. Both predate this batch and were left
   unchanged apart from frontmatter.

6. **Four repositories still do not declare their plugins.**
   `git-governance`, `docs-governance`, `ai-assisted-sdd-template`, and
   `platform-workflows` ship no `.claude/settings.json` with `enabledPlugins`,
   so plugin availability there depends on each developer's local
   configuration rather than on the repository. `.github` closed this for
   itself on 2026-08-01 — see
   [`docs/org-governance-adoption.md`](docs/org-governance-adoption.md).

## Canonical source

[`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md),
Sections 4 and 5. Where this file and the blueprint disagree, the blueprint wins.

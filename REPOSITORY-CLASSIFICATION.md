---
title: "Repository Classification"
doc_type: governance
description: "The four Licorsy repository categories, what each repository owns and must not own, the single-owner matrix that prevents duplicated policy, the verified portability status of each platform repository, and the open gaps tracked against that model."
status: active
version: "1.5.0"
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

The organization's private product repositories, plus future internal or client
products. They implement business capabilities and consume the platform
capabilities above.

They are named here as a **class, not individually, and deliberately so.** This
repository is public and its `README.md` renders as the public organization
profile, while the product repositories are private. Naming them would publish
the organization's private project list — the same reason
`ai-assisted-sdd-template` runs a sanitization check over its public export.
Governance rules apply to the category; nothing in this file needs a specific
product's name to be actionable.

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
| `ai-assisted-sdd-template` | Content yes, CI depends on Licorsy | See known gaps below |
| `platform-workflows` | N/A | Organization-specific by design; it is the thing others point at |

## Known gaps

Open issues against this model and against the documents that describe it.
Recording them here is deliberate: an undocumented gap gets rediscovered by
every future audit, which is the failure mode this repository exists to stop.
Each entry names the repository that owns the fix. Items 1, 2, 8, 9, 10, 11, 12,
and 13 are open; items 3, 4, 5, 6, and 7 are closed — kept here with their
resolution, because a gap that vanishes without a record gets rediscovered as a
new finding by the next audit.

1. **`ai-assisted-sdd-template` CI depends on Licorsy's reusable workflows.**
   Its `.github/workflows/pr-checks.yml` calls
   `licorsy/platform-workflows/.github/workflows/ci-docs.yml@v1` and
   `ci-security.yml@v1` as unguarded jobs. A repository created from the
   template outside this organization inherits a dependency on workflow
   definitions Licorsy controls and could change or delete.

   Two corrections to how this was first recorded, both from reading the file
   rather than inferring from the `uses:` line. It does **not** run CI against
   Licorsy's repositories — a public reusable workflow executes in the
   *caller's* context, on the caller's runners. And the fix cannot be copied
   from `git-governance`: that guard uses `hashFiles()`, which is valid in a
   step-level `if:` but not in the job-level `if:` a reusable-workflow call
   requires. The template's own comment records that as tested, not assumed.
   A job-level `github.repository_owner` condition would work; whether a
   template *should* ship CI that self-disables for its adopters is a design
   question, not a defect to quietly patch.

2. **Overlap between the template's doc-consistency reviewer and
   `docs-governance`'s.** Both audit a document set for semantic drift with the
   same tools and model. On inspection the overlap is narrower than it looks:
   the template's `.claude/agents/doc-consistency-reviewer.md` is a thin
   dispatcher into `agents/doc-consistency.md`, a versioned document with
   `related:` edges into the SDD graph and framing tied to the phase model
   ("once per cycle close, Phase 8 — Maintenance"). `docs-governance`'s auditor
   is deliberately generic and repository-agnostic.

   Consolidating would delete method content and break traceability edges, so
   this is **recorded as accepted overlap, not scheduled for removal.** Revisit
   only if the two prompts start disagreeing about what a finding is.

3. ~~**The scaffolded `CLAUDE.md` cites `.docgov.config.js` entries this
   repository's config does not define.**~~ **Closed 2026-08-01.** The prose
   described `facts` and `fragment_sync` entries that did not exist here, and
   carried an inert `<!-- fragment:branch-flow -->` marker pair with no
   destination. Resolved from both ends: the policy moved to
   [`AGENTS.md`](AGENTS.md) (item 10), the orphan markers were dropped because
   this repository's `README.md` carries no branch-flow diagram to sync against,
   and `.docgov.config.js` now declares three real `facts` entries — so the
   prose describes enforcement that is actually running.

4. ~~**`.github/PULL_REQUEST_TEMPLATE.md` cites tooling absent here.**~~
   **Closed 2026-08-01.** The checklist referenced `.github/scripts/doc-scope.js`,
   `CATEGORY_DIRS`, `documentation-metadata-standard.md`, and `docs/prompts/` —
   none of which exist in this repository — and omitted the `version-bump` rule
   that actually fails a PR. Rewritten against what this repository has.
   Nothing mechanical would have caught it: the file is in the frontmatter
   exceptions register precisely because it is injected verbatim into every
   pull request.

5. ~~**`CONTRIBUTING.md` and `GOVERNANCE.md` route document scope through
   `CATEGORY_DIRS`.**~~ **Closed 2026-08-01.** Both now describe
   `.docgov.config.js` as the governing mechanism, which it already was here,
   instead of one that "may eventually" supersede `CATEGORY_DIRS`.
   `CONTRIBUTING.md` keeps a pointer to `doc-scope.js` for repositories
   scaffolded from `ai-assisted-sdd-template`, where that file does exist and
   the config imports it.

6. ~~**`CLAUDE.md` and the blueprint disagree on the branch prefix.**~~
   **Closed 2026-08-01** by blueprint v1.1.0, which corrected Section 6.4 to
   `feat/*` and pointed at `git-governance` for the full taxonomy rather than
   restating it. Sections 11 and 13's source path were corrected to
   `state/resources.md` in the same revision.
   The blueprint body is consequently **no longer verbatim intake text**;
   v1.0.0 in `git log` is the archived original.

7. ~~**Repositories do not declare their plugins.**~~ **Closed 2026-08-01.**
   All five repositories now ship `.claude/settings.json` with
   `enabledPlugins`, so plugin availability belongs to the repository rather
   than to each developer's local configuration — see
   [`docs/org-governance-adoption.md`](docs/org-governance-adoption.md).

8. **The scaffolded `CLAUDE.md` no longer carries stale policy, but the
   mechanism that let it can recur.** `git-governance` shipped seven commits
   past its `v1.1.0` tag without a version bump, and because
   `init-governance.sh` copies from the installed plugin cache rather than
   from the repository, two repositories inherited a `CLAUDE.md` asserting
   that `develop` is not protected server-side. Both were corrected by hand
   and the v1.2.0 release moved the floating `v1` tag on 2026-08-01.

   Nothing prevents a repeat: the cache resolves tags, so any future work
   merged without a release silently diverges from what consumers receive.
   A release checklist, or a check that the tag matches `main`, would close
   it structurally.

9. **This repository does not meet its own security baseline.** Secret scanning
   and Dependabot security updates are disabled, and no workflow here runs
   dependency review — while
   [`SECURITY-BASELINE.md`](SECURITY-BASELINE.md) requires all three of every
   serious repository. Both fixes are repository settings, not code, so they
   cannot be applied by a pull request. Recorded here as well as in
   `SECURITY-BASELINE.md` so this register stays the single place to read for
   what is open.

10. **This repository's `AGENTS.md` has no counterpart in the plugin that
    scaffolds it.** `AGENTS.md` is now the source of truth here and `CLAUDE.md`
    is a one-line `@AGENTS.md` import, but `git-governance`'s
    `init-governance.sh` still scaffolds a full `CLAUDE.md` and its own
    `.docgov.config.js` still pins `facts` and `fragment_sync` against
    `CLAUDE.md`. So this repository is deliberately ahead of the plugin, and a
    re-scaffold would not reproduce it. The fix belongs to `git-governance`:
    scaffold `AGENTS.md` plus a thin `CLAUDE.md`, and repoint its own pins.
    Until then the four sibling repositories keep the old shape.

11. **Branch protection was applied to four of five repositories by something
    other than the plugin's script.** `scripts/setup-branch-protection.sh`
    creates one ruleset per branch, named `protect-develop` / `protect-staging` /
    `protect-main`, and sets `delete_branch_on_merge` on the repository. What is
    actually live on `.github`, `git-governance`, `docs-governance`, and
    `platform-workflows` is a single ruleset named `branch-protection` spanning
    all three refs — correct in its rules, but created some other way, which is
    why `delete_branch_on_merge` is still `false` on all four.
    `ai-assisted-sdd-template` carries **both** schemes, four overlapping
    rulesets. The two are not interchangeable: a single ruleset spanning three
    refs cannot give `develop` and `staging`/`main` different
    `allowed_merge_methods`, so the per-branch scheme is the one to converge on.

12. **`/git-check` reports a false negative on those same four repositories.**
    It looks for a ruleset named `protect-<branch>`, finds none, and reports
    branch protection as missing on repositories that are in fact protected.
    Owned by `git-governance`; the fix is to recognize the legacy name rather
    than to rename the live rulesets first.

13. **`docs-governance`'s CI guard has one clause while its `CLAUDE.md` claims
    three.** Its `.github/workflows/pr-checks.yml` guards the docs-governance
    step with `hashFiles('.docgov.config.js') != ''` alone, but its scaffolded
    `CLAUDE.md` asserts all three of the event-name, config-presence, and
    self-disable clauses. `git-governance` pins exactly this with a
    `docs-governance-guard-clauses` fact; `docs-governance` has no `facts` entry
    at all, which is why the drift survived in the one repository that ships the
    engine.

## Canonical source

[`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md),
Sections 4 and 5. Where this file and the blueprint disagree, the blueprint wins.

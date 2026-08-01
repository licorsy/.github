---
title: "Repository Classification"
doc_type: governance
description: "The four Licorsy repository categories, what each repository owns and must not own, the single-owner matrix that prevents duplicated policy, the verified portability status of each platform repository, and the open gaps tracked against that model."
status: active
version: "1.12.0"
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
Each entry names the repository that owns the fix. Items 1, 2, 8, 9, 10, and 14
are open; items 3, 4, 5, 6, 7, 11, 12, 13, 15, and 16 are closed — kept here
with their resolution, because a gap that vanishes without a record gets
rediscovered as a new finding by the next audit.

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

9. **Dependency review is still missing; the rest of the security baseline is
   now met.** The gap as first recorded was understated — it named only this
   repository, but secret scanning and Dependabot were disabled on **all five**.
   Both are now enabled everywhere, along with secret-scanning push protection,
   which refuses a push containing a credential rather than reporting it
   afterwards. See [`SECURITY-BASELINE.md`](SECURITY-BASELINE.md) for the
   verified per-control state.

   What remains open is dependency review: `platform-workflows`'
   `ci-security.yml` implements it as a reusable workflow and no repository here
   calls it.

   Also recorded there, because it fails silently: secret-scanning validity
   checks cannot be enabled on this organization. They require GitHub Advanced
   Security and the organization is on the free plan — but the API accepts the
   setting, returns 200, and leaves it disabled. Anything that trusts the
   response code instead of reading the value back will report it as on.

10. **This repository's `AGENTS.md` has no counterpart in the plugin that
    scaffolds it.** `AGENTS.md` is now the source of truth here and `CLAUDE.md`
    is a one-line `@AGENTS.md` import, but `git-governance`'s
    `init-governance.sh` still scaffolds a full `CLAUDE.md` and its own
    `.docgov.config.js` still pins `facts` and `fragment_sync` against
    `CLAUDE.md`. So this repository is deliberately ahead of the plugin, and a
    re-scaffold would not reproduce it. The fix belongs to `git-governance`:
    scaffold `AGENTS.md` plus a thin `CLAUDE.md`, and repoint its own pins.
    Until then the four sibling repositories keep the old shape.

11. ~~**Branch protection was applied to four of five repositories by something
    other than the plugin's script.**~~ **Closed 2026-08-01.** Four repositories
    carried a single ruleset named `branch-protection` spanning all three refs
    instead of the per-branch `protect-<branch>` scheme the script creates,
    which is why `delete_branch_on_merge` was still `false` on all four;
    `ai-assisted-sdd-template` carried both schemes at once. All five now run the
    per-branch scheme, the legacy rulesets are removed, and
    `delete_branch_on_merge` is `true` everywhere — verified by reading each
    ruleset back from the API.

    The per-branch shape is not cosmetic. A single ruleset spanning three refs
    has one `allowed_merge_methods` and so cannot give `develop` a different
    merge policy from `staging`/`main`, which is what the org-wide merge policy
    requires. `setup-branch-protection.sh` now also deletes a legacy ruleset when
    it finds one — after the per-branch rulesets are in place, never before.

12. ~~**`/git-check` reports a false negative on those same four repositories.**~~
    **Closed 2026-08-01, in both directions.** It matched on the ruleset's
    *name*, so it reported four genuinely protected repositories as unprotected.

    Item 11's migration removed the symptom — every ruleset is now named
    `protect-<branch>`, so even the old name-matching logic reports correctly.
    That is luck, not a fix, so the check was corrected as well: it now matches
    on `conditions.ref_name.include` and would survive the same divergence
    happening again.

    **The corrected check is on `git-governance`'s `develop` only.** It reaches
    consumers when `main` does, since the plugin cache resolves tags — the same
    dependency recorded as item 8.

13. ~~**`docs-governance`'s CI guard has one clause while its `CLAUDE.md` claims
    three.**~~ **Closed 2026-08-01.** The guard now carries all three clauses,
    and both it and `git-governance` pin the value with a
    `docs-governance-guard-clauses` fact. The root cause was the absence of that
    pin: `git-governance` had one and stayed correct, `docs-governance` had no
    `facts` entry at all. Both repositories now also run `facts` with
    `shadow: false` — the rule ships shadow-on, reporting without failing, which
    is why a pin that existed elsewhere still let this drift through.

    Fixing it surfaced a second, worse defect three lines away, now also fixed
    and pinned: the Conventional Commits step linted every subject in the PR
    range, including the `Merge pull request #N from ...` subjects GitHub
    generates itself. Those can never conform, so the check failed **by
    construction on every promotion PR** — the exact pull request it exists to
    guard. `git-governance` already carried the `--no-merges` fix; the scaffolded
    copies never received it. **This repository had the broken form too**, and
    is fixed in the same change — see item 15 for the one that remains.

14. **Two other public repositories still name the private product
    repositories.** This repository stopped naming them on 2026-08-01 (see the
    blueprint's v1.2.0 changelog entry), but the same names remain in
    `docs-governance` — 15 occurrences across 9 source files, as provenance
    comments recording which private repository each rule was extracted from —
    and in `ai-assisted-sdd-template`, 12 occurrences across 7 files. Both
    repositories are public.

    The template's case is the sharper one. It ships
    `.github/scripts/check-public-sanitization.js`, whose `NAME_PATTERNS` exist
    precisely to keep those names out of public content, and
    `sync-to-public-mirror.sh` excludes `docs/prompts/` and `docs/reports/`
    from the export for the same reason. But that script's own header records
    it as retired — *this* repository is now the public one — so the exclusion
    protects an export that no longer happens, while the content sits public in
    the repository it was meant to be excluded from.

    Not fixed here, because the two are different judgements and neither is
    this repository's to make: `docs-governance`'s comments are load-bearing
    (they explain why an exemption exists), and the template's are frozen
    historical record its own metadata standard forbids rewriting. Recorded so
    the decision is deliberate rather than overlooked.

15. ~~**`ai-assisted-sdd-template`'s commit lint still fails by construction on
    promotion PRs.**~~ **Closed 2026-08-01.** All four repositories that run this
    check now pass `--no-merges`, and all four pin it with a
    `commit-msg-lint-skips-merges` fact at `shadow: false`. The template captured
    the change as `docs/prompts/005-fix-commit-lint-merge-subjects.md` first, per
    its own Step 12 rule.

    Worth keeping the shape of this one on record, because it is the failure mode
    this register exists for. A single defect propagated to three repositories by
    scaffolding, and stayed invisible in all three for the same reason: the
    `facts` rule that would have caught it ships **shadow-on**, reporting without
    failing. `git-governance` had the fix and the pin; every copy made from it
    predated both. The fix was not the flag — it was turning the pin into
    something that can fail.

16. ~~**`platform-workflows` carries one of the five compliance artifacts.**~~
    **Closed 2026-08-01.** All five repositories are now 5/5, measured rather
    than asserted. `platform-workflows` also runs its own
    `governance-compliance.yml` against itself at `strict: true`, referenced by
    path rather than by tag — a caller pins the released tag, but the repository
    hosting a workflow has to test the revision in the pull request, or a change
    is only exercised after it ships.

    It went unnoticed because nothing measured it. The gap surfaced only when
    that workflow needed a deliberately non-compliant repository to test
    against, and the nearest one to hand was itself — which is the general
    lesson worth keeping: **the check and the thing it checks were the same
    repository, so nothing was watching it.** The five-artifact list had been
    stated in this register since it was written; stating it is what made it
    look covered.

    The `.docgov.config.js` scope turned out not to be the open question it
    looked like. That repository holds two Markdown files — `README.md`, already
    an org-wide frontmatter exception, and the newly scaffolded `CLAUDE.md` — so
    the corpus is one file and `scope_dirs` is empty. The full eight-field
    schema still applies: a repository small enough to skip it is where the
    exception starts spreading.

## Canonical source

[`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md),
Sections 4 and 5. Where this file and the blueprint disagree, the blueprint wins.

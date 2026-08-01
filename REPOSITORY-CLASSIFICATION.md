---
title: "Repository Classification"
doc_type: governance
description: "The four Licorsy repository categories, what each repository owns and must not own, the single-owner matrix that prevents duplicated policy, the verified portability status of each platform repository, and the open gaps tracked against that model."
status: active
version: "1.1.0"
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
| `ai-assisted-sdd-template` | Content yes, CI depends on Licorsy | See known gaps below |
| `platform-workflows` | N/A | Organization-specific by design; it is the thing others point at |

## Known gaps

Open issues against this model and against the documents that describe it.
Recording them here is deliberate: an undocumented gap gets rediscovered by
every future audit, which is the failure mode this repository exists to stop.
Each entry names the repository that owns the fix — items 1, 2, 3, and 8 belong
to other repositories; items 4, 5, and 6 are `.github`'s own. Item 7 is closed
and kept for the record.

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

3. **The scaffolded `CLAUDE.md` describes a `.docgov.config.js` this
   repository does not have.** Its "Documentation ownership" section cites
   `facts` and `fragment_sync` entries, and an orphan
   `<!-- fragment:branch-flow -->` marker with no counterpart, because the
   file is copied verbatim from `git-governance`, where those entries do
   exist. A local edit would survive re-scaffolding, since the script skips
   files that already exist — but it would fork the shared file, and every
   other repository scaffolded from the plugin inherits the same wrong text.
   The fix belongs in the plugin source. Same root cause as the stale-cache
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
   `.docgov.config.js` already governs. Both passages predate this batch and
   were deliberately left alone, since rewriting them is a content change
   rather than the metadata pass this work was scoped to.

6. **`CLAUDE.md` and the blueprint disagree on the branch prefix.**
   `CLAUDE.md` uses `feat/*` and the six-prefix taxonomy; the blueprint's
   Section 6.4 still says `feature/*`. `git-governance` renamed the prefix and
   added `refactor/`, so `CLAUDE.md` reflects current practice and the
   blueprint is the stale one — but this file states that the blueprint wins,
   which makes `CLAUDE.md` formally non-conforming until the blueprint is
   revised. Resolve it in the blueprint, not by reverting `CLAUDE.md`.

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

## Canonical source

[`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md),
Sections 4 and 5. Where this file and the blueprint disagree, the blueprint wins.

---
title: "Repository Classification"
doc_type: governance
description: "The four Licorsy repository categories, what each repository owns and must not own, the single-owner matrix that prevents duplicated policy, the verified portability status of each platform repository, and the open gaps tracked against that model."
status: active
version: "1.25.0"
created: 2026-08-01
updated: 2026-08-04
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
of 2026-08-02:

| Repository | Portable standalone? | Notes |
| --- | --- | --- |
| `git-governance` | Yes, with one caveat | The `pr-checks.yml` it scaffolds hardcodes `licorsy/docs-governance/action@v1`, but the step is gated so it stays inert without a local `.docgov.config.js`. A fork that also uses docs-governance would run Licorsy's action until it repoints that line |
| `docs-governance` | Yes | Engine is config-driven; all scoping comes from the consuming repository's `.docgov.config.js` |
| `ai-assisted-sdd-template` | Yes, with the CI opted out | Content is portable. The `ci-docs`/`ci-security` jobs call Licorsy's reusable workflows, but both are guarded by `if: github.repository_owner == 'licorsy'`, so a project created from the template outside this org gets no documentation or security automation until it repoints those `uses:` lines or drops the guard — an explicit choice rather than a silent dependency (gap 1, closed) |
| `platform-workflows` | N/A | Organization-specific by design; it is the thing others point at |

## Known gaps

Open issues against this model and against the documents that describe it.
Recording them here is deliberate: an undocumented gap gets rediscovered by
every future audit, which is the failure mode this repository exists to stop.
Each entry names the repository that owns the fix. Item 20 is **open**, and
only in its second half — the ruleset update, which item 21's closure has now
made safe to apply; items 2, 14, 17, 22,
and 23 are **accepted** — real states, deliberately not scheduled for change;
every other item is closed — kept here with its resolution, because a gap that
vanishes without a record gets rediscovered as a new finding by the next audit.

**On sequencing.** Gaps 9, 18, and 19 were closed as a single change per
repository rather than one promotion each. That was a correction. Gap 8 had just
been closed on its own, and because landing it moved `main` without carrying a
version bump, it forced an entire second release cycle to make
`release-integrity` green again. That case is what taught the rule; the rule
itself is no longer stated here. It is a standing policy, not a property of this
register, and it lives in [AGENTS.md](AGENTS.md) under "Promotion cadence" —
batch, promote once per window, bump in the same breath.

1. ~~**`ai-assisted-sdd-template` CI depends on Licorsy's reusable workflows.**~~
   **Closed 2026-08-02** (`licorsy/ai-assisted-sdd-template#19`). Its `ci-docs`
   and `ci-security` jobs now carry `if: github.repository_owner == 'licorsy'`,
   so the dependency is explicit rather than silent: outside this organization
   both skip, and the adopter chooses whether to repoint the `uses:` lines at
   their own copies or consume Licorsy's knowingly.

   Two things this entry originally got wrong, both corrected by reading the
   workflow instead of inferring from the `uses:` line. It does **not** run CI
   against Licorsy's repositories — a public reusable workflow executes in the
   *caller's* context, on the caller's runners — so the real exposure was a
   dependency on definitions Licorsy could change or delete. And the prescribed
   fix was impossible: `git-governance`'s guard uses `hashFiles()`, which is
   valid in a step-level `if:` but not in the job-level `if:` a
   reusable-workflow call requires.

   The fix also falsified a claim the template was making.
   `documentation-metadata-standard.md` Section 9 promised its docs automation
   ran "on every push and pull request", which a guarded job does not deliver;
   Section 9 (v1.26) now states that none of it runs outside `licorsy`. **A
   guard that silences a check is only half the change — the document
   describing that check has to stop over-promising in the same edit.**

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

8. ~~**The scaffolded `CLAUDE.md` no longer carries stale policy, but the
   mechanism that let it can recur.**~~ **Closed 2026-08-02.** Every repository
   consumed by tag now calls `release-integrity.yml`: `git-governance`,
   `docs-governance`, and `platform-workflows` — the three that publish a
   floating `v1`, each on its own offset schedule. That is the closure criterion,
   stated here because the entry previously said "every repository" and left the
   remaining two ambiguous.

   **`.github` and `ai-assisted-sdd-template` are out of scope, not outstanding.**
   Both are consumed from the default branch — organization defaults apply from
   `main`, and *Use this template* copies the default branch — so there is no tag
   through which a consumer could receive a stale version, which is the only
   failure this check detects. `.github` carries no tags at all;
   `ai-assisted-sdd-template`'s `v1.0.0`/`v1.1.0` are inert markers with no
   floating `v1`, and `main` is 30 commits past `v1.1.0` with nobody affected.
   Pointing the check at either would fail on *"floating tag 'v1' does not
   exist"* on its first run — a false positive, and the fastest way to train
   everyone to ignore a check that is correct everywhere else. The original
   finding follows.

   `git-governance` shipped seven commits
   past its `v1.1.0` tag without a version bump, and because
   `init-governance.sh` copies from the installed plugin cache rather than
   from the repository, two repositories inherited a `CLAUDE.md` asserting
   that `develop` is not protected server-side. Both were corrected by hand
   and the v1.2.0 release moved the floating `v1` tag on 2026-08-01.

   Nothing prevents a repeat: the cache resolves tags, so any future work
   merged without a release silently diverges from what consumers receive.
   A release checklist, or a check that the tag matches `main`, would close
   it structurally.

   **Still open after the 2026-08-01 v1.3.0 releases.** Both `git-governance`
   and `docs-governance` were released properly that day, each with a checklist
   in its promotion pull request — but a checklist written per release is a
   habit, not a mechanism, and this entry is about the mechanism. The tagging
   itself also produced a near-miss worth recording: `git tag -f v1 v1.3.0`
   points the floating tag at the *tag object* rather than the commit, and
   `git rev-list -n1 v1` still resolves to the right commit, so it looks
   correct. It was caught by git's own nested-tag hint and fixed with
   `v1.3.0^{}`, then verified against `ls-remote`'s peeled refs. An automated
   check comparing the tag to `main` would have caught both this and the
   original skipped release.

   **The mechanism now exists**, as `platform-workflows`'
   `release-integrity.yml`: it compares the floating tag and the version tag
   against `main` on a schedule, and catches all three failure modes seen here —
   `main` moved with nothing tagged, tagged without moving the floating tag, and
   the floating tag pointing at a *tag object* rather than a commit. It is
   scheduled rather than push-triggered because tagging happens *after* the
   merge, so a push-triggered run would fail every release by construction.

   **The entry stayed open until every repository in scope actually called it**,
   which is what the closure above records. Shipping the check is not the same as
   running it, and this register has already confused the two once — see item 16,
   where the compliance check lived in the least compliant repository.

   Its first real run proves the point: it found `platform-workflows` itself
   drifted, with `v1` still at `v1.0.1` while `main` had moved several commits
   past. Consumers pinning `@v1` were not even receiving `scorecard.yml` — and
   the README documented `uses: ...@v1` for `governance-compliance.yml`, which
   does not exist at `v1` at all. Anyone following that example would get
   "workflow not found". That is three repositories affected by this gap, not
   the one it was opened for.

9. ~~**Dependency review is missing.**~~ **Closed 2026-08-02.** The gap as first
   recorded was understated — it named only this repository, but secret scanning
   and Dependabot were disabled on **all five**. Both are now enabled
   everywhere, along with secret-scanning push protection, which refuses a push
   containing a credential rather than reporting it afterwards. See
   [`SECURITY-BASELINE.md`](SECURITY-BASELINE.md) for the verified per-control
   state.

   Dependency review is now wired: every repository calls
   `platform-workflows`' `ci-security.yml`, which self-gates the
   dependency-review job to `staging`/`main` and runs secret scanning on every
   trigger. No repository ships a dependency manifest yet, so dependency-review
   currently passes trivially — wired now so the coverage exists the moment one
   appears, rather than depending on someone remembering then.

   Also recorded there, because it fails silently: secret-scanning validity
   checks cannot be enabled on this organization. They require GitHub Advanced
   Security and the organization is on the free plan — but the API accepts the
   setting, returns 200, and leaves it disabled. Anything that trusts the
   response code instead of reading the value back will report it as on.

10. ~~**This repository's `AGENTS.md` has no counterpart in the plugin that
    scaffolds it.**~~ **Closed 2026-08-01** by `git-governance` v1.4.0.
    `AGENTS.md` carries the policy and `CLAUDE.md` is an `@AGENTS.md` import in
    `.github`, `git-governance`, `docs-governance`, and `platform-workflows`;
    `init-governance.sh` scaffolds both, and `git-governance` repointed its own
    `facts` and `fragment_sync` off `CLAUDE.md`.

    **`ai-assisted-sdd-template` is deliberately excluded.** Its `CLAUDE.md` and
    `AGENTS.md` are *peer* adapters over `docs/manuals/operation-manual.md` —
    neither is a source of truth, both are thin pointers, which is `ADR-0003`
    principle 2 and is enforced by `check-adapter-sync.js`. Collapsing one into
    a pointer at the other would break that script and contradict an active ADR,
    the same reasoning that made item 17 an accepted overlap. The template
    already has an `AGENTS.md`; what it does not have is a *hierarchy*, and it
    should not.

    Two consequences worth keeping. `init-governance.sh` cannot migrate an
    existing repository: skip-if-exists is per file, so a target with a full
    `CLAUDE.md` receives `AGENTS.md` and ends up stating the policy twice — the
    script cannot tell a stale full copy from a deliberate local one, and
    `/git-check` now reports that shape as a hand edit. And the split briefly
    opened a hole in `governance-compliance.yml`, which checked `CLAUDE.md` but
    not `AGENTS.md`: a repository with a thin pointer to a *missing* `AGENTS.md`
    would have passed with no policy at all. The compliance set is now six
    artifacts, not five.

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

14. **ACCEPTED (2026-08-02): two other public repositories name the private
    product repositories, deliberately.** Reviewed and left as-is. The
    provenance comments explain *why* each exemption exists, which is
    load-bearing for anyone reading the rule later, and the template's
    occurrences are frozen historical record its own metadata standard forbids
    rewriting. Neither names anything sensitive — the repositories are private,
    their names are not credentials. Recorded as accepted so it stops
    resurfacing as a finding in every future audit. The original assessment
    follows.

    This repository stopped naming them on 2026-08-01 (see the
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

17. **Overlap between `ai-assisted-sdd-template`'s `check-adapter-sync.js` and
    `docs-governance`'s `fragment_sync`.** Both keep a block of prose identical
    across two files. **Recorded as accepted overlap, not scheduled for
    removal** — the same disposition as item 2, and for a comparable reason.

    The technical blocker is gone: `docs-governance` v1.3.0 added an optional
    `anchor` to `fragment_sync`, which was the one thing the bespoke script did
    that the engine could not. Before that, swapping them would have silently
    dropped the check that the heading a block restates still exists, making it
    a regression rather than a deduplication.

    It is still not worth doing, on two grounds that outweigh the duplication:

    - The template's own `docs/prompts/004` already considered adopting
      `fragment_sync` and deferred it, with a stated revisit condition —
      "revisit only if a concrete gap surfaces." No gap has surfaced. The script
      has no defect; what changed is only that an alternative became viable,
      which is an argument about tidiness, not a concrete gap.
    - The script is named in an **active** ADR (`ADR-0003`, principles 2 and 5),
      the operation manual's tooling table, and the README. Removing it means
      amending an accepted architectural decision, and leaves eight citations in
      frozen record — `docs/prompts/001`/`003`/`004`, `CHANGELOG.md`,
      `PROPOSAL-TRACKING.md` — permanently naming a deleted file, which that
      repository's metadata standard forbids rewriting.

    Amending an active ADR to delete a hundred lines that work is a bad trade.
    Revisit only if the script actually breaks, or if a second repository needs
    the same behaviour — at which point the engine already supports it.

    The engine feature is not wasted either way: `anchor` is a general
    capability available to every consumer, and it is what makes this an
    accepted overlap by choice rather than by necessity.

18. ~~**Nothing enforces that `staging`/`main` receive only promotions.**~~
    **Closed 2026-08-02** by a `promotion-source` job in `pr-checks.yml`, which
    fails any pull request into `staging` not from `develop`, and into `main`
    not from `staging`. `hotfix/` is not an exception — the taxonomy is explicit
    that the label signals urgency, not a bypass route. It lives in
    `pr-checks.yml` rather than `platform-workflows` because `git-governance`
    scaffolds that file verbatim, so every repository receives the guard with no
    extra wiring. The original finding follows.

    The rulesets require a pull request and block direct pushes, force-pushes, and
    deletion — but they place no constraint on which branch a pull request comes
    *from*. `AGENTS.md`'s branch flow says `develop -> staging` and
    `staging -> main` are promotions and never a starting point for new work;
    that half is convention, and only that half.

    Found on 2026-08-01 while preparing this repository's own promotion:
    `staging` and `main` differed, and the difference was on the wrong side.
    `cda7932` — an agent-behavior issue template — was a single-parent commit
    straight onto `main`, so `develop` and `staging` both lacked the file.

    The consequence is worse than one stray file. A `staging -> main` merge
    preserves `main`'s side, so such a commit survives on `main` while `develop`
    stays missing it, and **every future promotion re-shows the same
    difference** until someone back-merges. Fixed here by back-merging `main`
    into `develop`, which is a repair, not a guard.

    GitHub rulesets have no "allowed source branch" rule for pull requests, so
    closing this structurally means a check rather than a setting: a workflow on
    `pull_request` into `staging`/`main` that fails when `github.head_ref` is not
    the expected upstream branch. That belongs in `platform-workflows` alongside
    `release-integrity.yml`, which exists for the same shape of problem —
    a policy stated in prose with nothing verifying it.

19. ~~**The CI Conventional Commits lint cannot prevent anything.**~~
    **Closed 2026-08-02** by removing it from `pr-checks.yml` in all five
    repositories. The `commit-msg` hook in `.pre-commit-config.yaml` remains the
    enforcement layer — it runs as each commit is written, which is earlier,
    cheaper, and actually preventive. Two fixes were built on top of the CI copy
    (`--no-merges`, then pinning that flag across four repositories) before
    anyone asked whether it could prevent anything at all; that is the more
    useful lesson than the removal itself. The original finding follows.

    It ran only on pull requests into `staging`/`main`, and on a
    `develop -> staging` pull
    request **every commit in range is already merged into `develop`**. So it can
    only report history that is unfixable without rewriting a protected branch.
    The one place it could act — a work branch into `develop` — is exactly where
    `pr-checks.yml` deliberately does not run, to save Actions quota. The local
    `commit-msg` hook already gates every new commit as it is written.

    Found on 2026-08-01 when both promotion pull requests failed on it, for two
    different reasons that between them rule out the obvious narrow fixes:

    - `.github`: a commit made straight onto `main`, outside the promotion flow,
      so the local hook never saw it. Already on `main` — an exclusion for
      published history (`--not origin/main`) would cover this one.
    - `ai-assisted-sdd-template`: a squash merge where GitHub used the *branch
      name* as the subject. On `develop`, not `main`, so that exclusion does not
      reach it.

    **Both promotions were merged with the check red, as a recorded exception
    rather than an oversight.** The findings are accurate and name real
    non-conforming commits; neither can be corrected without a force-push to a
    protected branch, which this repository's own policy forbids. Merging was
    judged better than either rewriting history or leaving the branch flow
    permanently unpromotable.

    What makes this a gap rather than a nuisance: two separate fixes were built
    on top of this check earlier the same day — adding `--no-merges`, then
    pinning that flag with a `facts` entry across four repositories — before
    anyone noticed the check is structurally inert. Effort went into making a
    gate correct that cannot gate.

    **Do not extend it further before deciding what it is for.** The options are
    to drop it and rely on the local hook, or to run it on `develop` pull
    requests where it could prevent something, at the cost of quota and of
    contradicting the rationale written into `AGENTS.md`. `--no-merges` is
    separately load-bearing either way: without it the check fails by
    construction on every promotion, because GitHub writes the merge subjects
    itself.

20. **Five of `ai-assisted-sdd-template`'s workflows report under one check
    name.** `adapter-rules-check.yml`, `adapter-sync-check.yml`,
    `scope-consistency-check.yml`, `state-staleness-check.yml`, and
    `step-reference-check.yml` each name their only job `check`, and GitHub
    derives a check's context from the job's `name:` (falling back to the job
    id). All five therefore appear as a single `check` context, indistinguishable
    from one another.

    The consequence is concrete rather than cosmetic: when required status checks
    were applied on 2026-08-02, these five had to be **left out** of
    `protect-staging` and `protect-main`, because requiring `check` cannot express
    *which* of them must pass. They run and they report — nothing is silently
    skipped — but they cannot be made blocking while they share a name.

    The fix is a rename to distinct job names, then adding the new contexts to
    both rulesets. Sequence matters: a required context that has never been
    reported blocks every pull request on *"Expected — waiting for status to be
    reported"*, so the rename must merge and run at least once **before** the
    ruleset is updated, never in the same step.

    Deferred deliberately on 2026-08-02, not overlooked. **Half closed
    2026-08-03** (`licorsy/ai-assisted-sdd-template#24`): the five job ids are
    now `adapter-rules`, `adapter-sync`, `scope-consistency`, `state-staleness`
    and `step-reference`, and four of them were observed reporting under their
    own context on that pull request. What remains is adding those contexts to
    `protect-staging` and `protect-main`, which is a **separate promotion
    window** by the rule above — it is not batched with the rename, on purpose.

21. ~~**`setup-branch-protection.sh` silently deletes the required status
    checks.**~~ **Closed 2026-08-04** (`licorsy/git-governance#41`). Each
    `protect-<branch>` ruleset is now read before it is written and its
    `required_status_checks` rule carried forward verbatim, and the run reports
    which branches it preserved rather than doing it silently.

    The entry left the fix open between two options — preserve the existing
    rule, or own it outright so the two sources agree. Surveying all five
    repositories settled it: the contexts are per-repository and cannot be
    derived. The docs check reports as `docs-governance` in four repositories
    and `ci-docs / docgov` in `ai-assisted-sdd-template`; `docs-governance` adds
    one context per matrix cell (`test (20)`, `test (24)`); `platform-workflows`
    adds `governance-compliance / governance-compliance`. A required context
    that is never reported blocks the branch permanently, so a script that
    guessed one would be worse than one that sets none — **preserve, never
    invent** is the contract, and the script's header owns it.

    Two consequences worth keeping. The script correspondingly cannot *remove* a
    required check either: it faithfully preserves whatever it finds, including
    a stale context, so dropping one means editing the ruleset directly. And
    **the fix is on `git-governance`'s `develop` only** — it reaches consumers
    when `main` does, since the plugin cache resolves tags, the same dependency
    recorded as items 8 and 12. Until that release, a re-run from an installed
    copy still deletes the checks. The original finding follows.

    The script builds one ruleset payload whose `rules` array is exactly
    `deletion`, `non_fast_forward`, and `pull_request`, then `PUT`s it over the
    existing ruleset when one is found. Rulesets are replaced wholesale, not
    merged, so the required status checks applied out of band on 2026-08-02 are
    not in that payload and do not survive a re-run — on any repository, for all
    three branches, with no warning.

    Nothing has hit this yet because the script has not been re-run since the
    checks were applied. It gets more likely, not less, from here: a daily
    promotion window means the governance scripts are exercised more often, and
    the failure is silent in the worst way — protection appears configured, the
    ruleset exists, and the only missing part is the one that blocks a bad
    promotion.

    The fix belongs to `git-governance`: either read the existing ruleset and
    preserve the `required_status_checks` rule, or make the script own that rule
    outright so the two sources agree. Recorded here because this repository is
    where the gap register lives; it is not this repository's to close.

22. **The "Actions never runs on `develop`" invariant holds in three of five
    repositories.** [AGENTS.md](AGENTS.md) states the design under "Remote
    validation layer" and it is true here, in `git-governance` and in
    `platform-workflows` — a pull request into `develop` triggers nothing in any
    of the three. Two repositories diverge, and the two cases are not the same
    finding:

    - `docs-governance`'s `tests.yml` uses a bare `on: pull_request` with no
      branch filter, so its unit tests run on every pull request into `develop`.
      **Accepted, not a defect.** That workflow is the repository's only
      automated test signal before a merge — its `.pre-commit-config.yaml` runs
      file hygiene and Conventional Commits, nothing that executes the test
      suite — and `develop` has no required status checks, so the run is
      advisory and blocks nothing. Removing it would trade a real signal for a
      consistency that costs nothing to break.
    - `ai-assisted-sdd-template` lists `develop` in its `pr-checks.yml` branch
      filter and carried six path-filtered workflows that additionally fired on
      `push` to `main` and `develop`, re-running on the merge what the pull
      request had already run. **The duplication is closed** (2026-08-03,
      `licorsy/ai-assisted-sdd-template#24`): the `push:` triggers are gone,
      with the reason left in the files as a comment, since a `push:` trigger
      looks like an omission to whoever tidies the workflow next.
      **The `pr-checks.yml` branch filter is accepted**, on the same reasoning
      as `docs-governance` above — `ci-docs` and `ci-security` on a `develop`
      pull request are a real signal rather than a second copy of one, and with
      no required checks on `develop` they block nothing. What made the six
      workflows a defect was that they duplicated a run, not that they ran.

23. **Three repositories carry permanent squash-era commits on `staging` and
    `main`.** Measured 2026-08-03, counting non-merge commits present on a
    promotion branch and absent from its source:

    | Repository | unique to `staging` | unique to `main` |
    | --- | --- | --- |
    | `.github` | 0 | 0 |
    | `docs-governance` | 0 | 0 |
    | `git-governance` | 1 | 1 |
    | `platform-workflows` | 2 | 2 |
    | `ai-assisted-sdd-template` | 2 | 2 |

    All of them date from the `hom -> staging` rename of 2026-07-31 and the
    first reusable-workflow imports, when promotions were still squashed —
    before the ruleset restricted `staging` and `main` to merge commits. The
    doubled pull-request suffixes in their subjects (`... (#2) (#3)`) are the
    signature of a squash of a squash.

    **Accepted, and permanent.** A squashed promotion delivers content as a new
    commit rather than as shared history, so the copy on `staging`/`main` has no
    counterpart on `develop` and no later promotion can clear it. Nothing is
    missing — the content reached every branch — and the merge-only ruleset
    stops any new instance from appearing. Rewriting the three protected
    branches to erase it would cost more than it is worth and is not proposed.

    Recorded because it is not inert: it makes a promotion-time drift check
    report a hit on three of five repositories forever. `git-governance`'s
    `/promote-window` was written to **stop** on exactly this signal, which
    would have refused to promote those three permanently; it now reports into
    the confirmation checklist instead (`licorsy/git-governance#38`). Any future
    check over this signal must make the same distinction.

## Canonical source

[`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md),
Sections 4 and 5. Where this file and the blueprint disagree, the blueprint wins.

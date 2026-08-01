# Contributing

This organization's projects follow the Spec-Driven Development process defined in
[ai-assisted-sdd-template](https://github.com/licorsy/ai-assisted-sdd-template). This
file defines the org-wide contribution conventions, inherited by every `licorsy`
repository that doesn't define its own `CONTRIBUTING.md`.

## The Change-as-prompt principle

Every `licorsy` repository proposes and reviews a non-trivial change before merging
it (see [GOVERNANCE.md](https://github.com/licorsy/.github/blob/main/GOVERNANCE.md)'s Change process section and the non-overridable
"Excessive agency" control in [SECURITY.md](https://github.com/licorsy/.github/blob/main/SECURITY.md)). **This table is the single
definition of both ways to satisfy it; every other document in this org points back
to it instead of restating it.**

| | Full mechanism | Lightweight path |
| --- | --- | --- |
| **Applies when** | repository has a `docs/prompts/` directory | repository has no `docs/prompts/` directory (this repo, today) |
| **Proposal record** | `docs/prompts/NNN-<slug>.md`, drafted and reviewed **before implementation begins** | the PR's own description, stated **before merge** — a later, weaker guarantee than the full mechanism's, accepted here in exchange for requiring no scaffold |
| **Trivial-change exemption** | typo fixes / obviously reversible one-liners skip the record — see Step 12a's tiering criteria in the operation manual if it's unclear which bucket a change falls into | same exemption, same criteria |
| **Decision records** | non-trivial process/architecture decisions also get a `docs/adr/` entry | the merged PR is the decision's lasting record; no `docs/adr/` file required |
| **Full definition** | `docs/manuals/operation-manual.md`, Step 12, in [ai-assisted-sdd-template](https://github.com/licorsy/ai-assisted-sdd-template) — read directly from that repository if this one has no local copy | this table |

"Proposing a change (full mechanism)" through "Prompt lifecycle (full mechanism)"
below spell out the full mechanism's `docs/prompts/`- and `docs/reports/`-specific
tooling in detail; skip both sections if this repository has neither directory.

## Proposing a change (full mechanism)

Every non-trivial change to a project (its docs, governance, prompts, or tooling) is
captured as a `docs/prompts/NNN-<slug>.md` file before implementation begins. Use
`docs/prompts/basic-prompt-template.md` as the starting scaffold. A small, related
batch of individually-minor fixes can share one prompt document (Step 12 rule 9);
structural changes get their own.

Proposals from an external improvement report dropped into `docs/reports/` get logged
into that folder's `PROPOSAL-TRACKING.md` at intake — one row per distinct proposal,
status `not-triaged` until a decision is made. Batch-evaluation sessions update that
file's `Status`/`Decision` columns directly; they don't produce a new summary document
each time.

## Prompt lifecycle (full mechanism)

Each prompt file's frontmatter `status` field tracks its own lifecycle:

1. `draft` — written, not yet approved for execution.
2. `active` — approved and being implemented on the current branch.
3. `archived` — merged and verified; the file stays in place as historical record (see
   `docs/manuals/documentation-metadata-standard.md` — there is no folder-per-status
   move, and no precedent for deleting a prompt file once archived).
4. `deprecated` — drafted but declined or superseded before execution; the decline
   reason is recorded in the prompt's own body, not only in `PROMPT-INDEX.md`
   (`docs/manuals/operation-manual.md`, Step 11, trigger 4).

`docs/prompts/PROMPT-INDEX.md` is the id/status/one-line-purpose index across every
prompt except the blank scaffold; update it alongside any status change.

## Conventions

- **File naming**: root-level entry-point files use UPPER-CASE (`README.md`,
  `QUICKSTART.md`, `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `SECURITY.md`,
  `CHANGELOG.md`); everything under `docs/` and `agents/`, plus `.github/workflows/`
  and `.github/scripts/`, uses lowercase-hyphenated names (`operation-manual.md`,
  `075-prompt-governance-hygiene-batch.md`). GitHub-mandated files in `.github/` keep
  the exact names GitHub requires (`CODEOWNERS`, `ISSUE_TEMPLATE/`,
  `PULL_REQUEST_TEMPLATE.md`).
- **Documentation metadata** (repositories with `.github/scripts/doc-scope.js`): every
  Markdown file in the living-document directories or in-scope root files carries the
  YAML frontmatter schema described in `docs/manuals/documentation-metadata-standard.md`
  Section 1. `doc-scope.js`'s `CATEGORY_DIRS` is authoritative for which *directories*
  are in scope; Section 1 is authoritative for which *root files* are (`QUICKSTART.md`
  is the only one) — the two enumerate different things and don't compete.
  The `CATEGORY_DIRS` mechanism is what's authoritative for now; it may eventually
  be superseded by the `docs-governance` plugin's own scope mechanism (see
  GOVERNANCE.md's "Plugin and
  process-template versioning" section).
- **Local validation gate** (repositories with `.pre-commit-config.yaml`): projects use
  `pre-commit` as their primary gate. After cloning, run `pre-commit install` and
  `pre-commit install --hook-type commit-msg` once. The configured hooks are file
  hygiene (trailing whitespace, EOF, YAML/JSON syntax, merge markers) and Conventional
  Commits on the commit subject.
- **Commit messages**: Conventional Commits (`feat`, `fix`, `docs`, `refactor`,
  `chore`, `test`, `perf`, `build`, `ci`) is the expected format everywhere. Where
  `.pre-commit-config.yaml` exists it's enforced by the `commit-msg` hook; where
  `.github/workflows/pr-checks.yml` exists it's re-checked on `staging`/`main` PRs.
- **Branch model**: branch names and promotion rules (`develop` → `staging` → `main`)
  are defined by the `git-governance` plugin, not here.

## Pull requests

Use `.github/PULL_REQUEST_TEMPLATE.md`'s checklist. Its first item recognizes both
rows of the table above (an approved `docs/prompts/` file, or this PR's own
description where there's no `docs/prompts/` directory), plus the separate
trivial/obviously-reversible exemption. The remaining items (living-document
frontmatter, prompt `status`) only apply where the corresponding tooling exists.

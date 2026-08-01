## Summary

<!-- What changed and why, in 1-3 sentences. -->

## Change-as-prompt checklist

- [ ] The change has a proposal record: if this repository has a `docs/prompts/` directory, an approved `docs/prompts/NNN-<slug>.md` written **before implementation began**; if it does not, this PR's own description, stated **before merge**. See [`CONTRIBUTING.md`](https://github.com/licorsy/.github/blob/main/CONTRIBUTING.md)'s Change-as-prompt table — **or** the change is trivial / obviously reversible.
- [ ] If it touched a governed Markdown file, its YAML frontmatter is current: `status`, `version`, `updated`, plus a changelog entry where that document keeps one. Scope and exclusions are declared in this repository's own `.docgov.config.js`; the org-wide exceptions register is in [`docs/org-governance-adoption.md`](https://github.com/licorsy/.github/blob/main/docs/org-governance-adoption.md).
- [ ] If it changed a versioned document's content, `version` was bumped in the same commit. Note this is **enforced remotely only on PRs into `staging`/`main`** — on `develop` it is a manual check.
- [ ] If this PR executes a `docs/prompts/` file's described change, that prompt's `status` moved to `active` while working, and to `archived` once merged and verified.

## Verification

<!-- Commands run and their output, files checked, workflow results. Evidence, not assertions. -->

## Summary

<!-- What changed and why, in 1-3 sentences. -->

## Change-as-prompt checklist

- [ ] This PR corresponds to an approved `docs/prompts/NNN-<slug>.md` (see `docs/manuals/operation-manual.md`, Step 12 — a shared prompt document may cover a batch of related minor fixes), **or** it is explicitly trivial/obviously reversible, **or** this repository has no `docs/prompts/` directory and this PR's description is the proposal record (see `CONTRIBUTING.md`'s Change-as-prompt table).
- [ ] If it touched any living-document directory or in-scope root file (`.github/scripts/doc-scope.js`'s `CATEGORY_DIRS` / `documentation-metadata-standard.md` Section 1), their YAML frontmatter (`status`, `version`, `updated`, changelog entry) is current — see `docs/manuals/documentation-metadata-standard.md`.
- [ ] If this PR executes a `docs/prompts/` file's described change, that prompt's own `status` field was moved to `active` while working and to `archived` once merged and verified.

## Verification

<!-- How you confirmed this works: commands run, files checked, workflow output, etc. -->

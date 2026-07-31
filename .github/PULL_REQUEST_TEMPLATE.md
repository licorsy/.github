## Summary

<!-- What changed and why, in 1-3 sentences. -->

## Change-as-prompt checklist

- [ ] This PR corresponds to an approved `docs/prompts/NNN-<slug>.md` (see `docs/manuals/operation-manual.md`, Step 12), **or** it is explicitly trivial/obviously reversible/already pre-authorized as part of a batch.
- [ ] If it touched any living-document directory (`documentation-metadata-standard.md` Section 1 / `.github/scripts/doc-scope.js`'s `CATEGORY_DIRS`), their YAML frontmatter (`status`, `version`, `updated`, changelog entry) is current — see `docs/manuals/documentation-metadata-standard.md`.
- [ ] If this PR executes a `docs/prompts/` file's described change, that prompt's own `status` field was moved to `active` while working and to `archived` once merged and verified.

## Verification

<!-- How you confirmed this works: commands run, files checked, workflow output, etc. -->

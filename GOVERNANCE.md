# Governance

## Decision-making

Architecture and process decisions are captured as Architecture Decision Records (ADRs)
in each project's `docs/adr/` directory, following the format in `ai-assisted-sdd-template`.

## Change process

All non-trivial changes follow the Change-as-prompt rule defined in
`.github/CONTRIBUTING.md`.

## Plugin versioning

Plugins (`git-governance`, `docs-governance`) follow semantic versioning.
Breaking changes increment the major version; downstream repositories pin `@v1`, `@v2`, etc.

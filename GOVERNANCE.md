---
title: "Governance"
doc_type: governance
description: "How decisions are made and changes are approved across the licorsy organization: where the canonical operating model lives, ownership and the private contact channel, the Change-as-prompt gate, and semantic versioning for plugins and the process template."
status: active
version: "1.0.0"
created: 2026-07-31
updated: 2026-08-01
language: en
id: governance
owner: Alexandre Clemente
tags: [governance, decision-making, versioning]
related: [organizational-blueprint, contributing, security-policy, code-of-conduct]
---

# Governance

## Canonical operating model

This file covers decision-making and change process. The organization's full
operating model — principles, repository architecture, engineering standards,
security baseline, maturity path — is
[`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md),
which is authoritative wherever any other document summarizes it.

## Ownership

This organization is solo-maintained. Until a per-repository `CODEOWNERS` file is
added, the maintainer of record for any `licorsy` repository is whoever appears in
that repository's commit history. The one private channel to them guaranteed to
exist is GitHub's "Report a vulnerability" feature on that repository (see
[SECURITY.md](https://github.com/licorsy/.github/blob/main/SECURITY.md)'s Reporting a concern section) — used for security disclosures by default,
and as this org's private channel of last resort for anything else that needs to
reach the maintainer privately, including conduct reports (see
[CODE_OF_CONDUCT.md](https://github.com/licorsy/.github/blob/main/CODE_OF_CONDUCT.md)).

## Decision-making

Architecture and process decisions are non-trivial changes like any other, gated by
the Change-as-prompt principle below. A repository using the full mechanism records
them as Architecture Decision Records (ADRs) in `docs/adr/`, following the format in
`ai-assisted-sdd-template`; see [CONTRIBUTING.md](https://github.com/licorsy/.github/blob/main/CONTRIBUTING.md)'s Change-as-prompt table for the
lightweight path's equivalent (the merged PR is the record — no `docs/adr/` file
required). In repositories that use that tooling, whether `docs/adr/` carries
living-document frontmatter is governed by the `CATEGORY_DIRS` enumeration
CONTRIBUTING.md's Conventions section describes; this file doesn't restate that list.

## Change process

All non-trivial changes follow the Change-as-prompt principle. Both compliance paths
— the full `docs/prompts/` mechanism and the lightweight PR-description path — are
defined in a single table in [CONTRIBUTING.md](https://github.com/licorsy/.github/blob/main/CONTRIBUTING.md); this file doesn't restate
which applies when.

## Plugin and process-template versioning

Plugins (`git-governance`, `docs-governance`) follow semantic versioning.
Breaking changes increment the major version; downstream repositories pin `@v1`, `@v2`, etc.

`ai-assisted-sdd-template` — the source of the Spec-Driven Development process, the
operation manual, and its step numbering cited throughout this org's docs, including
this repository's own — is versioned the same way. Step-number citations (e.g. "Step
12") track whatever version was current when last verified; treat an exact step
number as approximate after any upstream release and re-verify before relying on it.

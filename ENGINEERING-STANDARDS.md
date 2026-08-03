---
title: "Engineering Standards"
doc_type: governance
description: "Organization-wide engineering standards: the Lean + Spec-Driven Development delivery flow, the branch promotion model, the required quality gates at each promotion step, the standard document set every repository carries, the tooling strategy, and an addendum of tooling candidates that are explicitly not yet policy."
status: active
version: "1.1.0"
created: 2026-08-01
updated: 2026-08-03
language: en
id: engineering-standards
owner: Alexandre Clemente
tags: [engineering, sdd, delivery, quality-gates, tooling, governance]
related: [organizational-blueprint, architecture-principles, repository-classification, security-baseline]
---

# Engineering Standards

How work moves from idea to production across Licorsy repositories. This is an
entry point: [`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md)
is the canonical source, and Sections 6, 7, 8, and 10 there are authoritative
wherever this file summarizes.

## Delivery model

Licorsy uses **Lean + Spec-Driven Development (SDD)**. The delivery flow:

1. Opportunity and discovery
2. PRD
3. Plan
4. Architecture and ADRs
5. Spec
6. Tasks
7. Implementation
8. Testing
9. Staging validation
10. Production release
11. Operations feedback loop

The project-level mechanics of this method — artifact templates, prompt
lifecycle, step numbering — are owned by
[ai-assisted-sdd-template](https://github.com/licorsy/ai-assisted-sdd-template),
not by this repository.

## Branch model

Work branches merge into `develop`, which is then promoted `develop` →
`staging` → `main`. `develop` is the integration branch, `staging` is
pre-production validation, and `main` is the production release branch.

**Branch naming, the full prefix taxonomy, merge permissions, and commit
conventions are owned by [git-governance](https://github.com/licorsy/git-governance)
and are deliberately not restated here.** Duplicating them is the exact failure
mode the blueprint's Section 5.1 ownership matrix exists to prevent.

How often that promotion path is walked is not a `git-governance` question — it
is Licorsy-specific and therefore org policy. See `AGENTS.md` in this repository
under "Promotion cadence", also not restated here.

## Required gates

**Before coding**

- approved PRD
- approved architecture direction
- approved ADRs for major choices
- implementation spec available

**Before merge to `develop`**

- tests pass
- lint passes
- docs governance passes
- review completed

**Before merge to `staging`**

- integration checks pass
- security checks pass
- release checklist prepared

**Before merge to `main`**

- staging validated
- release notes prepared
- rollback path documented
- production observability ready

## Standard document set

**Organization-level governance** (this repository): `GOVERNANCE.md`,
`ARCHITECTURE-PRINCIPLES.md`, `ENGINEERING-STANDARDS.md`,
`REPOSITORY-CLASSIFICATION.md`, `SECURITY-BASELINE.md`. This follows the
blueprint's Section 7.1. Its Section 4.1 lists a different set for the same
repository — it adds `README.md` and the issue and pull request templates but
omits `SECURITY-BASELINE.md` — and 7.1 is the one taken here.

**Every product repository**, at minimum: `README.md`, `CLAUDE.md`, `AGENTS.md`
(if applicable), `catalog-info.yaml`, `docs/adr/`, `docs/architecture/`,
`docs/operations/`, `docs/status/`, `docs/risks/`, `CHANGELOG.md`.

**Under `docs/architecture/`**: `system-context.md`, `containers.md`,
`deployment.md`, `security.md`, `data-flow.md`.

**Under `docs/operations/`**: `runbook.md`, `incident-management.md`,
`backup-recovery.md`, `observability.md`, `release-process.md`.

## Tooling strategy

Licorsy prefers existing tools over internal replacements. The recommended set
— spanning source control, documentation, observability, security, testing,
infrastructure, and AI engineering — is enumerated in the blueprint's Section
10 and is not duplicated here. Several of its entries are alternatives rather
than a stack to adopt wholesale (LangSmith *or* Langfuse, Semgrep *or*
SonarQube Community, Tempo *or* Jaeger); read it as a shortlist per concern.

Build internally only when the capability is core to competitive
differentiation, market tools cannot express the governance or workflow needed,
or ongoing maintenance costs less than the tool sprawl it replaces.

### Addendum — candidates for a future blueprint revision

The following came out of a review of the SDLC reference material that predates
the blueprint. They are **not yet policy** — they are recorded here so the next
blueprint revision can accept or reject them deliberately rather than
rediscovering them:

- **HashiCorp Vault** — secrets management for any future that is not
  AWS-only. The current baseline (AWS Secrets Manager) assumes a single cloud.
- **LangGraph** — stateful, multi-step agent orchestration. Complements
  LangSmith/Langfuse, which cover observability rather than orchestration, and
  so is an addition to that shortlist rather than a substitute for it.
- **Excalidraw** — fast architecture sketching ahead of formal C4/Structurizr
  work.
- **Vocabulary note** — Licorsy's SDD approach belongs to the same pattern
  family as **GitHub Spec Kit**. Naming that association makes the model legible
  to people outside the organization, which is the point of the
  "standards use market language" principle.

Deliberately **not** proposed: Jira/Confluence, ServiceNow, TestRail, Splunk,
Datadog, and ELK. Each is a reasonable enterprise choice, but the
managed-services-first and golden-paths principles argue against that
operational weight while the organization is still establishing the earlier
levels of the blueprint's maturity model. Worth revisiting once Level 3 is the
working assumption.

## Canonical source

[`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md),
Sections 6, 7, 8, and 10. Where this file and the blueprint disagree, the
blueprint wins.

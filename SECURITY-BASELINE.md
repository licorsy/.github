---
title: "Security and Reliability Baseline"
doc_type: governance
description: "The organization-level security and reliability controls Licorsy repositories adopt, split by whether they apply to every repository or only to those with a deployment surface, the additional controls required for regulated or sensitive systems, and how this baseline relates to the org-wide SECURITY.md policy."
status: active
version: "1.1.0"
created: 2026-08-01
updated: 2026-08-01
language: en
id: security-baseline
owner: Alexandre Clemente
tags: [security, reliability, observability, incident-response, governance]
related: [organizational-blueprint, engineering-standards, architecture-principles, security-policy]
---

# Security and Reliability Baseline

The security and reliability controls Licorsy repositories adopt. This is an
entry point: [`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md)
is the canonical source, and Section 9 there is authoritative wherever this file
summarizes.

## How this relates to `SECURITY.md`

The two files cover different surfaces and neither overrides the other:

- **[SECURITY.md](SECURITY.md)** covers **LLM and agent operational risk** —
  prompt injection, insecure output handling, excessive agency, and the rest of
  the OWASP LLM Top 10 mapping — and is where vulnerability reporting is
  defined. Only its LLM/AI-specific section is non-overridable org-wide; its
  Scope section is a default that a repository's own `SECURITY.md` takes
  precedence over.
- **This file** is the **organization-level set of infrastructure and
  application security controls**. Its repository-level controls apply
  everywhere; its runtime-level controls apply once there is a deployment
  surface.

A documentation or tooling repository needs the org `SECURITY.md` plus this
file's repository-level controls. A repository with application code needs
three things: the org `SECURITY.md`'s non-overridable LLM section, this
baseline in full, and its **own** `SECURITY.md` describing its actual
application and deployment surface — which is the artifact the org
`SECURITY.md`'s Scope section asks for.

## Baseline controls

The blueprint lists these together as what every serious project adopts. They
split by what they need in order to apply at all:

**Repository-level — every Licorsy repository, including documentation and
tooling ones:**

- branch protection and rulesets
- dependency review
- secret scanning

**Runtime-level — every repository with a deployment surface:**

- structured logging
- tracing correlation IDs
- OpenTelemetry instrumentation
- backup strategy
- production smoke tests
- incident severity model
- blameless RCA process

The split is a reading of scope, not a relaxation: nothing here is optional for
a repository that has the surface the control applies to.

### Applied state

As of 2026-08-01, across all five `licorsy` repositories — not just this one:

| Control | State |
| --- | --- |
| Branch protection and rulesets | **Enabled.** One ruleset per protected branch on `develop`, `staging`, `main` |
| Secret scanning | **Enabled** |
| Secret scanning push protection | **Enabled** |
| Dependabot alerts and security updates | **Enabled** |
| Secret scanning validity checks | **Not available** — GitHub Advanced Security only, and this organization is on the free plan |
| Dependency review | **One repository of five.** Only `ai-assisted-sdd-template` calls `platform-workflows`' `ci-security.yml`, which implements it |

Push protection is the control worth naming separately: secret scanning reports
a leaked credential *after* it is pushed, push protection refuses the push. On
public repositories both are free.

The validity-checks row is recorded because the failure is silent, not loud.
`PATCH /repos/{owner}/{repo}` accepts
`security_and_analysis.secret_scanning_validity_checks.status: "enabled"`,
returns 200, and leaves the setting `disabled`. Reading the response back is the
only way to find out — which is why every row above was verified against the
API rather than assumed from a successful call.

**One repository-level control remains unmet: dependency review**, in four of
the five repositories. A baseline whose own home repository quietly fails it is
decoration, so it is stated rather than quietly dropped. Closing it means
consuming `platform-workflows`' `ci-security.yml`, the way
`ai-assisted-sdd-template` already does.

Worth being honest about what that would buy: none of the four has a dependency
manifest today, so dependency review would pass trivially. That makes it cheap
to add, not urgent — and it is the reason the gap survived unnoticed.

## Additional controls for regulated or sensitive systems

Where the system handles regulated, personal, or otherwise sensitive data, add:

- threat model
- data classification
- least privilege IAM
- encryption at rest and in transit
- audit logging

The architecture-principles rule that internet-facing systems get explicit
threat modeling is stated in
[ARCHITECTURE-PRINCIPLES.md](ARCHITECTURE-PRINCIPLES.md) and applies
independently of whether the system is regulated.

## Where these controls are documented

The baseline is implemented in code and configuration, and described in the
operations and architecture document sets defined in
[ENGINEERING-STANDARDS.md](ENGINEERING-STANDARDS.md) — in particular
`docs/architecture/security.md`, `docs/operations/incident-management.md`,
`docs/operations/backup-recovery.md`, and `docs/operations/observability.md`.

Implementation of the shared CI security checks themselves — dependency review,
secret scanning — is owned by
[platform-workflows](https://github.com/licorsy/platform-workflows), not by this
repository. Consume those workflows; do not reimplement them per repository.

## Canonical source

[`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md),
Section 9. Where this file and the blueprint disagree, the blueprint wins.

---
title: "Security and Reliability Baseline"
doc_type: governance
description: "The infrastructure and application security controls every Licorsy project with a runtime surface adopts, the additional controls required for regulated or sensitive systems, and how this baseline relates to the org-wide SECURITY.md policy."
status: active
version: "1.0.0"
created: 2026-08-01
updated: 2026-08-01
language: en
id: security-baseline
owner: Alexandre Clemente
tags: [security, reliability, observability, incident-response, governance]
related: [organizational-blueprint, engineering-standards, architecture-principles, security-policy]
---

# Security and Reliability Baseline

The controls every Licorsy project with a runtime surface adopts. This is an
entry point: [`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md)
is the canonical source, and Section 9 there is authoritative wherever this file
summarizes.

## How this relates to `SECURITY.md`

The two files cover different surfaces and neither overrides the other:

- **[SECURITY.md](SECURITY.md)** governs **LLM and agent operational risk** —
  prompt injection, insecure output handling, excessive agency, and the rest of
  the OWASP LLM Top 10 mapping. That policy applies organization-wide, is
  explicitly non-overridable, and is the complete default for repositories that
  ship no application code. It is also where vulnerability reporting is defined.
- **This file** is the **infrastructure and application security checklist** a
  repository adopts once it has application code and a deployment surface —
  precisely the case `SECURITY.md`'s own Scope section defers to a per-project
  policy.

A documentation or tooling repository needs `SECURITY.md` only. A product
repository needs both.

## Baseline controls

Every serious project adopts all of these:

- branch protection and rulesets
- dependency review
- secret scanning
- structured logging
- tracing correlation IDs
- OpenTelemetry instrumentation
- backup strategy
- production smoke tests
- incident severity model
- blameless RCA process

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

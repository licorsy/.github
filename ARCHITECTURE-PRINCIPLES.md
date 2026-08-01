---
title: "Architecture Principles"
doc_type: governance
description: "Organization-wide architecture principles, the architecture description standards Licorsy adopts (ADR, C4, arc42-lite, ISO 42010), the required architecture views, and the default architectural posture for Licorsy products."
status: active
version: "1.0.0"
created: 2026-08-01
updated: 2026-08-01
language: en
id: architecture-principles
owner: Alexandre Clemente
tags: [architecture, principles, adr, c4, governance]
related: [organizational-blueprint, engineering-standards, repository-classification, security-baseline]
---

# Architecture Principles

Organization-wide architecture principles for every Licorsy repository. This is
an entry point: [`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md)
is the canonical source, and Sections 2 and 6 there are authoritative wherever
this file summarizes.

## Organizational principles

1. **Platform before repetition** — any repeated engineering behavior should
   become a reusable capability, not a manual habit.
2. **Documentation is part of the system** — architecture, process, and
   operational decisions live in version control and are reviewable.
3. **Standards use market language** — established industry concepts and names
   are used whenever they exist.
4. **Managed services first** — prefer managed cloud services over building
   operational complexity in-house.
5. **Security by default** — secrets, dependencies, and deployments are governed
   by default, not by memory.
6. **Architecture decisions are explicit** — significant trade-offs are recorded
   as ADRs with rationale and consequences.
7. **Golden paths over unlimited flexibility** — new projects start from
   approved templates and reusable workflows.
8. **Public framework, private product by default** — frameworks, templates, and
   engineering capabilities may be public; products and sensitive client systems
   stay private until deliberately published.

## Architecture description standard

Each product repository documents its architecture using established standards
rather than bespoke formats:

- **ADR** (Architecture Decision Record) — for decisions.
- **C4 Model** — for diagrams.
- **arc42-lite** — for the architecture narrative.
- **ISO 42010** — for stakeholder concerns and viewpoints.

## Required architecture views

Every serious project includes all seven:

1. System Context View
2. Container View
3. Deployment View
4. Data Flow View
5. Operational View
6. Security View
7. Decision Log (ADR set)

The file layout these map onto is defined in
[ENGINEERING-STANDARDS.md](ENGINEERING-STANDARDS.md)'s Standard document set,
which this file does not restate.

## Default architecture style

The default architectural posture for Licorsy products:

- **Modular monolith first**
- **API-first**
- **Managed services first**
- **Event-driven only when justified**
- **Observability by default**
- **Infrastructure as Code mandatory**
- **Explicit threat modeling for internet-facing systems**

These are defaults, not prohibitions. Departing from one is a legitimate
decision — it is exactly the kind of trade-off principle 6 requires be recorded
as an ADR, with its rationale and consequences.

## Canonical source

[`docs/licorsy-organizational-blueprint.md`](docs/licorsy-organizational-blueprint.md),
Sections 2 and 6. Where this file and the blueprint disagree, the blueprint wins.

---
title: "Licorsy Organizational Blueprint"
subtitle: "Operating Model, Engineering Standards, and Platform Architecture"
doc_type: governance
description: "Canonical operating model for Licorsy — organizational principles, repository architecture and ownership boundaries, architecture and engineering standards, security and reliability baseline, tooling strategy, maturity model, and governance cadence."
status: active
version: "1.4.0"
created: 2026-08-01
updated: 2026-08-07
language: en
id: organizational-blueprint
owner: Alexandre Clemente
review_cycle: quarterly
tags: [governance, operating-model, architecture, platform, engineering-standards]
related: [architecture-principles, engineering-standards, repository-classification, security-baseline, governance]
---

# Licorsy Organizational Blueprint

Changelog:

- v1.4.0: Section 8.2's "Before merge to `develop`" gate no longer lists
  "review completed" — it contradicted `AGENTS.md`'s autonomous,
  zero-required-approval merge policy for `develop`. Also dropped "docs
  governance passes" from that same gate: not actually enforced pre-merge on
  `develop` in most repositories today. Replaced both with what actually
  gates that merge.
- v1.3.0: Section 14 distinguishes the *review* cadence it defines from the
  *delivery* cadence introduced in `AGENTS.md`, so the two uses of the word do
  not read as one policy stated twice.
- v1.2.0: Sections 4.4, 11, and 13 no longer name the private product
  repositories individually. This repository is public and they are not; naming
  them published the organization's private project list for no governance
  benefit, since every rule applies to the category. One of the names was also
  stale — it referred to a repository that does not exist.

---

## 1. Purpose

Licorsy is an AI-first software engineering organization designed to:
- build internal engineering capabilities as reusable platform products;
- deliver production-grade software using Spec-Driven Development;
- operate with professional standards in architecture, security, documentation, and delivery;
- function both as a software business foundation and as a professional portfolio for Applied AI Engineering.

This blueprint defines the target operating model, repository responsibilities, architectural standards, and maturity path for the organization.

## 2. Organizational principles

The organization follows these principles:

1. **Platform before repetition**
   Any repeated engineering behavior should become a reusable capability, not a manual habit.

2. **Documentation is part of the system**
   Architecture, process, and operational decisions must live in version control and be reviewable.

3. **Standards use market language**
   Licorsy uses established industry concepts and names whenever possible.

4. **Managed services first**
   Prefer managed cloud services before building operational complexity in-house.

5. **Security by default**
   Secrets, dependencies, and deployments must be governed by default, not by memory.

6. **Architecture decisions are explicit**
   Significant trade-offs are documented as ADRs with rationale and consequences.

7. **Golden paths over unlimited flexibility**
   New projects start from approved templates and reusable workflows.

8. **Public framework, private product by default**
   Frameworks, templates, and engineering capabilities may be public; products and sensitive client systems remain private until deliberately published.

## 3. Target operating model

Licorsy follows a **Product + Platform** operating model.

### 3.1 Capability areas

#### A. Business and Product
Owns:
- opportunity identification;
- product framing;
- stakeholder discovery;
- success metrics;
- roadmap prioritization;
- go-to-market readiness.

Primary artifacts:
- PRD
- stakeholder map
- opportunity assessment
- roadmap
- release notes
- KPI review

#### B. Architecture and Platform
Owns:
- templates and golden paths;
- engineering platform capabilities;
- CI/CD standards;
- documentation governance;
- Git governance;
- infrastructure baselines.

Primary artifacts:
- ADRs
- architecture principles
- platform roadmap
- reusable workflows
- repo classifications
- engineering standards

#### C. Delivery and Quality
Owns:
- implementation flow;
- test strategy;
- review gates;
- defect handling;
- release readiness.

Primary artifacts:
- implementation spec
- task breakdown
- test plan
- quality gates
- release checklist
- retrospective notes

#### D. Operations and Security
Owns:
- observability;
- incident response;
- runtime reliability;
- security baseline;
- secrets management;
- backup and recovery.

Primary artifacts:
- runbooks
- incident playbooks
- threat models
- SLOs
- RCA reports
- operational dashboards

## 4. Repository architecture

Licorsy repositories are classified into four categories.

### 4.1 Organization repositories

#### `.github`
Purpose:
- organization profile;
- default community health files;
- issue templates;
- pull request templates;
- shared governance documents.

Must contain:
- `README.md`
- `GOVERNANCE.md`
- `ARCHITECTURE-PRINCIPLES.md`
- `ENGINEERING-STANDARDS.md`
- `REPOSITORY-CLASSIFICATION.md`
- `.github/ISSUE_TEMPLATE/`
- `.github/PULL_REQUEST_TEMPLATE.md`

#### `platform-workflows`
Purpose:
- reusable GitHub Actions workflows;
- security checks;
- docs governance orchestration;
- release automation;
- deployment workflow building blocks.

Must contain:
- `README.md`
- `.github/workflows/ci-docs.yml`
- `.github/workflows/ci-security.yml`
- `.github/workflows/release.yml`
- `.github/workflows/deploy-aws.yml` (future)

### 4.2 Platform capability repositories

#### `git-governance`
Purpose:
- branch strategy;
- merge permissions;
- commit conventions;
- governance commands for Claude Code.

Ownership:
- defines Git workflow rules;
- does not own org-level defaults;
- does not own project architecture.

#### `docs-governance`
Purpose:
- deterministic documentation consistency checks;
- doc-specific CI validation;
- doc review subagents.

Ownership:
- governs Markdown consistency and factual drift;
- does not own generic CI orchestration;
- does not own business process content.

### 4.3 Method repository

#### `ai-assisted-sdd-template`
Purpose:
- golden path for new software projects;
- SDD operating model;
- project bootstrap;
- process manuals and project-level governance structure.

Ownership:
- project delivery method;
- project artifact structure;
- project bootstrap instructions.

Must not own:
- org-wide issue/PR templates;
- org-wide community health files;
- reusable CI implementation logic already hosted in `platform-workflows`;
- plugin logic from `git-governance` or `docs-governance`.

### 4.4 Product repositories

Examples:
- the organization's private product repositories
- future client or internal products

These are referred to as a class rather than by name: this repository is public
and the product repositories are private, so enumerating them here would publish
the organization's private project list. Every rule in this section applies to
the category, not to any one product.

Purpose:
- implement actual business capabilities;
- consume platform capabilities and template standards.

Must contain:
- project README
- ADRs
- architecture views
- runbooks
- deployment docs
- telemetry and status artifacts
- project-specific governance and risks

## 5. Responsibility boundaries

### 5.1 Decoupling rules

Use this ownership matrix to avoid redundancy.

| Concern | Single owner | Consumers |
|---|---|---|
| Org profile and default community files | `.github` | all repositories |
| Reusable CI workflows | `platform-workflows` | all delivery repositories |
| Branch/merge/commit governance | `git-governance` | all repositories using Claude Code |
| Markdown/document consistency | `docs-governance` | repositories with governed docs |
| Project delivery method | `ai-assisted-sdd-template` | all new product repositories |
| Product-specific implementation | product repositories | project team |

### 5.2 Separation rules

1. A repository may **consume** a capability without embedding its logic.
2. A repository may **reference** another repository in documentation without duplicating its policy.
3. Organization defaults belong in `.github`, not in every template.
4. CI workflow logic belongs in `platform-workflows`, not copied inline across repositories.
5. The template defines **how projects are structured**, not **how every org-level policy is implemented**.
6. Plugins should remain portable and usable outside Licorsy.

## 6. Architecture standards

Licorsy adopts the following market-standard architecture and engineering practices.

### 6.1 Architecture description standard

Each product repository should document architecture using:
- **ADR** for decisions;
- **C4 Model** for diagrams;
- **arc42-lite** for architecture narrative;
- **ISO 42010 thinking** for stakeholder concerns and viewpoints.

### 6.2 Required architecture views

Each serious project must include:

1. **System Context View**
2. **Container View**
3. **Deployment View**
4. **Data Flow View**
5. **Operational View**
6. **Security View**
7. **Decision Log (ADR set)**

### 6.3 Default architecture style

Default architectural posture for Licorsy products:

- **Modular monolith first**
- **API-first**
- **Managed services first**
- **Event-driven only when justified**
- **Observability by default**
- **Infrastructure as Code mandatory**
- **Explicit threat modeling for internet-facing systems**

### 6.4 Standard branch model

Licorsy standard branch progression:

`feat/*` -> `develop` -> `staging` -> `main`

Rules:
- `develop`: integration branch
- `staging`: pre-production validation branch
- `main`: production release branch

The full prefix taxonomy (`feat`, `fix`, `refactor`, `docs`, `chore`, `hotfix`)
and the merge-permission matrix are owned by `git-governance`, not restated here.

## 7. Standard document set

### 7.1 Required for organization-level governance

In `.github`:
- `GOVERNANCE.md`
- `ARCHITECTURE-PRINCIPLES.md`
- `ENGINEERING-STANDARDS.md`
- `REPOSITORY-CLASSIFICATION.md`
- `SECURITY-BASELINE.md`

### 7.2 Required for every product repository

At minimum:
- `README.md`
- `CLAUDE.md`
- `AGENTS.md` (if applicable)
- `catalog-info.yaml`
- `docs/adr/`
- `docs/architecture/`
- `docs/operations/`
- `docs/status/`
- `docs/risks/`
- `CHANGELOG.md`

### 7.3 Required architecture documents

Under `docs/architecture/`:
- `system-context.md`
- `containers.md`
- `deployment.md`
- `security.md`
- `data-flow.md`

### 7.4 Required operations documents

Under `docs/operations/`:
- `runbook.md`
- `incident-management.md`
- `backup-recovery.md`
- `observability.md`
- `release-process.md`

## 8. Engineering workflow standard

Licorsy uses a **Lean + Spec-Driven Development** delivery model.

### 8.1 Delivery flow

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

### 8.2 Required gates

Before coding:
- approved PRD
- approved architecture direction
- approved ADRs for major choices
- implementation spec available

Before merge to `develop`:
- tests pass
- lint passes
- pre-commit and commit-message checks pass; the merge itself is autonomous,
  zero required approvals by design — see `AGENTS.md`, "Merge policy", for why.
  Documentation governance is not a `develop`-merge gate in most repositories:
  it runs remotely only on `staging`/`main` promotion PRs (see AGENTS.md,
  "Remote validation layer"), and locally only where the optional
  `docgov-changed` pre-commit hook is actually installed.

Before merge to `staging`:
- integration checks pass
- security checks pass
- release checklist prepared

Before merge to `main`:
- staging validated
- release notes prepared
- rollback path documented
- production observability ready

## 9. Security and reliability baseline

Every serious project must adopt:

- branch protection and rulesets;
- dependency review;
- secret scanning;
- structured logging;
- tracing correlation IDs;
- OpenTelemetry instrumentation;
- backup strategy;
- production smoke tests;
- incident severity model;
- blameless RCA process.

For regulated or sensitive systems, add:
- threat model
- data classification
- least privilege IAM
- encryption at rest and in transit
- audit logging

## 10. Recommended tooling strategy

### 10.1 Use before build

Licorsy should prefer existing tools before creating internal replacements.

#### Source control and delivery
- GitHub
- GitHub Actions
- reusable workflows from `platform-workflows`

#### Documentation and architecture
- Markdown
- Mermaid
- Structurizr Lite or diagrams-as-code
- ADR templates
- arc42-lite templates

#### Observability
- OpenTelemetry
- Prometheus
- Grafana
- Loki
- Tempo or Jaeger
- CloudWatch (AWS-native baseline)

#### Security
- Gitleaks
- OWASP ZAP
- Semgrep or SonarQube Community
- Dependency-Track or dependency-review-action
- Snyk only when budget and value justify it

#### Testing and quality
- pytest / JUnit / Vitest depending on stack
- coverage tooling
- Allure Report
- k6 for performance
- Playwright for end-to-end tests

#### Infrastructure
- AWS CDK or Terraform
- AWS Secrets Manager
- Docker
- local Docker Compose for developer environments

#### AI engineering
- Claude Code
- MCP
- LangSmith or Langfuse
- RAG pipeline only when retrieval is clearly justified

### 10.2 Build internally only when

Licorsy should build internal tools only when:
- the capability is core to its competitive differentiation;
- market tools cannot express the governance or workflow needed;
- ongoing maintenance cost is lower than tool sprawl or SaaS cost.

## 11. Resource and capability management

Licorsy should maintain a living **Resource Inventory** sourced from
`state/resources.md` in the private product repository that owns it.

This inventory must classify:
- budget and runway;
- cloud credits and subscriptions;
- devices and environments;
- learning assets and certifications;
- reusable code assets;
- domain assets (organization, domain, branding);
- stakeholder and advisor assets;
- operational constraints.

Required output artifact:
- `docs/resources/resource-inventory.md`, in that same repository

The source `state/resources.md` already exists and is actively maintained; the
artifact above is the derived, publishable view and does not exist yet.

Recommended sections:
1. Financial runway
2. Tool subscriptions
3. Cloud accounts and credits
4. Hardware and local environments
5. Learning assets
6. Reusable engineering assets
7. Constraints and risks
8. Activation plan by quarter

## 12. Maturity model

### Level 1 — Structured Solo Engineering
Characteristics:
- template-driven delivery;
- docs and Git governance active;
- standard branch flow;
- architecture decisions documented.

### Level 2 — Platformized Delivery
Characteristics:
- reusable workflows active;
- org-wide standards stable;
- observability baseline active;
- release automation operational;
- service catalog present.

### Level 3 — Professional Software Organization
Characteristics:
- measurable delivery and reliability metrics;
- repeatable project onboarding;
- documented architecture and operations baselines;
- portfolio-grade public capabilities;
- client-ready security and delivery posture.

## 13. First 90-day priorities

### 0–30 days
- establish `.github`
- establish `platform-workflows`
- finalize repository classification
- separate overlapping responsibilities
- rename `hom` to `staging` across standards

### 31–60 days
- upgrade `ai-assisted-sdd-template` to consume platform capabilities only
- add architecture document kit
- add observability baseline kit
- add security baseline kit
- create `resource-inventory.md` from `state/resources.md` (Section 11)

### 61–90 days
- onboard at least one real product using the full model
- publish one public reference architecture
- activate release automation and basic SLO reporting
- measure lead time, deployment frequency, and incident learnings

## 14. Governance cadence

This section defines the **review** cadence — how often the organization looks at
itself. The **delivery** cadence, how often work is promoted from `develop` to
`main`, is a different policy and is owned by `AGENTS.md` under "Promotion
cadence", not restated here.

### Weekly
- execution review
- blocker review
- documentation drift review
- active project status review

### Monthly
- platform roadmap review
- cost and tooling review
- security baseline review
- architecture quality review

### Quarterly
- blueprint review
- maturity assessment
- public/private repository strategy review
- capability investment review

## 15. Success criteria

This blueprint is successful when:
- new projects can start without ad hoc setup;
- standards are reusable rather than copied;
- architectural decisions are understandable by outsiders;
- products can be operated with confidence;
- the organization demonstrates portfolio-quality engineering maturity.

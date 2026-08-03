# Licorsy — Life Core Systems

**An AI-first software engineering organization.** Licorsy builds internal engineering capabilities as reusable platform products, and delivers production-grade software using Spec-Driven Development.

It exists for two purposes at once: as the foundation of a software business, and as a working portfolio for Applied AI Engineering. The full target operating model is in [the organizational blueprint](https://github.com/licorsy/.github/blob/main/docs/licorsy-organizational-blueprint.md).

## How we work

- **Platform before repetition.** Any repeated engineering behavior becomes a reusable capability, not a manual habit.
- **Documentation is part of the system.** Architecture, process, and operational decisions live in version control and are reviewable.
- **Standards use market language.** Established industry concepts and names, wherever they exist.
- **Managed services first.** Prefer managed cloud services over operational complexity built in-house.
- **Security by default.** Secrets, dependencies, and deployments are governed by default, not by memory.
- **Architecture decisions are explicit.** Significant trade-offs are documented as ADRs, with rationale and consequences.
- **Golden paths over unlimited flexibility.** New projects start from approved templates and reusable workflows.
- **Public framework, private product by default.** Frameworks, templates, and engineering capabilities may be public; products and client systems stay private until deliberately published.

## Repositories

### Method

**[ai-assisted-sdd-template](https://github.com/licorsy/ai-assisted-sdd-template)** — AI-assisted, Spec-Driven Development template: governance, phased roadmap, documentation metadata standard, and agent workflows for Claude Code. The starting point for a new project in the organization.

### Platform capabilities

**[docs-governance](https://github.com/licorsy/docs-governance)** — a deterministic engine for documentation consistency, parameterized per repository by a `.docgov.config.js`. Ships as a CLI, a composite GitHub Action, a pre-commit hook, and a Claude Code plugin with review subagents. Checks frontmatter, internal links, changelog retention, declared counts, and cross-document sync — mechanically, before any model is asked to read anything.

**[git-governance](https://github.com/licorsy/git-governance)** — portable branch and merge governance: a branch/permission taxonomy, promotion commands, and an idempotent script that configures real GitHub rulesets. Installable in any repository.

**[platform-workflows](https://github.com/licorsy/platform-workflows)** — platform-level CI/CD and workflow automation shared across the organization's repositories.

### Organization

**[.github](https://github.com/licorsy/.github)** — organization profile, community health files, and the shared governance documents: engineering standards, architecture principles, and repository classification.

## Getting oriented

Start with [`ENGINEERING-STANDARDS.md`](https://github.com/licorsy/.github/blob/main/ENGINEERING-STANDARDS.md) and [`ARCHITECTURE-PRINCIPLES.md`](https://github.com/licorsy/.github/blob/main/ARCHITECTURE-PRINCIPLES.md) for how code and architecture are expected to look, and [`GOVERNANCE.md`](https://github.com/licorsy/.github/blob/main/GOVERNANCE.md) for how decisions are made.

---

Maintained by [Alexandre Clemente](https://github.com/aleclemente) — Applied AI Engineer. More at [aleclemente.github.io](https://aleclemente.github.io).

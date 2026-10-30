# regulation-ledger

Open-source compliance management system with adaptive forms, versioned regulatory
templates, and auditable evidence tracking for multi-jurisdiction regulatory compliance.

[Español](README.es.md)

---

## About

`regulation-ledger` solves a concrete problem: companies face overwhelming regulatory forms that don't match their actual profile, wrong size, risk class, or sector, leading to confusion, non-compliance, and wasted resources.

### Why this project matters

1. **Adaptive templates replace one-size-fits-all.** A 5-employee microbusiness and a 500-employee construction company have radically different compliance needs. `regulation-ledger` generates a minimal, relevant form set based on company profile.

2. **The registry is the product.** Forms live in a versioned JSON Schema registry, extensible by platform owners and by MSPs managing multiple clients. Normative changes are tracked as snapshots, so a form always states which version of which rule it was built against.

3. **Compliance data stays on-premise.** Self-hosted by design. For sectors with strict data residency requirements or organizations that require full control of their evidence trail.

4. **Evidence is verifiable, not just stored.** Every record is chained by hash, so an auditor can reconstruct what was filed, when, and against which normative version.

## Architecture
```
┌──────────────────────────────────────────────────────────────┐
│  Frontend  (Vue 3 + TypeScript + Pinia + JSONForms)          │
├──────────────────────────────────────────────────────────────┤
│  Backend   (Node.js + Fastify + Prisma ORM)                   │
├──────────────────────────────────────────────────────────────┤
│  Database (PostgreSQL + JSONB for form schemas)              │
├──────────────────────────────────────────────────────────────┤
│  Registry  (versioned JSON Schema + normative snapshots)     │
├──────────────────────────────────────────────────────────────┤
│  Evidence  (hash-chained, append-only trail)                 │
├──────────────────────────────────────────────────────────────┤
│  MSP Connector  (REST API + MCP for external tooling)        │
└──────────────────────────────────────────────────────────────┘
```

### Core layers

- **Form Builder Engine**, Versioned JSON Schema registry with categories and applicability rules (risk class, sector, employee count, jurisdiction). Locked forms protect mandatory regulatory requirements.
- **Template Engine**, Profile-driven menu generation. Company profile -> applicable template -> minimal relevant form set. No overload.
- **Normative snapshots**, Each rule version is captured as an immutable snapshot. When a norm changes, affected forms are flagged instead of silently drifting.
- **Expiration Alerts**, Automatic expiration notifications for documents, certifications, and periodic renewals. Email + dashboard.
- **Evidence chain**, Hash-chained, append-only record of submissions and supporting documents.
- **MSP Connector**, REST API + MCP server for MSPs managing multiple client companies. Bulk operations, dashboards, external integration.

The deterministic layers above are the core of the project. An optional, local, off-by-default assistant is available as a separate extension for form-authoring assistance; it is not required to run the system and is not part of the core roadmap.

## Core concept

Forms follow company profile:
```
Company Profile {
  risk_class: "III"
  sector: "construction"     // CIIU code
  employees: 45
  jurisdiction: "CO"
}
  → Template: "large-enterprise-construction"
  → Forms: 18 relevant forms (from ~30 total registry)
  → Menu: only what applies
```

## Documentation

Full technical documentation lives in [`docs/en/`](docs/en/):

- [Technical specification](docs/en/SPEC.md), problem, architecture, stack, phases and closed decisions.
- [Architecture](docs/en/architecture.md), the four layers and their API surface.
- [Roadmap](docs/en/roadmap.md), phases by month with deliverables and success criteria.

Documentación en español: [`docs/es/`](docs/es/).

## License

**Mozilla Public License 2.0 (MPL-2.0).** Permits commercial use, modification, and redistribution. Same license as Mozilla Firefox.

## Development and transparency

**Past use of generative AI.** The technical specification and the documentation in this
repository were drafted with AI assistance before this project adopted a written policy
on generative AI. That assistance is disclosed here for the record, and the underlying
content was reviewed and is owned by the maintainer.

**Going forward.** New code and documentation are written by hand. Generative AI is not
used to produce code, design, or documentation in this repository. Where AI tooling is
used for read-only review (for example, static analysis of existing code), it is
disclosed per-commit in the commit message.

Contributors are asked to state in their commit message whether any part of the change
was machine-generated, and to own the result of their work.

## Optional assistant extension

An optional local assistant for form-authoring assistance exists as a separate
extension: it runs off by default, requires a locally hosted model, and is not required
to operate any core layer. The core layers, registry, template engine, normative
snapshots, evidence chain, MSP connector, are fully deterministic and run without it.

## Status

- Repository: active development (October 2026)
- Production: no
- Funding: pursuing open-source grants and community sponsorship (details in maintainer's project roadmap)
- Maintainer: [Yaqui Martinez](https://github.com/mono079790-lab) (persona natural, Colombia)

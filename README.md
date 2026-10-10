# regulation-ledger

Open-source compliance management system with adaptive forms, regulatory templates, and AI-assisted evidence tracking for multi-jurisdiction regulatory compliance.

[Español](README.es.md)

---

## About

`regulation-ledger` solves a concrete problem: companies face overwhelming regulatory forms that don't match their actual profile — wrong size, risk class, or sector — leading to confusion, non-compliance, and wasted resources.

### Why this project matters

1. **Adaptive templates replace one-size-fits-all.** A 5-employee microbusiness and a 500-employee construction company have radically different compliance needs. `regulation-ledger` generates a minimal, relevant form set based on company profile.

2. **The registry is the product.** Forms live in a versioned JSON Schema registry, extensible by platform owners, MSPs managing multiple clients, and AI agents that generate new forms from raw regulatory text.

3. **Compliance data stays on-premise.** Self-hosted by design. For sectors with strict data residency requirements or organizations that require full control of their evidence trail.

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│  Frontend  (Vue 3 + TypeScript + Pinia + JSONForms)          │
├──────────────────────────────────────────────────────────────┤
│  Backend  (Node.js + Fastify + Prisma ORM)                   │
├──────────────────────────────────────────────────────────────┤
│  Database  (PostgreSQL + JSONB for form schemas)             │
├──────────────────────────────────────────────────────────────┤
│  AI Agent  (DSPy + configurable LLM)                         │
├──────────────────────────────────────────────────────────────┤
│  MSP Connector  (REST API + MCP for external agents)          │
└──────────────────────────────────────────────────────────────┘
```

### Core layers

- **Form Builder Engine** — Versioned JSON Schema registry with categories and applicability rules (risk class, sector, employee count, jurisdiction). Locked forms protect mandatory regulatory requirements.
- **Template Engine** — Profile-driven menu generation. Company profile → applicable template → minimal relevant form set. No overload.
- **Expiration Alerts** — Automatic expiration notifications for documents, certifications, and periodic renewals. Email + dashboard.
- **AI Agent** — Generates forms from regulatory text. Assists during form filling. Validates cross-norm compliance when regulations change.
- **MSP Connector** — REST API + MCP server for MSPs managing multiple client companies. Bulk operations, dashboards, external AI agent integration.

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

Full technical documentation: [`docs/en/`](docs/en/)

## License

**Mozilla Public License 2.0 (MPL-2.0).** Permits commercial use, modification, and redistribution. Same license as Mozilla Firefox.

## Status

- Repository: active development (October 2026)
- Production: no
- Funding: pursuing open-source grants and community sponsorship (details in maintainer's project roadmap)
- Maintainer: [Yaqui Martinez](https://github.com/mono079790-lab) (persona natural, Colombia)

# Roadmap

This document details the 6-month roadmap for `regulation-ledger`. Each milestone has verifiable success criteria.

## H1: evidence-core v0.1 (Month 1 — October 2026)

**Goal:** Hash chain engine for immutable records.

**Deliverables:**
- TypeScript library with strict typing
- `EvidenceRecord` data model
- `appendEvidence()` with prev_hash validation
- `verifyChain()` with tampering detection
- `exportAuditTrail()` in JSON and CSV
- Unit tests with >80% coverage
- README with usage examples

**Success criteria:**
- `npm run test` passes with >80% coverage
- README example runs end-to-end
- API surface matches `docs/architecture.md` Layer 1

## H2: regulation-snapshot v0.1 (Month 2 — November 2026)

**Goal:** Versioned legal constants registry.

**Deliverables:**
- `LegalConstant` data model
- `registerConstant()` with overlap detection
- `getConstantsForDate()` with timezone handling
- `getConstantHistory()` for audit purposes
- Seed data: 5 key Colombian SST norms (Resolución 0312/2019, Decreto 1072/2015, Ley 1562/2012, Decreto 1295/1994, Circular 008/2019)
- Unit tests + integration tests with `evidence-core`
- README with examples

**Success criteria:**
- For dates 2020-2025, `getConstantsForDate()` returns the correct norm in force on that date
- Integration with `evidence-core` works end-to-end
- Seed data validates against `LegalConstant` schema

## H3: integration + compliance-validate v0.1 (Month 3 — December 2026)

**Goal:** Rule engine + library integration.

**Deliverables:**
- YAML/JSON rule parser
- 5 pre-loaded SST validation rules (training date validity, medical exam periodicity, committee composition, etc.)
- CLI for running validations: `regulation-ledger validate <record> <rules>`
- Integration tests covering all 4 modules together
- README with rule authoring guide

**Success criteria:**
- 5 sample rules execute correctly via CLI
- Integration tests pass
- Rule authoring guide is clear enough for a non-author to write a new rule

## H4: ledger-api v0.1 (Month 4 — January 2027)

**Goal:** HTTP API + Docker deployment.

**Deliverables:**
- Fastify server with all endpoints from `docs/architecture.md` Layer 4
- OpenAPI 3.0 spec auto-generated
- Docker compose for on-premise deploy
- Optional API key authentication
- Integration tests via HTTP

**Success criteria:**
- `docker compose up` brings up the API and persistent storage
- 3 endpoints documented in OpenAPI spec
- API tests pass via HTTP client
- Response time <100ms for single-record operations

## H5: docs + landing + demo (Month 5 — February 2027)

**Goal:** Public-facing materials.

**Deliverables:**
- Bilingual README (Spanish + English)
- Technical documentation generated from TypeScript types
- Static landing page at `regulation-ledger.org`
- Public demo deployed to a free hosting service (fly.io, Railway, or similar)
- 5-minute demo video showing the use case

**Success criteria:**
- Landing page accessible at `regulation-ledger.org`
- Demo allows anonymous users to try basic operations
- Demo video published and linked from README
- All docs available in Spanish and English

## H6: public release + community (Month 6 — March 2027)

**Goal:** Production-ready v1.0 + community onboarding.

**Deliverables:**
- GitHub v1.0 release
- `CONTRIBUTING.md` with development setup
- `CODE_OF_CONDUCT.md`
- `SECURITY.md` with vulnerability disclosure policy
- Issue templates (bug report, feature request, question)
- First external issue processed end-to-end
- Blog post published on dev.to or similar
- Submission to awesome-open-source lists relevant to compliance

**Success criteria:**
- GitHub v1.0 release tag exists
- At least 1 external issue has been responded to and resolved
- README links to published blog post
- Project appears in at least 1 awesome-open-source list

## Stretch goals (post-grant)

- Web UI for non-technical users (form-based record creation)
- Integration with government APIs (e.g. Colombian Ministerio del Trabajo)
- Multi-language support beyond Spanish/English
- Compliance templates for additional regulatory domains (RGPD, SOX, HIPAA)
- Optional cloud mode for organizations without on-premise capability

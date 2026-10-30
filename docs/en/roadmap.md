# Roadmap — Regulation Ledger

## Core strategy

Development is driven by the **Form Builder Engine** (versioned JSON Schema registry) and
**company templates** (adaptive menu derived from the company profile).

---

## Phase 1: Foundation — Month 1-2

**Goal:** a working Form Builder Engine with a JSON Schema registry.

### Deliverables
- Prisma schema: `Form`, `FormVersion`, `FormCategory`, `Company`, `CompanyProfile`
- REST API: CRUD for forms and versions
- JSONForms Vue renderer connected to the registry
- Category and tag system per form
- Locked forms (not deletable, only versionable)

### Success criterion
A dynamic form rendered from JSON Schema in Vue 3 with no new code per form.

---

## Phase 2: Company Profiles + Wizard — Month 2-3

**Goal:** a setup wizard that generates the adaptive menu.

### Deliverables
- 7-step wizard (company, sites, areas, responsible parties, classification)
- CompanyProfile: `risk_class`, `ciiu_sector`, `employee_count`, `jurisdiction`
- Template Engine: profile → applicable template → form menu
- CRUD for companies and profiles
- Per-company permissions and roles

### Success criterion
Company created → the menu shows only the forms that apply to its profile.

---

## Phase 3: Expiration system — Month 3-4

**Goal:** automatic alerts for document expiration.

### Deliverables
- Table of regulatory frequencies (annual, monthly, quarterly)
- Cron job that computes upcoming expirations
- Email notifications
- Dashboard of pending expirations
- Renewal history

### Success criterion
The system alerts before a document or certification expires.

---

## Phase 4: Evidence and regulatory snapshots — Month 4-6

**Goal:** a verifiable evidence trail and control over regulatory validity.

### Deliverables
- Append-only hash chain over submissions and supporting documents
- Plain JSON export: the full history is readable without the application
- Immutable snapshot per norm version
- Detection of affected forms when a norm changes
- Cross revalidation of rules when the norm changes

### Success criterion
An auditor can reconstruct what was submitted, when, and against which version of the
norm, using only the export.

---

## Phase 5: MSP connector — Month 7-12

**Goal:** an API for MSPs managing multiple client companies.

### Deliverables
- REST API with API key authentication
- MCP server for external integrations
- MSP dashboard: all client companies, aggregated metrics
- Bulk operations (create company, assign template, import data)
- Webhooks for external notifications

### Success criterion
An MSP manages 50 companies from a single API.

---

## Roadmap milestones

| Date | Milestone |
|------|-----------|
| month 2 | Working Form Builder + wizard |
| month 4 | Expirations + regulatory snapshots |
| month 6 | Evidence chain + export |
| month 12 | MSP connector + public demo |

## Optional extension: authoring assistance

Outside the main path and with no dependency on the core layers, there is an optional
extension providing a local assistant for form authoring: it runs disabled by default,
requires a locally hosted model, and is not needed to operate any core layer. It is
developed as a community contribution, not as part of the phases above.

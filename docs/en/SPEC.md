# Regulation Ledger — Technical Specification

> Version: 0.2.0
> Date: 10-oct-2026
> Changelog v0.2: Architectural decisions closed (section 5) and section 6 status updated.
> Author: Yaqui Martinez
> Status: Public technical specification

---

## 1. Context and Problem

### 1.1 The problem with current SG-SST tools

Current SG-SST tools on the market present excessive cognitive load for end users. Companies receive tools with too many irrelevant modules for their specific profile, generating:

- **Excessive cognitive load** → frustration → system abandonment
- **Irrelevant modules** that don't apply based on company size, sector, or risk class
- **Operational overload** without tangible reduction in actual risk
- **Confusing experience** for users without prior regulatory knowledge

The correct architecture starts from: company profile → templates → minimal viable menu.

### 1.2 What Colombian law actually requires

Not 113 forms. Approximately 30 unique forms organized by the PHVA cycle (Decreto 1072/2015 + Resolución 0312/2019):

| Category | Unique forms | Base regulation |
|----------|-------------|-----------------|
| COMPLIANCE | 5 | Res. 0312/2019 |
| DOCUMENTS | 4 | Decreto 1072/2015 |
| RISKS | 3 | GTC 45, Res. 1409/2012 |
| ROAD SAFETY | 2 | Res. 1565/2014, Res. 7495/2020 |
| PEOPLE | 2 | Decreto 1072/2015 |
| PPE | 1 | Decreto 1072/2015 |
| TRAINING | 2 | Res. 0312/2019 |
| OPERATIONS | 3 | Res. 1401/2007, Res. 0312/2019 |
| COMMITTEES | 1 | Res. 1918/2009 |
| COMMUNICATION | 1 | Decreto 1072/2015 |
| ENVIRONMENTAL | 2 | Decreto 1072/2015 |
| EMERGENCIES | 2 | Decreto 1072/2015 |
| CONFIGURATION | 2 | Decreto 1072/2015 |

**Real total: ~30 unique forms.** The difference is views, not forms.

### 1.3 Expiration dates and renewals (from regulatory analysis)

| Document | Periodicity | Legal framework |
|----------|-------------|----------------|
| SST Policy | 1 year (update) | Res. 0312/2019 Art. 4 |
| Annual work plan | December (following year) | Res. 0312/2019 Art. 26 |
| Minimum standards self-assessment | December (annual) | Res. 0312/2019 Art. 26 |
| Legal matrix | Continuous update | Decreto 1072/2015 |
| GTC 45 Risk matrix | Update on changes | GTC 45:2012 |
| Occupational medical evaluation | 1 year | Decreto 1072/2015 |
| Safety inspections | Periodicity by risk level | Res. 0312/2019 |
| Fire extinguisher checks | Quarterly / semiannual | NFPA |
| Internal SG-SST audit | 1 year | Res. 0312/2019 |
| Training sessions | 1 year (schedule) | Res. 0312/2019 |
| COPASST conformation | Annual renewal | Res. 1918/2009 |
| Accident reports (FURAT) | Immediate / 2 days | Res. 1401/2007 |
| Emergency plan | Annual update | Decreto 1072/2015 |

---

## 2. System Architecture

### 2.1 Core philosophy

> **The product is not the form. It is the registry + template engine + AI agent.**
> Forms are a consequence, not the product.

Three independent layers:

```
┌─────────────────────────────────────────────────┐
│  AI AGENT — Construction and assistance          │
├─────────────────────────────────────────────────┤
│  TEMPLATE ENGINE — Adaptive menu by profile      │
├─────────────────────────────────────────────────┤
│  FORM BUILDER ENGINE — Form registry            │
│  (pure data, no business logic)                │
└─────────────────────────────────────────────────┘
```

### 2.2 Block 1 — Form Builder Engine

**Responsibility:** Store and serve form definitions as pure JSON Schema.

**Technology base:**
- **JSONForms Vue** — JSON Schema standard renderer for Vue 3
- **Storage:** PostgreSQL with JSONB for schemas

**Do not reinvent the wheel.** Use an established library and store schemas in the DB.

#### Form registry

```typescript
interface FormDefinition {
  id: string;                    // UUID
  slug: string;                  // "initial-evaluation-sst"
  name: string;                 // "Initial SG-SST Evaluation"
  description: string;

  category: string;             // "compliance"
  subcategory?: string;         // "evaluation"

  locked: boolean;              // true = legal requirement (cannot be deleted)
  editable: boolean;            // true = company can modify

  applicable_to: {
    risk_classes: ('I'|'II'|'III'|'IV'|'V')[];
    sectors: string[];          // CIIU code or "all"
    min_employees: number;
    max_employees?: number;
    jurisdiction: string[];     // ["CO", "MX", "PE"]
  };

  json_schema: object;          // JSON Schema of the form
  ui_schema?: object;           // UI Schema (layout)
  regulation_refs: string[];    // ["Resolution 0312 Art. 2.2.4.6.5"]
  expiration: {                // Alert config
    periodicity: 'monthly'|'quarterly'|'semiannual'|'annual'|'eventual';
    expiration_month?: number;  // 1-12
    alert_days?: number;        // days before expiration
  };

  version: number;
  created_at: string;
  updated_at: string;
  created_by: string;           // 'system' | user_id | 'ai_agent'
}
```

#### Categories registry

```typescript
interface Category {
  id: string;
  slug: string;                 // "compliance"
  name: string;                // "Legal Compliance"
  description: string;
  icon: string;                // emoji or icon name
  order: number;               // menu order
  color?: string;              // theme color
  default_templates: string[]; // which templates include it by default
}
```

**Standard categories (CO):**
- compliance
- documents
- risks
- road-safety
- people
- ppe
- training
- operations
- committees
- communication
- environmental
- emergencies
- configuration

### 2.3 Block 2 — Wizard + Template Engine

**Responsibility:** Generate the applicable form menu based on the company profile.

#### Company profile

```typescript
interface CompanyProfile {
  id: string;
  name: string;
  nit: string;                  // tax ID

  // Data that determines applicable forms
  risk_class: 'I'|'II'|'III'|'IV'|'V';
  sector: string;               // CIIU code
  num_employees: number;
  city: string;
  jurisdiction: string;         // "CO" default

  // Wizard config
  wizard_completed: boolean;
  wizard_template?: string;     // template_id used
  wizard_data?: object;         // collected data

  // Menu config
  enabled_forms: string[];      // active form_definition_ids
  disabled_forms: string[];     // disabled form_definition_ids
  custom_forms: FormDefinition[]; // forms created by the company

  created_at: string;
  updated_at: string;
}
```

#### Templates

```typescript
interface Template {
  id: string;
  slug: string;                 // "sme-risk-i-ii"
  name: string;                // "SME Risk I-II"
  description: string;

  // Profile that activates this template
  applies_to: {
    risk_classes: ('I'|'II'|'III'|'IV'|'V')[];
    sectors?: string[];
    employee_range?: [number, number];
  };

  // Forms included in this template
  include_forms: string[];     // form_definition_ids
  exclude_forms: string[];     // form_definition_ids (exceptions)
  suggested_order: string[];    // form_definition_ids in completion order

  // Wizard steps for this template
  wizard_steps: WizardStep[];

  created_by: 'system' | user_id;
  is_public: boolean;           // visible to other users
}

interface WizardStep {
  id: string;
  order: number;
  title: string;
  description: string;
  fields: object[];            // JSON Schema subset for this step
  completed: boolean;
}
```

**Default templates (CO):**

| Template | Profile | Forms |
|----------|---------|-------|
| microenterprise | risk I-II, 1-5 employees | ~12 forms |
| sme | risk I-III, 6-50 employees | ~20 forms |
| large-enterprise | risk III-V, 51+ employees | ~30 forms |
| construction | any risk, construction sector | +Full PESV Road Safety |
| health | any risk, health sector | +Biosafety |
| manufacturing | any risk, manufacturing sector | +Chemicals |
| industrial | risk IV-V, industrial sector | +All |

#### Template engine

```typescript
function getApplicableTemplate(profile: CompanyProfile): Template {
  // 1. Search exact match by profile
  // 2. If no exact match, search by risk_class
  // 3. If none, use "sme" as fallback
  // Returns template with resolved forms
}

function getFormsForProfile(profile: CompanyProfile): FormDefinition[] {
  const template = getApplicableTemplate(profile);
  const locked = getLockedForms();  // always present
  const applicable = template.include_forms
    .filter(id => !template.exclude_forms.includes(id))
    .concat(locked);
  return applicable.map(id => getFormDefinition(id));
}
```

### 2.4 Block 3 — Expiration Alerts System

**Responsibility:** Notify before a document expires.

```typescript
interface Expiration {
  id: string;
  company_id: string;
  form_definition_id: string;
  expiration_date: string;      // ISO date
  alert_date: string;          // date when alert is sent
  status: 'pending'|'alert_sent'|'expired'|'renewed';
  last_submission_id?: string;  // last submission ID
  alert_sent_at?: string;
  renewed_at?: string;
}

// Daily job running at midnight
async function checkExpirations() {
  const tomorrow = addDays(today, 1);
  const due = await db.expirations.findMany({
    where: {
      status: 'pending',
      alert_date: { lte: tomorrow }
    }
  });

  for (const e of due) {
    await sendAlert(e);
    await db.expirations.update(e.id, {
      status: 'alert_sent',
      alert_sent_at: now()
    });
  }
}
```

**Alert types:**
- **Email** — to form responsible parties
- **Dashboard** — badge in menu
- **Push notification** (future)
- **Weekly summary** (future)

### 2.5 Block 4 (optional extension) — Authoring assistant

**Responsibility:** Build new forms, assist the user, validate cross-regulatory compliance.

> **Scope.** This block is an optional extension, off by default and not required to
> operate any core layer. Layers 1-3 and 5 (registry, template engine,
> snapshots/expirations, evidence and MSP connector) are deterministic and work without
> it. It is developed independently of the main path.

#### 4a. Form generation from regulation

```typescript
interface AIAgentFormGeneration {
  // Input: norm text or PDF
  // Output: FormDefinition ready to store in registry

  async function generateFormFromNorm(
    normText: string,
    metadata: { category: string; jurisdiction: string }
  ): Promise<FormDefinition>;

  // Steps:
  // 1. Parse text → extract fields, types, validations
  // 2. Identify legal references → regulation_refs
  // 3. Classify → category, applicable_to
  // 4. Generate JSON Schema
  // 5. Save to registry as draft
  // 6. Notify admin for review
}
```

**Agent tools:**
- `web_search` — search for updated regulation texts
- `document_analysis` — extract structure from regulatory PDFs
- `json_schema_generator` — generate schema from structured text

#### 4b. Form-filling assistance

```typescript
interface AIAssistanceRequest {
  form_id: string;
  current_field: string;
  user_context: {
    company: CompanyProfile;
    profile_completed: boolean;
  };
}

async function assistFieldFilling(req: AIAssistanceRequest): Promise<{
  explanation: string;          // what the field asks in plain language
  evidence_hint: string;         // what evidence is typically uploaded
  related_fields: string[];      // related fields in other forms
  regulation_excerpt: string;    // relevant regulation text
}>;
```

#### 4c. Cross-regulation validation

```typescript
// When a regulation changes, the agent re-evaluates
async function revalidateFormsOnRegulationChange(
  regulationId: string,
  newVersionText: string
): Promise<{
  affected_forms: string[];
  proposed_changes: FormDefinition[];
  breaking_changes: string[];   // fields removed or modified
}>;
```

#### 4d. Configurable AI engine

The agent uses a configurable LLM — not tied to a specific provider.

```typescript
interface LLMConfig {
  provider: 'openai' | 'anthropic' | 'ollama' | 'custom';
  model: string;
  api_key?: string;             // for cloud
  base_url?: string;            // for self-hosted (Ollama)
}
```

**Decision:** Use **DSPy** (declarative LM programs) as abstraction. Allows changing models without changing code.

### 2.6 Block 5 — MSP / Multi-tenant Architecture

**Responsibility:** Allow an MSP to manage multiple client companies from a single instance, with isolated data and optional white-labeling.

#### Multi-tenancy

```typescript
interface Tenant {
  id: string;                   // tenant_id
  name: string;
  slug: string;                 // subdomain

  // Data isolation
  schema_name: string;           // PostgreSQL schema per tenant
                                  // Or: tenant_id field in each table

  // White-label branding
  branding?: {
    logo_url?: string;
    primary_color?: string;
    favicon_url?: string;
    custom_domain?: string;      // CNAME
  };

  // MSP config
  is_msp: boolean;               // true = is an MSP
  msp_id?: string;              // if client of an MSP, which one

  // Plan limits
  plan: 'starter'|'professional'|'enterprise';
  max_companies?: number;       // for MSP
  max_users?: number;
  features_enabled: string[];

  created_at: string;
  updated_at: string;
}
```

#### Multi-tenant data model

**Chosen option: Shared schema with tenant_id**

```
// Advantage: simpler to operate
// Disadvantage: must always filter by tenant_id
// Advantage outweighs disadvantage for this use case

Tables with tenant_id:
  - companies
  - company_forms (form assignments per company)
  - form_submissions (form data)
  - expirations
  - alerts

Tables without tenant_id (platform owner):
  - form_definitions (global registry)
  - categories
  - templates
  - tenant (MSP config)
```

#### MSP Connector — API for external agents

```typescript
interface MSPConnector {
  // An MSP (or external agent) can create/manage companies
  // via API with their own authentication

  createCompany(data: {
    name: string;
    nit: string;
    risk_class: string;
    sector: string;
    num_employees: number;
    wizard_template?: string;
  }): Promise<CompanyProfile>;

  bulkCreateCompanies(csv: Buffer): Promise<{
    created: number;
    failed: number;
    errors: string[];
  }>;

  getCompanyStatus(companyId: string): Promise<{
    forms_completed: number;
    forms_pending: number;
    upcoming_expirations: number;
    compliance_score: number;
  }>;

  generateReport(companyId: string, format: 'pdf'|'xlsx'): Promise<Buffer>;

  // Also for external AI agent
  submitForm(companyId: string, formId: string, data: object): Promise<string>;
  getFormStatus(companyId: string, formId: string): Promise<string>;
}
```

#### External agent via MCP

```
regulation-ledger exposes an MCP (Model Context Protocol) server
that allows an external agent to:
  1. List managed companies
  2. View compliance status
  3. Create/modify forms
  4. Generate reports
  5. Receive expiration alerts

This is what the user called "LaserBeam" — an agent that connects
to regulation-ledger to build forms automatically.
```

---

## 3. Technology Stack

| Layer | Technology | Justification |
|-------|-----------|---------------|
| Frontend | Vue 3 + TypeScript | Small attack surface, strict typing, mature form ecosystem |
| State | Pinia | Vue 3 state management |
| Backend | Node.js + Express or Fastify | Single runtime for API and jobs, simple self-hosted deployment |
| Database | PostgreSQL + JSONB | Structured data + flexible schemas |
| ORM | Prisma | Type-safe, migrations, multi-tenant |
| Auth | Lucia | Multi-tenant with roles, self-hosted |
| Form rendering | JSONForms Vue | JSON Schema standard for Vue |
| AI | DSPy + configurable LLM | Declarative, swappable |
| API | REST + MCP (future) | MSP/AI connector |
| Hosting | Self-hosted (Docker) | User wants cash, not SaaS |
| CI/CD | GitHub Actions | Repo on GitHub |

---

## 4. Development Phases

### Phase 1 — V1.0 (Base + Minimal Wizard)
- [ ] PostgreSQL + Prisma setup with multi-tenant schema
- [ ] Form registry with 10 locked forms (critical CO)
- [ ] Categories registry
- [ ] Onboarding wizard (3 steps: company, risk, employees)
- [ ] Template engine (3 templates: micro, sme, large)
- [ ] Form renderer from JSON Schema
- [ ] Submission storage
- [ ] Auth with roles (admin, editor, viewer)

### Phase 2 — V1.1 (Templates + Expirations)
- [ ] Remaining locked forms (~30 total)
- [ ] Templates by sector (construction, health, manufacturing)
- [ ] Expiration system with email alerts
- [ ] Compliance dashboard (% completed)
- [ ] Dynamic menu from registry

### Phase 3 — V2.0 (AI Agent)
- [ ] DSPy setup with configurable LLM
- [ ] Form generation agent from regulation text
- [ ] In-form filling assistance (inline chat)
- [ ] Cross-regulation validation

### Phase 4 — V2.1 (MSP + Multi-tenant)
- [ ] MSP REST API connector
- [ ] MCP server for external agent
- [ ] White-labeling (logo, colors, domain)
- [ ] Bulk operations for MSPs

### Phase 5 — V2.2 (Multi-region)
- [ ] MX, PE jurisdiction
- [ ] International templates
- [ ] UI translations

---

## 5. Closed Architectural Decisions (v0.2)

> Decisions taken after external research. Citations included where applicable.

### 5.1 Multi-tenancy: shared schema with `tenant_id` + Row-Level Security (RLS)

**Decision:** Shared schema with `tenant_id` column in every table, **enforced by PostgreSQL RLS** (not just application convention).

**Evidence:**
- 2025-2026 consensus (Streamkap, AWS, dev.to, Supabase): "For the vast majority of modern SaaS applications, the shared-schema model using a `tenant_id` column is the way to go. It provides the clearest path to scalability and cost savings." [streamkap.com](https://streamkap.com/resources-and-guides/postgresql-cdc-multi-tenant)
- RLS in PostgreSQL is production-grade: AWS demonstrates cross-tenant writes return `UPDATE 0` when the policy blocks them, no additional code required if you don't rely on convention. [aws.amazon.com](https://aws.amazon.com/blogs/database/multi-tenant-data-isolation-with-postgresql-row-level-security/)
- Honest trade-off: schema-per-tenant produces 100K instances impossible to operate; shared schema scales, RLS protects.

**Canonical implementation:**
```sql
ALTER TABLE companies ENABLE ROW LEVEL SECURITY;
ALTER TABLE companies FORCE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON companies
  USING (tenant_id = current_setting('app.current_tenant')::uuid);
```
Every request sets `SET LOCAL app.current_tenant = '<uuid>'` before executing. Without manual exception, no leaks.

**Tables with `tenant_id`:** companies, company_forms, form_submissions, vencimientos, alerts.
**Global tables (no `tenant_id`):** form_definitions, categories, templates, form_versions (global registry, readable by all tenants).

### 5.2 Form renderer: JSONForms Vue

**Decision:** JSONForms Vue (`@jsonforms/vue`), not react-jsonschema-form (rjsf).

**Change from SPEC v0.1 recommendation:** SPEC v0.1 said "rjsf, more popular". That was incorrect: rjsf **is React-only**. The recommendation is invalidated because the stack is already defined as Vue 3.

**Evidence:**
- rjsf issue [#4482](https://github.com/rjsf-team/react-jsonschema-form/issues/4482) (official maintainers): "You may just want to build a new system called Vue Json Schema Form" — there is no official Vue version of rjsf.
- JSONForms Vue 3.8.0 (jun-2026): first-class, MIT, official Vue 3 bindings, same core as React/Angular. [eclipsesource/jsonforms](https://github.com/eclipsesource/jsonforms)
- JSONForms Community: "focuses on customizability and declarative forms. JSON Schema + UI Schema." [jsonforms discourse](https://jsonforms.discourse.group/t/compare-to-react-jsonschema-form/553)
- Comparison 2026: rjsf 900K weekly downloads (more popular), JSONForms 238K (more customizable). Both MIT. [faun.dev](https://faun.dev/toolbox/json-forms-vs-react-jsonschema-form/)

**Trade-off:** JSONForms sometimes requires custom renderers when a field doesn't fit the default sets (e.g. legal matrix fields in SST). Worth it for native Vue 3.

**Implementation:** `@jsonforms/vue` + `@jsonforms/vue-vanilla` to start, `@jsonforms/vue-vuetify` if we want Vuetify as the design system.

### 5.3 Optional assistance extension: DSPy as core, LangChain optional for tooling

**Decision:** DSPy for the internal agent (form generation, assistance, cross-validation). LangChain/LangGraph discarded as core (overhead, integration is not our bottleneck).

**Evidence:**
- DSPy is a compiler for prompts: you declare `Signature` + `Module`, the optimizer (`MIPROv2`) automatically searches for instructions and adapts to model changes. [randomcommits.com](https://randomcommits.com/dspy-vs-langchain)
- Real case (randomcommits): they switched gpt-3.5 → gpt-4-turbo and all 50 LangChain chains failed because prompts were hardcoded strings. DSPy re-compiles automatically.
- Benchmark case: 0.83 → 0.90 exact-match when migrating to DSPy in multi-category classification. Cost: $12 of compile, 30 min.
- "DSPy vs LangChain are not the same layer. DSPy optimizes the prompt given a dataset. LangChain connects providers + tools + retrievers." [tinyagents.dev](https://tinyagents.dev/vs/dspy-vs-langchain)
- Our use case = agent that generates JSON Schema from regulatory text. We have a dataset (Decree/Resolution texts), we have a metric (valid schema + covered fields). **DSPy fits perfectly.**
- LangChain would only be justified if we needed connectors to 600+ external tools. Not our case.

**Decision rule:** if in the future we want an external agent (LaserBeam/MCP) to build forms in regulation-ledger, we expose LangChain tool-calls or a simple REST API. But the internal agent is DSPy.

### 5.4 Optional extension inference: configurable cloud + self-hosted, default self-hosted for sensitive data

**Decision:** neutral architecture with **Ollama self-hosted as default**, OpenAI/Anthropic optional.

**Cost evidence (September 2026):**
| Volume | OpenAI GPT-5.6 Luna | Ollama rented (RTX 4090) | Ollama owned |
|---|---|---|---|
| 5M tokens/month | ~$2.50 | ~$6 | ~$97 (capex) |
| 100M tokens/month | ~$50 | ~$121 | ~$115 |
| 1B tokens/month | ~$800 | ~$1,207 | — |

[markaicode.com / sept 2026](https://markaicode.com/pricing/ollama-vs-openai-api-pricing)

**Reading:** Self-host rented **never wins** vs OpenAI cheapest. Self-host owned **only wins above ~100M tokens/month**. For regulation-ledger (SST audit, Colombian regulations), the bottleneck is compliance, not throughput. Use cheap cloud (GPT-5.6 Luna, $0.50/1M blended) up to ~100M tokens/month, then self-host if adoption justifies it.

**Privacy:** Ollama local = data never leaves the host. Compatible with Colombian regulations (Ley 1581 Habeas Data). OpenAI/Anthropic cloud = requires DPA + jurisdiction evaluation.

**Operational recommendation:**
1. MVP default: Ollama local with open model (Llama 3.1 8B or similar) for zero cloud dependency.
2. Enterprise client requesting large models: OpenAI/Anthropic via API, configurable in `config.yaml`.
3. Documented as "Self-hosted default, configurable to cloud" in `.env.example`.

### 5.5 Expiration alerts: daily cron polling + `vencimientos` table

**Decision:** keeps the polling job from SPEC v0.1 (line 286-302). No external webhook.

**Evidence:**
- Cron polling for recurring tasks at fixed cadence is the ideal case. "Cron is fine for recurring system tasks like cache warming, report generation, and log rotation. It runs on a fixed cadence and does its job well." [posthook.io](https://posthook.io/compare/cron-vs-durable-scheduling)
- If your work is "execute this SQL every hour at midnight", cron is the answer. Webhooks make sense when the event producer triggers the notification (doesn't apply for time-dependent expirations).
- Limitation to watch: "every concern gets its own polling query and its own cron job. At five or ten concerns, you have a growing set of pollers." If regulation-ledger reaches many alert types, reconsider. For now: 1 poller (expirations) is trivial.

**Added monitoring posture:** cron doesn't have alerting by default. If the job fails silently, nobody finds out. **Add a health-check endpoint in ledger-api that returns `last_cron_run_at`** and Prometheus metrics to alert if the job doesn't run in 24h+5min.

### 5.6 External agent: REST API first, MCP server in V2.1

**Decision:** keeps from SPEC v0.1: REST API as MSP/AI connector in V2.1, MCP server in a later phase.

**Evidence (landscape changed since oct-2025):**
- MCP spec v2025-11-25 is already stable, OAuth 2.0 Resource Server compliance, enterprise support. [modelcontextprotocol.info](https://modelcontextprotocol.info/blog/first-mcp-anniversary)
- IBM App Connect 13.0.8 production-ready: "expose your REST APIs as MCP tools without code changes". [ibm.com](https://www.ibm.com/docs/en/app-connect/13.0.x?topic=solutions-developing-mcp-tools)
- REST remains the practical initial choice: "REST first, MCP later" still holds because REST covers 100% of MSP cases, and "exposing REST as MCP tools" is trivial when desired.

**Corrected stance:** instead of "REST first, MCP later", the closed decision is **REST for V2.0 (MSP connector), MCP as thin adapter over REST in V2.1**. The MCP spec v2025-11-25 is stable; adding it as a wrapper is 1 sprint of work, not a month.

---

## 6. Pending Tasks (v0.2)

> Tasks marked ✅ are resolved in section 5.

### Closed research

- ✅ [x] Evaluate rjsf vs JSONForms — **resolved in 5.2: JSONForms Vue**
- ✅ [x] Evaluate DSPy vs LangChain — **resolved in 5.3: DSPy as core**
- ✅ [x] Compare Ollama self-hosted vs. OpenAI API cost/benefit — **resolved in 5.4**
- ✅ [x] Evaluate Form.io vs JSONForms for MSP use case — **resolved in 5.2: rjsf discarded because it doesn't support Vue. Form.io not considered.**

### Open research

- [ ] Deepen specific expirations (fire extinguisher checks, ladders — exact periodicity). Action: consolidate the expirations table with references to NFPA, Icontec, and sector-specific regulations.
- [ ] Exact CIIU code listing by sector. Action: extract CIIUs associated with each sector (construction, health, manufacturing, industrial) from the DANE activities table.
- [ ] Research templates in other countries (MX: NOM-035, PE: RM). Action: evaluate whether Mexican NOM-035 (psychosocial factors), STPS, and Peruvian RM-Numeral regulations apply to the templates architecture, or are vertical-specific for future expansion.
- [ ] Cron operational monitoring. Action: implement `GET /health/cron` endpoint that returns `last_cron_run_at` and configure Prometheus alert if it exceeds 24h+5min. (Derived from 5.5)

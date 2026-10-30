# Regulation Ledger — Especificación Técnica

> Versión: 0.2.0
> Fecha: 10-oct-2026
> Changelog v0.2: Decisiones arquitectónicas cerradas (sección 5) y estado de la sección 6 actualizado.
> Autor: Yaqui Martinez
> Estado: Especificación técnica pública

---

## 1. Contexto y Problema

### 1.1 El problema del mercado SG-SST actual

Las herramientas de SG-SST en el mercado presentan una carga cognitiva excesiva para el usuario final. Las empresas reciben herramientas con demasiados módulos irrelevantes para su perfil específico, lo que genera:

- **Carga cognitiva excesiva** → frustración → abandono del sistema
- **Módulos irrelevantes** que no aplican según tamaño, sector o clase de riesgo de la empresa
- **Sobrecarga operativa** sin reducción tangible del riesgo real
- **Experiencia confusa** para usuarios sin conocimiento normativo previo

La arquitectura correcta parte de perfil de empresa → templates → menú mínimo viable.

### 1.2 Lo que realmente exige la ley colombiana

No son 113 formularios. Son ~30 formularios únicos organizados por el ciclo PHVA (Decreto 1072/2015 + Resolución 0312/2019):

| Categoría | Forms únicos | Normativa base |
|-----------|-------------|----------------|
| CUMPLIMIENTO | 5 | Res. 0312/2019 |
| DOCUMENTOS | 4 | Decreto 1072/2015 |
| RIESGOS | 3 | GTC 45, Res. 1409/2012 |
| SEGURIDAD VIAL | 2 | Res. 1565/2014, Res. 7495/2020 |
| PERSONAS | 2 | Decreto 1072/2015 |
| EPP | 1 | Decreto 1072/2015 |
| CAPACITACIÓN | 2 | Res. 0312/2019 |
| OPERACIÓN | 3 | Res. 1401/2007, Res. 0312/2019 |
| COMITÉS | 1 | Res. 1918/2009 |
| COMUNICACIÓN | 1 | Decreto 1072/2015 |
| AMBIENTAL | 2 | Decreto 1072/2015 |
| EMERGENCIAS | 2 | Decreto 1072/2015 |
| CONFIGURACIÓN | 2 | Decreto 1072/2015 |

**Total real: ~30 formularios únicos.** La diferencia son vistas, no forms.

### 1.3 Vencimientos y renovaciones (del análisis normativo)

| Documento | Periodicidad | Marco legal |
|-----------|-------------|------------|
| Política SST | 1 año (actualización) | Res. 0312/2019 Art. 4 |
| Plan anual de trabajo | Diciembre (año siguiente) | Res. 0312/2019 Art. 26 |
| Autoevaluación estándares mínimos | Diciembre (anual) | Res. 0312/2019 Art. 26 |
| Matriz legal | Actualización continua | Decreto 1072/2015 |
| Matriz GTC 45 (riesgos) | Actualización ante cambios | GTC 45:2012 |
| Evaluación médica ocupacional | 1 año | Decreto 1072/2015 |
| Inspecciones seguridad | Periodicidad según riesgo | Res. 0312/2019 |
| Arqueos extintores | Trimestral / semestral | NFPA |
| Auditoría interna SG-SST | 1 año | Res. 0312/2019 |
| Capacitaciones | 1 año (cronograma) | Res. 0312/2019 |
| COPASST conformación | Renovación anual | Res. 1918/2009 |
| Reporte accidentes (FURAT) | Inmediato / 2 días | Res. 1401/2007 |
| Plan emergencias | Actualización anual | Decreto 1072/2015 |

---

## 2. Arquitectura del Sistema

### 2.1 Filosofía central

> **El producto no es el formulario. Es el registry + motor de templates + agente IA.**
> Los formularios son consecuencia, no el producto.

Tres capas independientes:

```
┌─────────────────────────────────────────────────┐
│  AGENTE IA — Construcción y asistencia           │
├─────────────────────────────────────────────────┤
│  MOTOR DE TEMPLATES — Menú adaptativo por perfil │
├─────────────────────────────────────────────────┤
│  FORM BUILDER ENGINE — Registry de formularios   │
│  (datos puros, sin lógica de negocio)           │
└─────────────────────────────────────────────────┘
```

### 2.2 Bloque 1 — Form Builder Engine

**Responsabilidad:** Almacenar y servir definiciones de formularios como JSON Schema puro.

**Base tecnológica:**
- **JSONForms Vue** — renderer de forms desde JSON Schema para Vue 3
- **Almacenamiento:** PostgreSQL con JSONB para los schemas

**No reinventamos la rueda.** Usamos una librería establecida y almacenamos los schemas en DB.

#### Registry de formularios

```typescript
interface FormDefinition {
  id: string;                    // UUID
  slug: string;                  // "evaluacion-inicial-sst"
  nombre: string;                // "Evaluación inicial SG-SST"
  descripcion: string;

  categoria: string;             // "cumplimiento"
  subcategoria?: string;          // "evaluacion"

  locked: boolean;               // true = requisito legal (no eliminable)
  editable: boolean;             // true = empresa puede modificar

  applicable_to: {
    clases_riesgo: ('I'|'II'|'III'|'IV'|'V')[];
    sectores: string[];           // CIIU o "todos"
    min_empleados: number;
    max_empleados?: number;
    jurisdiccion: string[];       // ["CO", "MX", "PE"]
  };

  json_schema: object;           // JSON Schema del form
  ui_schema?: object;             // UI Schema (layout)
  normas_refs: string[];         // ["Resolucion 0312 Art. 2.2.4.6.5"]
  vencimientos: {                // Config de alertas
    periodicidad: 'mensual'|'trimestral'|'semestral'|'anual'|'eventual';
    mes_vencimiento?: number;    // 1-12
    dias_alerta?: number;        // días antes de vencer
  };

  version: number;
  created_at: string;
  updated_at: string;
  created_by: string;            // 'system' | user_id | 'ai_agent'
}
```

#### Categories registry

```typescript
interface Category {
  id: string;
  slug: string;                  // "cumplimiento"
  nombre: string;                // "Cumplimiento Legal"
  descripcion: string;
  icono: string;                 // emoji o nombre de icono
  orden: number;                 // orden en el menú
  color?: string;                // color del tema
  templates_default: string[];   // qué templates la incluyen por defecto
}
```

**Categorías estándar (CO):**
- cumplimiento
- documentos
- riesgos
- seguridad-vial
- personas
- epp
- capacitacion
- operacion
- comités
- comunicacion
- ambiental
- emergencias
- configuracion

### 2.3 Bloque 2 — Wizard + Template Engine

**Responsabilidad:** Generar el menú de formularios aplicables según el perfil de la empresa.

#### Perfil de empresa

```typescript
interface CompanyProfile {
  id: string;
  nombre: string;
  nit: string;

  // Datos que determinan los forms aplicables
  clase_riesgo: 'I'|'II'|'III'|'IV'|'V';
  sector: string;                // código CIIU
  num_empleados: number;
  ciudad: string;
  jurisdiccion: string;          // "CO" default

  // Config del wizard
  wizard_completado: boolean;
  wizard_template?: string;      // template_id usado
  wizard_data?: object;          // datos collected

  // Config del menú
  enabled_forms: string[];      // form_definition_ids activos
  disabled_forms: string[];     // form_definition_ids deshabilitados
  custom_forms: FormDefinition[]; // forms creados por la empresa

  created_at: string;
  updated_at: string;
}
```

#### Templates

```typescript
interface Template {
  id: string;
  slug: string;                  // "pymes-riesgo-i-ii"
  nombre: string;                // "PYME Riesgo I-II"
  descripcion: string;

  // Perfil que activa este template
  applies_to: {
    clases_riesgo: ('I'|'II'|'III'|'IV'|'V')[];
    sectores?: string[];
    num_empleados_range?: [number, number];
  };

  // Forms incluidos en este template
  include_forms: string[];       // form_definition_ids
  exclude_forms: string[];       // form_definition_ids (excepciones)
  orden_sugerido: string[];      // form_definition_ids en orden de diligenciamiento

  // Wizard steps para este template
  wizard_steps: WizardStep[];

  created_by: 'system' | user_id;
  is_public: boolean;            // visible para otros usuarios
}

interface WizardStep {
  id: string;
  orden: number;
  titulo: string;
  descripcion: string;
  fields: object[];             // JSON Schema subset para este step
  completado: boolean;
}
```

**Templates por defecto (CO):**

| Template | Perfil | Forms |
|---------|--------|-------|
| microempresa | riesgo I-II, 1-5 empleados | ~12 forms |
| pyme | riesgo I-III, 6-50 empleados | ~20 forms |
| empresa-grande | riesgo III-V, 51+ empleados | ~30 forms |
| construccion | cualquier riesgo, sector construcción | +Seguridad Vial PESV completo |
| salud | cualquier riesgo, sector salud | +Bioseguridad |
| manufactura | cualquier riesgo, sector manufactura | +Químicos |
| industrial | riesgo IV-V, sector industrial | +Todo |

#### Motor de templates

```typescript
function getApplicableTemplate(profile: CompanyProfile): Template {
  // 1. Buscar template exacto por perfil
  // 2. Si no hay exacto, buscar template por clase_riesgo
  // 3. Si no hay, usar "pyme" como fallback
  // Retorna template con forms resueltos
}

function getFormsForProfile(profile: CompanyProfile): FormDefinition[] {
  const template = getApplicableTemplate(profile);
  const locked = getLockedForms();  // siempre presentes
  const applicable = template.include_forms
    .filter(id => !template.exclude_forms.includes(id))
    .concat(locked);
  return applicable.map(id => getFormDefinition(id));
}
```

### 2.4 Bloque 3 — Sistema de Alertas de Vencimientos

**Responsabilidad:** Notificar antes de que un documento venza.

```typescript
interface Vencimiento {
  id: string;
  company_id: string;
  form_definition_id: string;
  fecha_vencimiento: string;      // ISO date
  fecha_alerta: string;          // fecha cuando se envía la alerta
  estado: 'pendiente'|'alerta_enviada'|'vencido'|'renovado';
  ultimo_registro_id?: string;   // último submission ID
  alert_sent_at?: string;
  renewed_at?: string;
}

// Job diário que corre a medianoche
async function verificarVencimientos() {
  const manana = addDays(today, 1);
  const vencimientos = await db.vendimientos.findMany({
    where: {
      estado: 'pendiente',
      fecha_alerta: { lte: manana }
    }
  });

  for (const v of vencimientos) {
    await enviarAlerta(v);
    await db.vendimientos.update(v.id, {
      estado: 'alerta_enviada',
      alert_sent_at: now()
    });
  }
}
```

**Tipos de alerta:**
- **Email** — a los responsables del form
- **Dashboard** — badge en el menú
- **Notificación push** (futuro)
- **Resumen semanal** (futuro)

### 2.5 Bloque 4 (extensión opcional) — Asistente de autoría

**Responsabilidad:** Construir formularios nuevos, asistir al usuario, validar cruces normativos.

> **Alcance.** Este bloque es una extensión opcional, apagada por defecto y no
> necesaria para operar ninguna capa core. Las capas 1-3 y 5 (registry, template engine,
> snapshots/vencimientos, evidencia y conector MSP) son deterministas y funcionan sin él.
> Se desarrolla de forma independiente del camino principal.

#### 4a. Generación de formularios por normativa

```typescript
interface AIAgentFormGeneration {
  // Input: texto de una norma o PDF
  // Output: FormDefinition listo para存入 registry

  async function generateFormFromNorma(
    normaTexto: string,
    metadata: { categoria: string; jurisdiccion: string }
  ): Promise<FormDefinition>;

  // Pasos:
  // 1. Parsear el texto → extraer campos, tipos, validaciones
  // 2. Identificar referencias legales → norms_refs
  // 3. Clasificar → categoría, applicable_to
  // 4. Generar JSON Schema
  // 5. Guardar en registry como draft
  // 6. Notificar a admin para revisión
}
```

**Herramientas del agente:**
- `web_search` — buscar textos de normas actualizadas
- `document_analysis` — extraer estructura de PDFs normativos
- `json_schema_generator` — generar schema desde texto estructurado

#### 4b. Asistencia en diligenciamiento

```typescript
interface AIAssistanceRequest {
  form_id: string;
  current_field: string;
  user_context: {
    empresa: CompanyProfile;
    perfil_completado: boolean;
  };
}

async function assistFieldFilling(req: AIAssistanceRequest): Promise<{
  explanation: string;           // qué pide el campo en lenguaje simple
  evidence_hint: string;         // qué evidencia typically se sube
  related_fields: string[];       // campos relacionados en otros forms
  normativa_excerpt: string;      // texto relevante de la norma
}>;
```

#### 4c. Validación cruzada normativa

```typescript
// Cuando cambia una norma, el agente reevalúa
async function revalidateFormsOnNormaChange(
  normaId: string,
  nuevaVersionTexto: string
): Promise<{
  affected_forms: string[];
  proposed_changes: FormDefinition[];
  breaking_changes: string[];    // campos eliminados o modificados
}>;
```

#### 4d. Motor IA configurable

El agente usa un LLM configurable — no se ata a un provider específico.

```typescript
interface LLMConfig {
  provider: 'openai' | 'anthropic' | 'ollama' | 'custom';
  model: string;
  api_key?: string;              // para cloud
  base_url?: string;             // para self-hosted (Ollama)
}
```

**Decisión:** Usar **DSPy** (declarative LM programs) como abstracción. Permite cambiar de modelo sin cambiar código.

### 2.6 Bloque 5 — Arquitectura MSP / Multi-tenant

**Responsabilidad:** Permitir que un MSP gestione múltiples empresas clientes desde una sola instancia, con sus propios datos aislados y branding opcional.

#### Multi-tenancy

```typescript
interface Tenant {
  id: string;                   // tenant_id
  name: string;
  slug: string;                  // subdomain

  // Aislamiento de datos
  schema_name: string;           // PostgreSQL schema por tenant
                                 // O: campo tenant_id en cada tabla

  // Branding (white-label)
  branding?: {
    logo_url?: string;
    primary_color?: string;
    favicon_url?: string;
    custom_domain?: string;       // CNAME
  };

  // Config del MSP
  is_msp: boolean;               // true = es un MSP
  msp_id?: string;               // si es cliente de un MSP, cuál MSP

  // Límites del plan
  plan: 'starter'|'professional'|'enterprise';
  max_companies?: number;        // para MSP
  max_users?: number;
  features_enabled: string[];

  created_at: string;
  updated_at: string;
}
```

#### Modelo de datos multi-tenant

**Opción elegida: Shared schema con tenant_id**

```
// Ventaja: más simple de operar
// Desventaja: requiere siempre filtrar por tenant_id
// La ventaja supera la desventaja para el caso de uso

Tablas con tenant_id:
  - companies
  - company_forms (asignaciones de forms por empresa)
  - form_submissions (datos de los forms)
  - vencimientos
  - alerts

Tablas sin tenant_id (son del platform owner):
  - form_definitions (registry global)
  - categories
  - templates
  - tenant (config del MSP)
```

#### Conector MSP — API para agentes externos

```typescript
interface MSPConnector {
  // Un MSP (o agente externo) puede crear/gestionar empresas
  // vía API con autenticación propia

  createCompany(data: {
    nombre: string;
    nit: string;
    clase_riesgo: string;
    sector: string;
    num_empleados: number;
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
    vencimientos_proximos: number;
    compliance_score: number;
  }>;

  generateReport(companyId: string, format: 'pdf'|'xlsx'): Promise<Buffer>;

  // También para el agente IA externo
  submitForm(companyId: string, formId: string, data: object): Promise<string>;
  getFormStatus(companyId: string, formId: string): Promise<string>;
}
```

#### Agente externo vía MCP

```
regulation-ledger expone un servidor MCP (Model Context Protocol)
que permite a un agente externo:
  1. Listar empresas gestionadas
  2. Ver estado de cumplimiento
  3. Crear/modificar forms
  4. Generar reportes
  5. Recibir alertas de vencimientos

Esto es lo que el usuario llamó "LaserBeam" — un agente que se conecta
a regulation-ledger para construir formularios automáticamente.
```

---

## 3. Stack Tecnológico

| Capa | Tecnología | Justificación |
|------|-----------|---------------|
| Frontend | Vue 3 + TypeScript | Menor superficie de ataque, tipado estricto, ecosistema deformularios maduro |
| Estado | Pinia | State management Vue 3 |
| Backend | Node.js + Express o Fastify | Runtime único para API y jobs, despliegue self-hosted simple |
| Base de datos | PostgreSQL + JSONB | Datos estructurados + schemas flexibles |
| ORM | Prisma | Type-safe, migrations, multi-tenant |
| Auth | Lucia | Multi-tenant con roles, self-hosted |
| Form rendering | JSONForms Vue | JSON Schema estándar para Vue |
| AI | DSPy + LLM configurable | Declarativo, swappable |
| API | REST + MCP (futuro) | Conector MSP/AI |
| Hosting | Self-hosted (Docker) | El usuario quiere efectivo, no SaaS |
| CI/CD | GitHub Actions | Repo en GitHub |

---

## 4. Fases de Desarrollo

### Fase 1 — V1.0 (Base + Wizard mínimo)
- [ ] PostgreSQL + Prisma setup con schema multi-tenant
- [ ] Registry de formularios con 10 forms locked (críticos CO)
- [ ] Categories registry
- [ ] Wizard de onboarding (3 steps: empresa, riesgo, empleados)
- [ ] Template engine (3 templates: micro, pyme, grande)
- [ ] Renderer de forms desde JSON Schema
- [ ] Submission storage
- [ ] Auth con roles (admin, editor, viewer)

### Fase 2 — V1.1 (Templates + Vencomientos)
- [ ] Resto de forms locked (~30 total)
- [ ] Templates por sector (construcción, salud, manufactura)
- [ ] Sistema de vencimientos con alertas email
- [ ] Dashboard de cumplimiento (% completado)
- [ ] Menú dinámico desde registry

### Fase 3 — V2.0 (Agente IA)
- [ ] DSPy setup con LLM configurable
- [ ] Agente de generación de forms desde normativa
- [ ] Asistencia en diligenciamiento (chat inline)
- [ ] Validación cruzada normativa

### Fase 4 — V2.1 (MSP + Multi-tenant)
- [ ] Conector MSP REST API
- [ ] Servidor MCP para agente externo
- [ ] White-labeling (logo, colores, dominio)
- [ ] Bulk operations para MSPs

### Fase 5 — V2.2 (Multi-región)
- [ ] Jurisdicción MX, PE
- [ ] Templates internacionales
- [ ] Traducciones de UI

---

## 5. Decisiones Arquitectónicas Cerradas (v0.2)

> Decisiones tomadas tras investigación externa. Citas en línea cuando aplica.

### 5.1 Multi-tenancy: schema compartido con `tenant_id` + Row-Level Security (RLS)

**Decisión:** Shared schema con columna `tenant_id` en cada tabla, **forzado por PostgreSQL RLS** (no solo convención de aplicación).

**Evidencia:**
- 2025-2026 consensus (Streamkap, AWS, dev.to, Supabase): "For the vast majority of modern SaaS applications, the shared-schema model using a `tenant_id` column is the way to go. It provides the clearest path to scalability and cost savings." [streamkap.com](https://streamkap.com/resources-and-guides/postgresql-cdc-multi-tenant)
- RLS en PostgreSQL está production-grade: AWS demuestra cross-tenant writes devuelven `UPDATE 0` cuando la policy bloquea, no requiere código adicional si no confiar en la convención. [aws.amazon.com](https://aws.amazon.com/blogs/database/multi-tenant-data-isolation-with-postgresql-row-level-security/)
- Trade-off honesto: schema-per-tenant es 100K instancias imposibles de operar; shared schema escala, RLS protege.

**Implementación canónica:**
```sql
ALTER TABLE companies ENABLE ROW LEVEL SECURITY;
ALTER TABLE companies FORCE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON companies
  USING (tenant_id = current_setting('app.current_tenant')::uuid);
```
Cada request setea `SET LOCAL app.current_tenant = '<uuid>'` antes de ejecutar. Sin exception manual, no hay fugas.

**Tablas con `tenant_id`:** companies, company_forms, form_submissions, vencimientos, alerts.
**Tablas globales (sin tenant_id):** form_definitions, categories, templates, form_versions (registry global, lectura por todos los tenants).

### 5.2 Form renderer: JSONForms Vue

**Decisión:** JSONForms Vue (`@jsonforms/vue`), no react-jsonschema-form (rjsf).

**Cambio respecto a la recomendación del SPEC v0.1:** el SPEC v0.1 decía "rjsf, más popular". Era incorrecto: rjsf **es solo React**. La recomendación queda invalida porque el stack ya está definido como Vue 3.

**Evidencia:**
- rjsf issue [#4482](https://github.com/rjsf-team/react-jsonschema-form/issues/4482) (mantenedores oficial): "You may just want to build a new system called Vue Json Schema Form" — no hay versión Vue oficial de rjsf.
- JSONForms Vue 3.8.0 (jun-2026): primera clase, MIT, bindings oficiales para Vue 3, mismo core que React/Angular. [eclipsesource/jsonforms](https://github.com/eclipsesource/jsonforms)
- JSONForms Community: "focuses on customizability and declarative forms. JSON Schema + UI Schema." [jsonforms discourse](https://jsonforms.discourse.group/t/compare-to-react-jsonschema-form/553)
- Comparativa 2026: rjsf 900K downloads/semana (más popular), JSONForms 238K (más customizable). Ambos MIT. [faun.dev](https://faun.dev/toolbox/json-forms-vs-react-jsonschema-form/)

**Trade-off:** JSONForms requiere a veces escribir renderers custom cuando un campo no encaja en los sets default (ej: campos SST tipo matriz legal). Vale la pena por tener Vue 3 nativo.

**Implementación:** `@jsonforms/vue` + `@jsonforms/vue-vanilla` para arrancar, `@jsonforms/vue-vuetify` si queremos Vuetify como design system.

### 5.3 Extensión opcional de asistencia: DSPy como núcleo, LangChain opcional para tooling

**Decisión:** DSPy para el asistente interno de la extensión opcional (generación de forms, asistencia, validación cruzada). LangChain/LangGraph descartado como core (overhead, integración no es nuestro cuello de botella).

**Evidencia:**
- DSPy es un compiler para prompts: declaras `Signature` + `Module`, el optimizer (`MIPROv2`) busca instrucciones automáticamente y se adapta al cambiar de modelo. [randomcommits.com](https://randomcommits.com/dspy-vs-langchain)
- Caso real (randomcommits): cambiaron gpt-3.5 → gpt-4-turbo y todos los 50 LangChain chains fallaron porque los prompts eran strings hardcoded. DSPy re-compila automáticamente.
- Caso benchmark: 0.83 → 0.90 exact-match al migrar a DSPy en clasificación multi-categoría. Costo: $12 de compile, 30 min.
- "DSPy vs LangChain no son el mismo layer. DSPy optimiza el prompt dado un dataset. LangChain conecta providers + tools + retrievers." [tinyagents.dev](https://tinyagents.dev/vs/dspy-vs-langchain)
- Caso de uso nuestro = agente que genera JSON Schema desde texto normativo. Tenemos dataset (textos de Decretos/Resoluciones), tenemos métrica (schema válido + campos cubiertos). **DSPy ajusta perfecto.**
- LangChain solo justificaría si necesitáramos conectores a 600+ herramientas externas. No es nuestro caso.

**Regla decisión:** si en el futuro queremos que un agente externo (LaserBeam/MCP) construya forms en regulation-ledger, exponemos los tool-calls de LangChain o una API REST simple. Pero el agente interno es DSPy.

### 5.4 Inferencia de la extensión opcional: configurable cloud + self-hosted, default self-hosted para datos sensibles

**Decisión:** arquitectura neutral con **Ollama self-hosted como default**, OpenAI/Anthropic opcional.

**Evidencia de costos (septiembre 2026):**
| Volumen | OpenAI GPT-5.6 Luna | Ollama rentado (RTX 4090) | Ollama propio |
|---|---|---|---|
| 5M tokens/mes | ~$2.50 | ~$6 | ~$97 (capex) |
| 100M tokens/mes | ~$50 | ~$121 | ~$115 |
| 1B tokens/mes | ~$800 | ~$1,207 | — |

[markaicode.com / sept 2026](https://markaicode.com/pricing/ollama-vs-openai-api-pricing)

**Lectura:** Self-host rentado **nunca gana** vs OpenAI más barato. Self-host propio **gana recién arriba de ~100M tokens/mes**. Para regulation-ledger (auditoría SST, normativas colombianas), el cuello es compliance, no throughput. Usar cloud barato (GPT-5.6 Luna, $0.50/1M blended) hasta ~100M tokens/mes, después self-host si la adopción lo justifica.

**Privacidad:** Ollama local = datos nunca salen del host. Compatible con normativas colombianas (Ley 1581 Habeas Data). OpenAI/Anthropic cloud = requiere DPA + evaluación de jurisdicción.

**Recomendación operativa:**
1. Default MVP: Ollama local con modelo open (Llama 3.1 8B o similar) para cero dependencia cloud.
2. Cliente enterprise que pide modelos grandes: OpenAI/Anthropic vía API, configurable en `config.yaml`.
3. Documentado como "Self-hosted default, configurable a cloud" en el `.env.example`.

### 5.5 Alertas de vencimientos: cron polling diario + tabla `vencimientos`

**Decisión:** mantiene el job polling del SPEC v0.1 (línea 286-302). No webhook externo.

**Evidencia:**
- Cron polling para tareas recurrentes a cadencia fija es el caso ideal. "Cron is fine for recurring system tasks like cache warming, report generation, and log rotation. It runs on a fixed cadence and does its job well." [posthook.io](https://posthook.io/compare/cron-vs-durable-scheduling)
- Si tu trabajo es "ejecutar este SQL cada hora a medianoche", cron es la respuesta. Webhooks tienen sentido cuando el productor del evento dispara la notificación (no aplica para vencimientos que dependen del tiempo).
- Limitación a tener en cuenta: "every concern gets its own polling query and its own cron job. At five or ten concerns, you have a growing set of pollers." Si regulation-ledger llega a muchos tipos de alerta, reconsiderar. Por ahora: 1 poller (vencimientos) es trivial.

**Postura de monitoreo añadida:** cron no trae alerting por default. Si el job falla silencioso, nadie se entera. **Agregar health-check endpoint en el ledger-api que devuelva `last_cron_run_at`** y metricas Prometheus para alertar si el job no corre en 24h+5min.

### 5.6 Agente externo: REST API primero, MCP server en V2.1

**Decisión:** mantiene del SPEC v0.1: REST API como conector MSP/AI en V2.1, MCP server en fase posterior.

**Evidencia (cambia el landscape respecto a oct-2025):**
- MCP spec v2025-11-25 ya está estable, OAuth 2.0 Resource Server compliance, soporte enterprise. [modelcontextprotocol.info](https://modelcontextprotocol.info/blog/first-mcp-anniversary)
- IBM App Connect 13.0.8 production-ready: "expose your REST APIs as MCP tools without code changes". [ibm.com](https://www.ibm.com/docs/en/app-connect/13.0.x?topic=solutions-developing-mcp-tools)
- REST sigue siendo la elección práctica inicial: "REST first, MCP después" sigue vigente porque REST tiene el 100% de los casos MSP cubiertos, y "exponer REST como MCP tools" es trivial cuando se quiera.

**Postura corregida:** en lugar de "REST primero, MCP después", la decisión cerrada es **REST para V2.0 (conector MSP), MCP como adaptador delgado sobre REST en V2.1**. La especificación MCP v2025-11-25 es estable; agregarlo como wrapper es 1 sprint de trabajo, no un mes.

---

## 6. Tareas Pendientes (v0.2)

> Las tareas marcadas ✅ están resueltas en la sección 5.

### Investigación cerrada

- ✅ [x] Evaluar rjsf vs JSONForms — **resuelto en 5.2: JSONForms Vue**
- ✅ [x] Evaluar DSPy vs LangChain — **resuelto en 5.3: DSPy como core**
- ✅ [x] Comparar costo/beneficio Ollama self-hosted vs. OpenAI API — **resuelto en 5.4**
- ✅ [x] Evaluar Form.io vs react-jsonschema-form para el caso MSP — **resuelto en 5.2: rjsf descartado porque no soporta Vue. Form.io no se considera.**

### Investigación abierta

- [ ] Profundizar en vencimientos específicos (arqueos, extintores, escaleras — periodicidad exacta). Acción: consolidar tabla de vencimientos con referencias a NFPA, Icontec, y normativa sectorial.
- [ ] Listado exacto de CIIU por sector. Acción: extraer los CIIUs asociados a cada sector (construcción, salud, manufactura, industria) desde la tabla de actividades del DANE.
- [ ] Investigación de templates en otros países (MX: NOM-035, PE: RM). Acción: evaluar si las normas mexicanas NOM-035 (factores psicosociales), STPS, y peruanas RM-Numeral aplican a la arquitectura de templates, o si son vertical-specific para futura expansión.
- [ ] Cron monitoring operacional. Acción: implementar endpoint `GET /health/cron` que devuelva `last_cron_run_at` y configurar alert Prometheus si supera 24h+5min. (Derivado de 5.5)

# Regulation Ledger, Especificación Técnica

> Versión: 0.2.0
> Fecha: 10-oct-2026
> Changelog v0.2: Decisiones arquitectónicas cerradas (sección 5) y estado de la sección 6 actualizado.
> Autor: Yaqui Martinez
> Estado: Especificación técnica pública

---

## 1. Contexto y Problema

### 1.1 El problema del mercado SG-SST actual

Las herramientas de SG-SST en el mercado presentan una carga cognitiva excesiva para el usuario final. Las empresas reciben herramientas con demasiados módulos irrelevantes para su perfil específico, lo que genera:

- **Carga cognitiva excesiva** -> frustración -> abandono del sistema
- **Módulos irrelevantes** que no aplican según tamaño, sector o clase de riesgo de la empresa
- **Sobrecarga operativa** sin reducción tangible del riesgo real
- **Experiencia confusa** para usuarios sin conocimiento normativo previo

La arquitectura correcta parte de perfil de empresa -> templates -> menú mínimo viable.

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

> **El producto no es el formulario. Es el registry y el motor de templates.**
> Los formularios son consecuencia, no el producto.

Tres capas independientes:
```
┌─────────────────────────────────────────────────┐
│  EVIDENCIA Y SNAPSHOTS: registro verificable     │
├─────────────────────────────────────────────────┤
│  MOTOR DE TEMPLATES: menú adaptativo por perfil  │
├─────────────────────────────────────────────────┤
│  FORM BUILDER ENGINE: registry de formularios    │
│  (datos puros, sin lógica de negocio)           │
└─────────────────────────────────────────────────┘
```

### 2.2 Bloque 1, Form Builder Engine

**Responsabilidad:** Almacenar y servir definiciones de formularios como JSON Schema puro.

**Base tecnológica:**
- **JSONForms Vue**, renderer de forms desde JSON Schema para Vue 3
- **Almacenamiento:** PostgreSQL con JSONB para los schemas

**No reinventamos la rueda.** Usamos una librería establecida y almacenamos los schemas en DB.

#### Registry de formularios
```

typescript
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
```

typescript
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

### 2.3 Bloque 2, Wizard + Template Engine

**Responsabilidad:** Generar el menú de formularios aplicables según el perfil de la empresa.

#### Perfil de empresa
```

typescript
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
```

typescript
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
```

typescript
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

### 2.4 Bloque 3, Sistema de Alertas de Vencimientos

**Responsabilidad:** Notificar antes de que un documento venza.
```

typescript
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
- **Email**, a los responsables del form
- **Dashboard**, badge en el menú
- **Notificación push** (futuro)
- **Resumen semanal** (futuro)

### 2.5 Bloque 4 (extensión opcional), Asistente de autoría

**Responsabilidad:** Construir formularios nuevos, asistir al usuario, validar cruces normativos.

> **Alcance.** Este bloque es una extensión opcional, apagada por defecto y no
> necesaria para operar ninguna capa core. Las capas 1-3 y 5 (registry, template engine
> snapshots/vencimientos, evidencia y conector MSP) son deterministas y funcionan sin él.
> Se desarrolla de forma independiente del camino principal.

#### 4a. Generación de formularios por normativa
```

typescript
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
- `web_search`, buscar textos de normas actualizadas
- `document_analysis`, extraer estructura de PDFs normativos
- `json_schema_generator`, generar schema desde texto estructurado

#### 4b. Asistencia en diligenciamiento
```

typescript
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
```

typescript
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

El agente usa un LLM configurable, no se ata a un provider específico.
```

typescript
interface LLMConfig {
  provider: 'openai' | 'anthropic' | 'ollama' | 'custom';
  model: string;
  api_key?: string;              // para cloud
  base_url?: string;             // para self-hosted (Ollama)
}
```

**Decisión:** Usar **DSPy** (declarative LM programs) como abstracción. Permite cambiar de modelo sin cambiar código.

### 2.6 Bloque 5, Arquitectura MSP / Multi-tenant

**Responsabilidad:** Permitir que un MSP gestione múltiples empresas clientes desde una sola instancia, con sus propios datos aislados y branding opcional.

#### Multi-tenancy
```

typescript
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

#### Conector MSP, API para agentes externos
```

typescript
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

Un agente externo se conecta a regulation-ledger por ese servidor
para construir y gestionar formularios automáticamente.
```

---

## 3. Stack Tecnológico

| Capa | Tecnología | Justificación |
|------|-----------|---------------|
| Frontend | Vue 3 + TypeScript | Menor superficie de ataque, tipado estricto, ecosistema de formularios maduro |
| Estado | Pinia | State management Vue 3 |
| Backend | Node.js + Express o Fastify | Runtime único para API y jobs, despliegue self-hosted simple |
| Base de datos | PostgreSQL + JSONB | Datos estructurados + schemas flexibles |
| ORM | Prisma | Type-safe, migrations, multi-tenant |
| Auth | Lucia | Multi-tenant con roles, self-hosted |
| Form rendering | JSONForms Vue | JSON Schema estándar para Vue |
| API | REST + MCP (futuro) | Conector MSP |
| Hosting | Self-hosted (Docker) | Despliegue en infraestructura propia del operador |
| CI/CD | GitHub Actions | Repo en GitHub |

---

## 4. Fases de Desarrollo

### Fase 1, V1.0 (Base + Wizard mínimo)
- [ ] PostgreSQL + Prisma setup con schema multi-tenant
- [ ] Registry de formularios con 10 forms locked (críticos CO)
- [ ] Categories registry
- [ ] Wizard de onboarding (3 steps: empresa, riesgo, empleados)
- [ ] Template engine (3 templates: micro, pyme, grande)
- [ ] Renderer de forms desde JSON Schema
- [ ] Submission storage
- [ ] Auth con roles (admin, editor, viewer)

### Fase 2, V1.1 (Templates + Vencimientos)
- [ ] Resto de forms locked (~30 total)
- [ ] Templates por sector (construcción, salud, manufactura)
- [ ] Sistema de vencimientos con alertas email
- [ ] Dashboard de cumplimiento (% completado)
- [ ] Menú dinámico desde registry

### Fase 3, V2.0 (Conector MSP + multi-tenant)
- [ ] Conector MSP REST API
- [ ] Servidor MCP como adaptador sobre la API
- [ ] White-labeling (logo, colores, dominio)
- [ ] Operaciones en lote para MSPs

### Fase 4, V2.1 (Multi-región)
- [ ] Conector MSP REST API
- [ ] Servidor MCP para agente externo
- [ ] White-labeling (logo, colores, dominio)
- [ ] Bulk operations para MSPs

### Fase 5, V2.2 (Cobertura internacional)
- [ ] Jurisdicción MX, PE
- [ ] Templates internacionales
- [ ] Traducciones de UI

> La extensión opcional de asistencia (sección 5.3) no forma parte de estas fases ni
> del alcance financiado.

---

## 5. Decisiones de arquitectura

Decisiones cerradas que condicionan la implementación. Cada una documenta la decisión
su justificación técnica y su efecto en el código.

### 5.1 Multi-tenancy: esquema compartido con `tenant_id` y Row-Level Security

**Decisión:** esquema compartido con columna `tenant_id` en cada tabla, **forzado por
PostgreSQL RLS**, no por convención de la aplicación.

**Justificación:** un esquema por tenant no es operable a escala. El esquema compartido
mantiene una sola base y delega el aislamiento en el motor, que lo aplica aunque el
código de aplicación tenga un fallo.

**Implementación canónica:**
```

sql
ALTER TABLE companies ENABLE ROW LEVEL SECURITY;
ALTER TABLE companies FORCE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON companies
  USING (tenant_id = current_setting('app.current_tenant')::uuid);
```

Cada request ejecuta `SET LOCAL app.current_tenant = '<uuid>'` antes de la consulta.

**Tablas con `tenant_id`:** companies, company_forms, form_submissions, vencimientos, alerts.
**Tablas globales:** form_definitions, categories, templates, form_versions, registro
compartido, de lectura para todos los tenants.

### 5.2 Renderizado de formularios: JSONForms

**Decisión:** `@jsonforms/vue` con JSON Schema + UI Schema.

**Justificación:** es la opción con bindings oficiales para Vue 3, el framework del
proyecto, y mantiene el mismo núcleo que sus variantes para React y Angular.

**Costo asumido:** algunos campos del dominio, como las matrices legales de SST
requieren renderers propios.

**Implementación:** `@jsonforms/vue` + `@jsonforms/vue-vanilla` al inicio;
`@jsonforms/vue-vuetify` si se adopta Vuetify como sistema de diseño.

### 5.3 Extensión opcional de asistencia

**Decisión:** el asistente de autoría queda fuera del núcleo. Cuando se implemente, usa DSPy.

**Justificación:** el caso de uso es generar JSON Schema a partir de texto normativo
con un conjunto de datos disponible (decretos y resoluciones) y una métrica verificable
(schema válido y campos cubiertos). DSPy optimiza el prompt contra esa métrica y se
recompila cuando cambia el modelo, que es el fallo característico de los prompts
escritos a mano.

**Alcance:** ninguna capa del núcleo depende de esta extensión.

### 5.4 Inferencia de la extensión opcional

**Decisión:** arquitectura neutral, con inferencia local por defecto y proveedores en la
nube como alternativa configurable.

**Justificación:** los datos de cumplimiento SST son sensibles bajo la Ley 1581 de Habeas
Data. La inferencia local mantiene los datos en el host del cliente; un proveedor en la
nube exige acuerdo de tratamiento y evaluación de jurisdicción.

**Implementación:** proveedor y modelo configurables en `config.yaml`, documentados en
`.env.example`. El asistente se distribuye apagado.

### 5.5 Alertas de vencimientos

**Decisión:** job de polling diario sobre la tabla `vencimientos`.

**Justificación:** los vencimientos dependen del tiempo, no de un evento externo. Un
planificador con cadencia fija cubre el caso sin infraestructura de eventos. Si el
catálogo de alertas crece, la decisión se revisa.

**Monitoreo:** el ledger-api expone `last_cron_run_at` en su endpoint de salud, con una
métrica para alertar si el job no corre en 24 h.

### 5.6 Integración con agentes externos

**Decisión:** API REST para el conector MSP en V2.0; servidor MCP como adaptador sobre esa
misma API en V2.1.

**Justificación:** REST cubre los casos de integración del MSP y es la superficie sobre la
que se construye cualquier otro protocolo. El adaptador MCP no duplica lógica.

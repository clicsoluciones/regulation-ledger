# Roadmap — Regulation Ledger

## Estrategia central

Desarrollo basado en **Form Builder Engine** (registry de JSON Schema versionado) y **Templates por compañía** (menú adaptativo desde perfil de empresa).

---

## Fase 1: Foundation — Mes 1-2

**Meta:** Form Builder Engine funcional con registry de JSON Schema.

### Entregables
- Prisma schema: `Form`, `FormVersion`, `FormCategory`, `Company`, `CompanyProfile`
- API REST: CRUD de formularios y versiones
- JSONForms Vue renderer conectado al registry
- Sistema de categorías y tags por formulario
- Formularios bloqueados (no eliminables, solo versionables)

### Criterio de éxito
Un formulario dinámico renderizado desde JSON Schema en Vue 3 sin código nuevo por formulario.

---

## Fase 2: Company Profiles + Wizard — Mes 2-3

**Meta:** Wizard de configuración que genera el menú adaptativo.

### Entregables
- Wizard de 7 pasos (empresa, sedes, áreas, responsables, clasificación)
- CompanyProfile: `clase_riesgo`, `sector_ciiu`, `num_empleados`, `jurisdiccion`
- Template Engine: perfil → template aplicable → menú de formularios
- CRUD de companies y profiles
- Permisos y roles por empresa

### Criterio de éxito
Empresa creada → menú muestra solo los formularios que le aplican según su perfil.

---

## Fase 3: Sistema de Vencimientos — Mes 3-4

**Meta:** Alertas automáticas de expiración documental.

### Entregables
- Tabla de frecuencias normativas (anual, mensual, trimestral)
- Cron job que calcula próximas expiraciones
- Notificaciones por email
- Dashboard de vencimientos pendientes
- Historial de renovaciones

### Criterio de éxito
El sistema alerta antes de que venza un documento o certificación.

---

## Fase 4: Evidencia y snapshots normativos — Mes 4-6

**Meta:** Traza de evidencia verificable y control de vigencia normativa.

### Entregables
- Cadena de hashes append-only sobre presentaciones y documentos de soporte
- Export a JSON plano: el historial completo se puede leer sin la aplicación
- Snapshot inmutable por versión de norma
- Detección de formularios afectados cuando una norma cambia
- Revalidación cruzada de reglas cuando cambia la norma

### Criterio de éxito
Un auditor puede reconstruir qué se presentó, cuándo y contra qué versión de la norma,
usando solo el export.

---

## Fase 5: Conector MSP — Mes 7-12

**Meta:** API para MSPs que gestionan múltiples empresas cliente.

### Entregables
- API REST con autenticación por API key
- Servidor MCP para integraciones externas
- Dashboard MSP: todas las empresas cliente, métricas agregadas
- Operaciones en masa (crear empresa, asignar template, importar datos)
- Webhooks para notificaciones externas

### Criterio de éxito
Un MSP gestiona 50 empresas desde una sola API.

---

## Hitos del roadmap

| Fecha | Hito |
|-------|------|
| mes 2 | Form Builder funcional + wizard |
| mes 4 | Vencimientos + snapshots normativos |
| mes 6 | Cadena de evidencia + export |
| mes 12 | Conector MSP + demo pública |

## Extensión opcional: asistencia en la autoría de formularios

Fuera del camino principal y sin dependencia de las capas core, existe una extensión
opcional con un asistente local para la autoría de formularios: se ejecuta apagada por
defecto, requiere un modelo alojado localmente y no es necesaria para operar ninguna capa
core. Se desarrolla como contribución de la comunidad, no como parte de las fases
anteriores.

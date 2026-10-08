# regulation-ledger

Sistema open source de gestión de cumplimiento normativo con formularios adaptativos, templates regulatorios y seguimiento de evidencia asistido por IA para cumplimiento regulatorio multi-jurisdiccional.

[English](README.md)

---

## Acerca del proyecto

`regulation-ledger` resuelve un problema concreto: las empresas enfrentan formularios regulatorios abrumadores que no coinciden con su perfil real — tamaño incorrecto, clase de riesgo, o sector — generando confusión, incumplimiento y desperdicio de recursos.

### Por qué importa este proyecto

1. **Los templates adaptativos reemplazan el enfoque único.** Una microempresa de 5 empleados y una constructora de 500 tienen necesidades de cumplimiento radicalmente diferentes. `regulation-ledger` genera un conjunto mínimo y relevante de formularios basado en el perfil de la empresa.

2. **El registry es el producto.** Los formularios viven en un registry versionado de JSON Schema, extensible por el owner de la plataforma, por MSPs que gestionan múltiples clientes, y por agentes IA que generan nuevos formularios desde texto normativo bruto.

3. **Los datos de compliance se quedan on-premise.** Diseñado para auto-hosting. Para sectores con requisitos estrictos de residencia de datos u organizaciones que requieren control total de su evidencia.

## Arquitectura

```
┌──────────────────────────────────────────────────────────────┐
│  Frontend  (Vue 3 + TypeScript + Pinia + JSONForms)          │
├──────────────────────────────────────────────────────────────┤
│  Backend  (Node.js + Fastify + Prisma ORM)                   │
├──────────────────────────────────────────────────────────────┤
│  Base de datos  (PostgreSQL + JSONB para schemas de forms)  │
├──────────────────────────────────────────────────────────────┤
│  Agente IA  (DSPy + LLM configurable)                      │
├──────────────────────────────────────────────────────────────┤
│  Conector MSP  (API REST + MCP para agentes externos)        │
└──────────────────────────────────────────────────────────────┘
```

### Capas core

- **Form Builder Engine** — Registry de JSON Schema versionado con categorías y reglas de aplicabilidad (clase de riesgo, sector, número de empleados, jurisdicción). Formularios bloqueados protegen requisitos regulatorios obligatorios.
- **Template Engine** — Generación de menú adaptativa por perfil. Perfil de empresa → template aplicable → conjunto mínimo de formularios relevantes. Sin sobrecarga.
- **Sistema de Vencimientos** — Alertas automáticas de expiración para documentos, certificaciones y renovaciones periódicas. Notificaciones por email + dashboard.
- **Agente IA** — Genera formularios desde texto normativo bruto. Asiste durante el diligenciamiento. Valida cumplimiento cruzado cuando las normas cambian.
- **Conector MSP** — API REST + servidor MCP para MSPs que gestionan múltiples empresas cliente. Operaciones en masa, dashboards, integración con agentes IA externos.

## Concepto central

Los formularios siguen el perfil de la empresa:

```
Company Profile {
  clase_riesgo: "III"
  sector: "construccion"      // código CIIU
  num_empleados: 45
  jurisdiccion: "CO"
}
  → Template: "empresa-grande-construccion"
  → Forms: 18 formularios relevantes (de ~30 en el registry)
  → Menú: solo lo que aplica
```

## Documentación

Documentación técnica completa en [`docs/es/`](docs/es/).

## Licencia

**Mozilla Public License 2.0 (MPL-2.0).** Permite uso comercial, modificación y redistribución. Misma licencia que Mozilla Firefox.

## Estado

- Repositorio: desarrollo activo (octubre 2026)
- Producción: no
- Financiación: abierta a grants open source y sponsorship comunitario
- Maintainer: [Yaqui Martinez](https://github.com/mono079790-lab) (persona natural, Colombia)

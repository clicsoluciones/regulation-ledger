# regulation-ledger

> Open-source immutable evidence ledger and legislative snapshot engine for compliance systems.

[English](#english) · [Español](#espa%C3%B1ol)

---

## English

`regulation-ledger` is an open-source library and HTTP API for building regulatory compliance systems with verifiable audit trails. The project addresses a concrete problem: organizations that must comply with evolving regulation (tax, labor, environmental, health and safety, data protection) need to demonstrate which law applied to each event or decision at any given date, and that the record cannot be altered after the fact.

### Why this project matters

1. **Continuous audit replaces periodic audit.** Companies spend millions on external audits that could be self-audited continuously with cryptographic proof.
2. **Regulation changes over time.** Most companies maintain evidence but cannot tell which law applied to a given record on a given date. `regulation-ledger` seals that information together with the evidence.
3. **Compliance data is sensitive.** Most current solutions are SaaS. `regulation-ledger` is on-premise by design, viable for sectors with strict data residency requirements (health, finance, defense, government).

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  ledger-api  (Fastify HTTP API, OpenAPI spec)               │
├─────────────────────────────────────────────────────────────┤
│  compliance-validate  (declarative rules engine, YAML/JSON) │
├─────────────────────────────────────────────────────────────┤
│  regulation-snapshot  (versioned legal constants registry)  │
├─────────────────────────────────────────────────────────────┤
│  evidence-core  (hash chain, immutable records, signatures) │
└─────────────────────────────────────────────────────────────┘
```

### Modules

- **evidence-core** — hash chain engine for immutable records. Each record carries timestamp, payload, prev_hash, hash, and signature. Detects tampering. Exports audit trails.
- **regulation-snapshot** — versioned legal constants registry. Register a norm with its effective dates; query which norm applied to a specific date. Initial domain: Colombian occupational health and safety (Resolución 0312/2019, Decreto 1072/2015, Ley 1562/2012). Architecture open for any regulatory domain.
- **compliance-validate** — on-device rule engine that runs declarative rules (YAML/JSON) without cloud services. Critical for sensitive data.
- **ledger-api** — REST API exposing the three libraries above. OpenAPI spec. Docker compose for on-premise deploy.

### Roadmap (6 months)

| Milestone | Month | Deliverable |
|---|---|---|
| H1: evidence-core v0.1 | 1 | Node/TS library with hash chain + tests + docs |
| H2: regulation-snapshot v0.1 | 2 | Versioned legal constants + SST seed data |
| H3: integration + compliance-validate v0.1 | 3 | Library integration + declarative rules engine |
| H4: ledger-api v0.1 | 4 | Fastify HTTP API + OpenAPI + Docker compose |
| H5: docs + landing + demo | 5 | Bilingual README, technical docs, landing page, public demo |
| H6: public release + community | 6 | GitHub v1.0 release, CONTRIBUTING.md, first external issue processed |

### Use case: Colombian SST

600,000+ Colombian companies must comply with Decreto 1072/2015 and Resolución 0312/2019 for Occupational Health and Safety Management Systems (SG-SST). Today, evidence is scattered in local folders, loose PDFs, with no cryptographic traceability. With `regulation-ledger`, each event (training, medical exam, internal audit) generates an immutable record with the applicable law sealed.

### License

**Mozilla Public License 2.0 (MPL-2.0).** Permits commercial use, modification, and redistribution, with the condition that improvements to MPL-licensed files be made available under the same license. Same license used by Mozilla for Firefox and other tools.

### Status

- Repository: skeleton (October 2026)
- Production: no
- Funding: pending open-source grants and community sponsorship application
- Maintainer: [Yaqui Martinez](https://github.com/mono079790-lab) (persona natural, Colombia)

---

## Español

`regulation-ledger` es una librería y API HTTP open source para construir sistemas de cumplimiento regulatorio con auditorías verificables. El proyecto resuelve un problema concreto: las organizaciones que deben cumplir regulación cambiante (tributaria, laboral, ambiental, sanitaria, de protección de datos) necesitan demostrar qué norma aplicaba a cada evento o decisión en una fecha dada, y que el registro no pueda ser alterado después.

### Por qué importa este proyecto

1. **Auditoría continua reemplaza auditoría periódica.** Las empresas gastan millones en auditorías externas que podrían auto-auditarse continuamente con prueba criptográfica.
2. **La regulación cambia con el tiempo.** La mayoría de empresas mantienen evidencia pero no pueden decir qué ley aplicaba a un registro en una fecha dada. `regulation-ledger` sella esa información con la evidencia.
3. **Los datos de compliance son sensibles.** La mayoría de soluciones actuales son SaaS. `regulation-ledger` es on-premise por diseño, viable para sectores con requisitos estrictos de residencia de datos (salud, financiero, defensa, gobierno).

### Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│  ledger-api  (API HTTP con Fastify, spec OpenAPI)            │
├─────────────────────────────────────────────────────────────┤
│  compliance-validate  (motor de reglas declarativas YAML/JSON)│
├─────────────────────────────────────────────────────────────┤
│  regulation-snapshot  (registro versionado de constantes    │
│                         legales)                            │
├─────────────────────────────────────────────────────────────┤
│  evidence-core  (hash chain, registros inmutables, firmas)  │
└─────────────────────────────────────────────────────────────┘
```

### Módulos

- **evidence-core** — motor de hash chain para registros inmutables. Cada registro lleva timestamp, payload, prev_hash, hash y firma. Detecta manipulación. Exporta audit trails.
- **regulation-snapshot** — registro versionado de constantes legales. Registra una norma con sus fechas de vigencia; consulta qué norma aplicaba a una fecha específica. Dominio inicial: seguridad y salud en el trabajo colombiana (Resolución 0312/2019, Decreto 1072/2015, Ley 1562/2012). Arquitectura abierta para cualquier dominio regulatorio.
- **compliance-validate** — motor de validación on-device que ejecuta reglas declarativas (YAML/JSON) sin servicios cloud. Crítico para datos sensibles.
- **ledger-api** — API REST que expone las tres librerías anteriores. Spec OpenAPI. Docker compose para deploy on-premise.

### Roadmap (6 meses)

| Hito | Mes | Entregable |
|---|---|---|
| H1: evidence-core v0.1 | 1 | Librería Node/TS con hash chain + tests + docs |
| H2: regulation-snapshot v0.1 | 2 | Constantes legales versionadas + seed data SST |
| H3: integration + compliance-validate v0.1 | 3 | Integración de librerías + motor de reglas declarativas |
| H4: ledger-api v0.1 | 4 | API HTTP Fastify + OpenAPI + Docker compose |
| H5: docs + landing + demo | 5 | README bilingüe, docs técnicas, landing page, demo pública |
| H6: release público + comunidad | 6 | GitHub v1.0 release, CONTRIBUTING.md, primer issue externo procesado |

### Caso de uso: SST colombiana

Más de 600.000 empresas colombianas deben cumplir el Decreto 1072/2015 y la Resolución 0312/2019 para el Sistema de Gestión de Seguridad y Salud en el Trabajo (SG-SST). Hoy, la evidencia está dispersa en carpetas locales, PDFs sueltos, sin trazabilidad criptográfica. Con `regulation-ledger`, cada evento (capacitación, examen médico, auditoría interna) genera un registro inmutable con la norma aplicable sellada.

### Licencia

**Mozilla Public License 2.0 (MPL-2.0).** Permite uso comercial, modificación y redistribución, con la condición de que las mejoras a archivos bajo MPL se pongan a disposición bajo la misma licencia. Misma licencia que usa Mozilla para Firefox y otras herramientas.

### Estado

- Repositorio: esqueleto (octubre 2026)
- Producción: no
- Financiación: abierta a grants open source y sponsorship comunitario
- Maintainer: [Yaqui Martinez](https://github.com/mono079790-lab) (persona natural, Colombia)

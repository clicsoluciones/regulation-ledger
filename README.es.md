# regulation-ledger

Sistema open source de gestión de cumplimiento normativo con formularios adaptativos,
templates regulatorios versionados y seguimiento de evidencia auditable para
cumplimiento regulatorio multi-jurisdiccional.

[English](README.md)

---

## Acerca del proyecto

`regulation-ledger` resuelve un problema concreto: las empresas enfrentan formularios regulatorios abrumadores que no coinciden con su perfil real — tamaño incorrecto, clase de riesgo, o sector — generando confusión, incumplimiento y desperdicio de recursos.

### Por qué importa este proyecto

1. **Los templates adaptativos reemplazan el enfoque único.** Una microempresa de 5 empleados y una constructora de 500 tienen necesidades de cumplimiento radicalmente diferentes. `regulation-ledger` genera un conjunto mínimo y relevante de formularios basado en el perfil de la empresa.

2. **El registry es el producto.** Los formularios viven en un registry versionado de JSON Schema, extensible por el owner de la plataforma y por MSPs que gestionan múltiples clientes. Los cambios normativos se registran como snapshots inmutables, así que cada formulario indica contra qué versión de qué regla fue construido.

3. **Los datos de compliance se quedan on-premise.** Diseñado para auto-hosting. Para sectores con requisitos estrictos de residencia de datos u organizaciones que requieren control total de su evidencia.

4. **La evidencia es verificable, no solo se guarda.** Cada registro se encadena por hash, de modo que un auditor puede reconstruir qué se presentó, cuándo y contra qué versión normativa.

## Arquitectura

```
┌──────────────────────────────────────────────────────────────┐
│  Frontend  (Vue 3 + TypeScript + Pinia + JSONForms)          │
├──────────────────────────────────────────────────────────────┤
│  Backend   (Node.js + Fastify + Prisma ORM)                  │
├──────────────────────────────────────────────────────────────┤
│  Base de datos  (PostgreSQL + JSONB para schemas de forms)  │
├──────────────────────────────────────────────────────────────┤
│  Registry  (JSON Schema versionado + snapshots normativos)   │
├──────────────────────────────────────────────────────────────┤
│  Evidencia  (cadena de hashes, solo-append)                  │
├──────────────────────────────────────────────────────────────┤
│  Conector MSP  (API REST + MCP para integraciones)           │
└──────────────────────────────────────────────────────────────┘
```

### Capas core

- **Form Builder Engine** — Registry de JSON Schema versionado con categorías y reglas de aplicabilidad (clase de riesgo, sector, número de empleados, jurisdicción). Formularios bloqueados protegen requisitos regulatorios obligatorios.
- **Template Engine** — Generación de menú adaptativa por perfil. Perfil de empresa → template aplicable → conjunto mínimo de formularios relevantes. Sin sobrecarga.
- **Snapshots normativos** — Cada versión de regla se captura como snapshot inmutable. Cuando una norma cambia, los formularios afectados se marcan en vez de derivar silenciosamente.
- **Sistema de Vencimientos** — Alertas automáticas de expiración para documentos, certificaciones y renovaciones periódicas. Notificaciones por email + dashboard.
- **Cadena de evidencia** — Registro append-only encadenado por hash de presentaciones y documentos de soporte.
- **Conector MSP** — API REST + servidor MCP para MSPs que gestionan múltiples empresas cliente. Operaciones en masa, dashboards, integraciones externas.

Las capas deterministas anteriores son el núcleo del proyecto. Un asistente opcional, local y apagado por defecto está disponible como extensión separada para asistir en la autoría de formularios; no es necesario para operar el sistema ni forma parte del roadmap core.

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

## Desarrollo y transparencia

**Uso pasado de IA generativa.** La especificación técnica y la documentación de este
repositorio fueron redactadas con asistencia de IA antes de que el proyecto adoptara una
política escrita sobre IA generativa. Esa asistencia se declara aquí para el registro, y
el contenido subyacente fue revisado y es responsabilidad del maintainer.

**De ahora en adelante.** El código y la documentación nuevos se escriben a mano. No se
usa IA generativa para producir código, diseño ni documentación en este repositorio.
Cuando se usa herramienta de IA en modo read-only (por ejemplo, análisis estático de
código existente), se declara commit a commit en el mensaje del commit.

Se pide a los colaboradores indicar en el mensaje del commit si alguna parte del cambio
fue generada por máquina, y asumir la responsabilidad del resultado.

## Extensión opcional de asistencia

Existe una extensión opcional con un asistente local para la autoría de formularios: se
ejecuta apagada por defecto, requiere un modelo alojado localmente y no es necesaria para
operar ninguna capa core. Las capas core — registry, template engine, snapshots
normativos, cadena de evidencia, conector MSP — son completamente deterministas y
funcionan sin ella.

## Estado

- Repositorio: desarrollo activo (octubre 2026)
- Producción: no
- Financiación: abierta a grants open source y sponsorship comunitario (detalles en el roadmap del maintainer)
- Maintainer: [Yaqui Martinez](https://github.com/mono079790-lab) (persona natural, Colombia)

# Arquitectura

`regulation-ledger` sigue una arquitectura estrictamente por capas, de abajo hacia arriba.
Cada capa depende solo de las capas inferiores, nunca de las superiores.

## Capa 1: `evidence-core`

La base. Aporta el modelo de datos y las primitivas para crear registros inmutables con
integridad criptográfica.

**Modelo de datos:**
```

typescript
interface EvidenceRecord {
  id: string;                 // UUID v4
  timestamp: string;          // ISO 8601 con zona horaria
  domain: string;             // p. ej. "sst.colombia", "tax.colombia"
  payload: Record<string, unknown>;
  prev_hash: string;          // SHA-256 hex del registro anterior ("0".repeat(64) en el génesis)
  hash: string;               // SHA-256 hex de la forma canónica de este registro
  signature?: string;         // firma Ed25519 opcional
}
```

**Superficie de API:**

- `appendEvidence(record): EvidenceRecord`, valida el encadenamiento, calcula el hash y devuelve el registro añadido.
- `verifyChain(records): { valid: boolean; brokenAt?: number }`, detecta manipulación.
- `exportAuditTrail(records, format): Buffer`, exporta a JSON, CSV o PDF.
- `signRecord(record, privateKey): EvidenceRecord`, añade firma Ed25519.
- `verifySignature(record, publicKey): boolean`, verifica la firma Ed25519.

## Capa 2: `regulation-snapshot`

Registro versionado de constantes legales. Una norma tiene una fecha `effective_from` y
una `effective_to` opcional. El sistema puede responder qué valor tenía la constante X en
la fecha Y para el dominio Z.

**Modelo de datos:**
```

typescript
interface LegalConstant {
  id: string;
  domain: string;             // p. ej. "sst.colombia"
  key: string;                // p. ej. "max_training_hours_per_year"
  value: unknown;             // cualquier valor serializable a JSON
  effective_from: string;     // fecha ISO
  effective_to?: string;      // fecha ISO; indefinida = sigue vigente
  source_url?: string;        // URL a la publicación oficial
  source_title?: string;      // p. ej. "Resolución 0312/2019"
  jurisdiction: string;       // p. ej. "CO", "EU", "MX"
}
```

**Superficie de API:**

- `registerConstant(constant): void`, añade al registro; falla si se solapa con uno existente.
- `getConstantsForDate(domain, date, jurisdiction?): LegalConstant[]`, constantes vigentes en una fecha.
- `getConstantHistory(domain, key): LegalConstant[]`, todas las versiones de una constante en el tiempo.
- `attachLegislationToRecord(record, constants): EvidenceRecord`, extiende un registro de `evidence-core` con la instantánea normativa aplicable.

## Capa 3: `compliance-validate`

Motor de reglas declarativo. Las reglas son archivos YAML o JSON que describen condiciones.
El motor evalúa un registro, o un conjunto, contra las reglas y produce un informe de
cumplimiento.

**Ejemplo de regla:**
```

yaml
id: sst.training.has_valid_date
domain: sst.colombia
severity: error
description: "El evento de formación debe tener fecha dentro del año fiscal en curso"
when:
  record.type: "training_event"
assert:
  - year(record.date) == year(now)
source: "Resolución 0312/2019 Art. 18"
```

**Superficie de API:**

- `loadRules(rulesPath): Rule[]`, carga desde archivo o directorio.
- `validate(record, rules): ComplianceReport`, devuelve un informe estructurado.
- `validateBatch(records, rules): ComplianceReport[]`, validación por lote.
- `exportReport(report, format): Buffer`, JSON, PDF o HTML.

## Capa 4: `ledger-api`

API HTTP que expone las tres librerías anteriores. Construida sobre Fastify.

**Endpoints (v0.1 prevista):**

- `POST /evidence`, añade un registro de evidencia.
- `GET /evidence/:id`, recupera un registro por id.
- `GET /evidence/verify?from=...&to=...`, verifica un segmento de la cadena.
- `GET /snapshot/:domain?date=...&jurisdiction=...`, instantánea normativa aplicable.
- `POST /validate`, ejecuta la validación de cumplimiento.
- `GET /health`, comprobación de estado.

**Despliegue:**

Docker Compose para despliegue en infraestructura propia. Contenedor más volumen
persistente para el registro.

## Ejemplo de flujo de datos
```
El usuario sube un evento de formación
        │
        ▼
┌─────────────────────────────────────────────────────┐
│ ledger-api.POST /evidence                           │
│                                                     │
│  1. Obtiene la instantánea normativa de fecha+dominio│
│     → regulation-snapshot.getConstantsForDate()     │
│                                                     │
│  2. Adjunta la instantánea al registro              │
│     → regulation-snapshot.attachLegislationToRec()  │
│                                                     │
│  3. Añade el registro a la cadena                   │
│     → evidence-core.appendEvidence()                │
│                                                     │
│  4. Opcionalmente valida contra las reglas          │
│     → compliance-validate.validate()                │
│                                                     │
│  5. Devuelve el registro persistido + informe       │
└─────────────────────────────────────────────────────┘
        │
        ▼
Guardado en volumen persistente local
```

## Por qué MPL-2.0

- **Compatible con uso comercial.** Las empresas pueden construir productos propietarios
 encima sin abrir su código.
- **Las mejoras al núcleo vuelven al núcleo.** Si alguien modifica un archivo de
 `regulation-ledger` y lo distribuye, la modificación debe publicarse bajo MPL-2.0.
- **Copyleft a nivel de archivo.** A diferencia de GPL, MPL-2.0 no exige abrir toda la
 obra derivada: solo los archivos con licencia MPL que hayan sido modificados.

Es la misma licencia que Mozilla usa para Firefox, Thunderbird y el motor Servo.

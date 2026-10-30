# Architecture

`regulation-ledger` follows a strict bottom-up layered architecture. Each layer depends only on the layers below it, never above.

## Layer 1: `evidence-core`

The foundation. Provides the data model and primitives for creating immutable records with cryptographic integrity.

**Data model:**
```

typescript
interface EvidenceRecord {
  id: string;                 // UUID v4
  timestamp: string;          // ISO 8601 with timezone
  domain: string;             // e.g. "sst.colombia", "tax.colombia"
  payload: Record<string, unknown>;
  prev_hash: string;          // SHA-256 hex of previous record (or "0".repeat(64) for genesis)
  hash: string;               // SHA-256 hex of this record's canonical form
  signature?: string;         // optional Ed25519 signature
}
```

**API surface:**

- `appendEvidence(record): EvidenceRecord`, validates encadenamiento, computes hash, returns appended record.
- `verifyChain(records): { valid: boolean; brokenAt?: number }`, detects tampering.
- `exportAuditTrail(records, format): Buffer`, exports to JSON, CSV, or PDF.
- `signRecord(record, privateKey): EvidenceRecord`, adds Ed25519 signature.
- `verifySignature(record, publicKey): boolean`, verifies Ed25519 signature.

## Layer 2: `regulation-snapshot`

Versioned registry of legal constants. A norm has an effective_from and optional effective_to date. The system can answer "what was the value of constant X on date Y in domain Z".

**Data model:**
```

typescript
interface LegalConstant {
  id: string;
  domain: string;             // e.g. "sst.colombia"
  key: string;                // e.g. "max_training_hours_per_year"
  value: unknown;             // any JSON-serializable value
  effective_from: string;     // ISO date
  effective_to?: string;      // ISO date, undefined = still in force
  source_url?: string;        // URL to official publication
  source_title?: string;      // e.g. "Resolución 0312/2019"
  jurisdiction: string;       // e.g. "CO", "EU", "MX"
}
```

**API surface:**

- `registerConstant(constant): void`, adds to registry, fails if overlaps with existing.
- `getConstantsForDate(domain, date, jurisdiction?): LegalConstant[]`, returns all constants effective on a given date.
- `getConstantHistory(domain, key): LegalConstant[]`, returns all versions of a constant over time.
- `attachLegislationToRecord(record, constants): EvidenceRecord`, extends an evidence-core record with the applicable law snapshot.

## Layer 3: `compliance-validate`

Declarative rules engine. Rules are YAML or JSON files describing conditions. The engine evaluates a record (or set of records) against the rules and produces a compliance report.

**Rule example:**
```

yaml
id: sst.training.has_valid_date
domain: sst.colombia
severity: error
description: "Training event must have a date within the current fiscal year"
when:
  record.type: "training_event"
assert:
  - year(record.date) == year(now)
source: "Resolución 0312/2019 Art. 18"
```

**API surface:**

- `loadRules(rulesPath): Rule[]`, loads from file or directory.
- `validate(record, rules): ComplianceReport`, returns a structured report.
- `validateBatch(records, rules): ComplianceReport[]`, bulk validation.
- `exportReport(report, format): Buffer`, JSON, PDF, or HTML.

## Layer 4: `ledger-api`

HTTP API exposing the three libraries above. Built on Fastify for performance and modern async/await semantics.

**Endpoints (planned v0.1):**

- `POST /evidence`, append a new evidence record.
- `GET /evidence/:id`, retrieve a record by id.
- `GET /evidence/verify?from=...&to=...`, verify a chain segment.
- `GET /snapshot/:domain?date=...&jurisdiction=...`, get applicable law snapshot.
- `POST /validate`, run compliance validation.
- `GET /health`, health check.

**Deployment:**

Docker compose for on-premise deploy. Single binary or container + persistent volume for the registry.

## Data flow example
```
User uploads training event
        │
        ▼
┌─────────────────────────────────────────────────────┐
│ ledger-api.POST /evidence                          │
│                                                     │
│  1. Get applicable law snapshot for date+domain     │
│     → regulation-snapshot.getConstantsForDate()    │
│                                                     │
│  2. Attach law snapshot to record                  │
│     → regulation-snapshot.attachLegislationToRec() │
│                                                     │
│  3. Append record to chain                         │
│     → evidence-core.appendEvidence()               │
│                                                     │
│  4. Optionally validate against rules              │
│     → compliance-validate.validate()               │
│                                                     │
│  5. Return persisted record + validation report    │
└─────────────────────────────────────────────────────┘
        │
        ▼
Stored in local persistent volume
```

## Why MPL-2.0

- **Commercial-friendly.** Companies can build proprietary products on top without opening their code.
- **Improvements to the core return to the core.** If someone modifies a `regulation-ledger` file and distributes it, the modification must be released under MPL-2.0. This prevents proprietary forks of the core itself.
- **File-level copyleft.** Unlike GPL, MPL-2.0 doesn't require that the entire derivative work be open. Only the modified MPL-licensed files need to be released.

This is the same license used by Mozilla for Firefox, Thunderbird, and the Servo browser engine.

# Arquitectura Modular — Regulation Ledger

> Fecha: 08-oct-2026
> Autor: Yaqui Martinez
> Proyecto: regulation-ledger

---

## Diagnóstico: El Problema del Mercado Actual

Herramientas SG-SST con módulos sueltos = carga cognitiva excesiva para usuarios sin conocimiento normativo.
Una PYME con 5 empleados y riesgo clase II ve los mismos módulos que una constructora con 200 y riesgo V.
El resultado: confusión, frustración, abandono.

El problema de arquitectura no se resuelve solo con módulos de IA.

---

## Solución: Tres Bloques Arquitectónicos

### Bloque 1 — Form Builder Engine (Core Dinámico)

**Propósito:** Sistema de construcción de formularios dinámica.

**Enfoque:**
- Form Builder open-source como base (ej: FormBuilder, JSONForms, or similar)
- Base de datos que registra cada formulario por: categoría, clase de riesgo, sector, jurisdicción
- Formularios inmutables (requisitos legales) marcados como `locked: true` — no se eliminan
- Menú del sistema se genera dinámicamente desde el registry

**Estructura del registry:**

```
Form {
  id: uuid
  slug: "evaluacion-inicial-sst"
  nombre: "Evaluación inicial SG-SST"
  categoria: "cumplimiento"
 locked: true                          // requisito legal, no eliminable
  applicable_to: {                     // determina cuándo aplica
    clases_riesgo: [I, II, III, IV, V]
    sectores: ["todos"]
    min_empleados: 1
  }
  fields: [...]                        // JSON Schema del formulario
  normativa_refs: ["Resolucion 0312 Art.2.2.4.6.5"]
}
```

**Beneficio:** El menú se adapta solo. Si la empresa es clase I, ve los forms que le aplican.

---

### Bloque 2 — Wizard + Template Engine (UX Minimalista)

**Propósito:** Eliminar la sobrecarga inicial. El usuario solo ve lo que necesita.

**Flujo:**
1. **Wizard de onboarding** → empresa configura perfil (clase riesgo, sector, empleados, ubicación)
2. **Template por perfil** → sistema genera menú minimalista con forms aplicables
3. **Personalización** → usuario puede agregar forms adicionales desde el registry

**Ejemplo:**

```
Empresa nueva: riesgo III, construcción, 30 empleados
→ Wizard responde: 12 forms aplicables
→ Menú muestra solo esos 12
→ Los 101 restantes: disponibles para agregar desde el registry
```

**Templates por defecto:**
- Microempresa (1-5 empleados, riesgo I-II)
- PYME (6-50 empleados, riesgo I-III)
- Empresa grande (51+ empleados, riesgo III-V)
- Construcción (independiente de tamaño)
- Salud (independiente de tamaño)
- Manufactura (independiente de tamaño)
- Industrial (independiente de tamaño)

Cada template es un conjunto de forms + el orden sugerido de diligenciamiento.

**Beneficio:** Costo operativo reducido. El usuario no paga por lo que no necesita.

---

### Bloque 3 — Agente IA para Construcción Dinámica de Formularios

**Propósito:** El sistema crece solo. No necesita ciclos de desarrollo tradicionales.

**Capacidades:**

1. **Generación de formularios por normativa nueva**
   - Input: texto de una resolución/ley
   - Output: JSON Schema del formulario + metadata
   - Agente procesa el texto, extrae campos requeridos, genera form

2. **Asistencia al usuario en diligenciamiento**
   - El agente guía al usuario campo por campo
   - Explica cada requisito legal en lenguaje simple
   - Sugiere evidencia/documentación necesaria

3. **Validación cruzada normativa**
   - Cuando cambia una norma, el agente reevalúa los forms afectados
   - Propone actualizaciones al registry

**Arquitectura del agente:**
- LLM como motor (compatible con infraestructura open-source)
- Tools: web_search (normativas), document_analysis, json_schema_generator
- Plugin system para adicionar nuevas capacidades

---

## Bloque 4 — Motor Normativo por Región (Futuro V2+)

**Propósito:** No solo Colombia. El sistema escala a otras jurisdicciones.

**Flujo:**
1. Identificarnormativa por región/país
2. Extraer requisitos aplicables
3. Generar template regional
4. El registry acepta `jurisdiccion: "CO"` | `"MX"` | `"PE"` etc.

**Ejemplo:**
```
Jurisdicción: Colombia
  → Resolución 0312/2019
  → Decreto 1072/2015
  → Resolución 1409/2012 (alturas)
  → GTC 45:2012

Jurisdicción: México
  → NOM-035-STPS-2018 (psicosocial)
  → NOM-030-STPS-2009 (servicios preventivos)
```

---

## Impacto en Desarrollo

### Fase 1 (V1.0) — MVP
- Form Builder engine con registry básico
- 10-15 formularios locked (los más críticos)
- Wizard de onboarding con 2 templates
- Sin agente IA (IA viene en V2)

### Fase 2 (V1.1) — Expansión
-剩余 formularios locked hasta ~40
- Templates adicionales por sector
- Menú dinámico desde registry

### Fase 3 (V2.0) — IA
- Agente de generación de formularios
- Asistencia en diligenciamiento
- Validación cruzada normativa

### Fase 4 (V2.1+) — Multi-región
- Motor de normativas por jurisdicción
- Templates internacionales
- Registry federado

---

## Decisiones Arquitectónicas Clave

| Decisión | Selección | Justificación |
|----------|----------|---------------|
| Form Builder base | Open-source (JSONForms o similar) | Evitar reinventar la rueda, comunidad activa |
| Almacenamiento forms | JSON Schema en DB | Portabilidad, versionable, estándar |
| Motor de templates | Database-driven | Cambio sin código, admins pueden crear |
| Agente IA | Herramienta con LLM configurable | Plug-and-play con el mejor modelo disponible |
| Jurisdicción | Colombia primero | Mercado inmediato, después escalar |
| lock vs. unlock | locked=requisito legal | Garantiza compliance, no se puede borrar lo requerido |

---

## Resumen

| Bloque | Complejidad | Prioridad |
|--------|-------------|-----------|
| Form Builder Engine | Alta | V1.0 |
| Wizard + Templates | Media | V1.0 |
| Agente IA | Alta | V2.0 |
| Motor Normativo Regional | Alta | V2.1+ |

El core de regulation-ledger NO es el formulario individual — es el **registry + motor de templates + agente IA**. Los formularios son consecuencia, no el producto.

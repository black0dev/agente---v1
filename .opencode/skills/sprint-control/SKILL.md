---
name: sprint-control
description: Skill de control de sprints para Plan2Ship. Mantiene el estado de cada sprint del Blueprint, fuerza el ciclo completo Review→QA→Merge por sprint, y asegura la continuación automática hasta completar todos los sprints. Use ONLY when managing sprint lifecycle in Plan2Ship.
---

# Skill: Sprint Control

Este skill es el **motor principal y orquestador** del ciclo Plan2Ship.

## Arquitectura de Skills (LEER PRIMERO)

```
sprint-control  ← MOTOR PRINCIPAL (este skill)
│   Orquesta el ciclo completo sprint por sprint
│   Mantiene SPRINT_STATUS.md
│   Fuerza: BUILD → QA → REVIEW → MERGE → DOD
│   Inicia el siguiente sprint automáticamente
│
├── build           ← MOTOR DE CONSTRUCCIÓN (fase BUILD de cada sprint)
│   │   Invocado POR sprint-control en la fase de construcción
│   │   Divide el trabajo entre subagentes
│   │   Crea ramas feature/* y coordina DB→Backend→Frontend
│   │   Verifica ausencia de placeholders antes de terminar
│
├── testing         ← MOTOR DE PRUEBAS (fase QA de cada sprint)
│   Invocado POR sprint-control después del BUILD
│
├── review          ← MOTOR DE REVISIÓN (fase REVIEW de cada sprint)
│   Invocado POR sprint-control después del QA
│
├── merge           ← MOTOR DE MERGE (fase MERGE de cada sprint)
│   Invocado POR sprint-control después del REVIEW
│
├── blueprint-validation  ← VALIDACIÓN CRUZADA (usado por review y build)
│
├── branch-management     ← GESTIÓN DE RAMAS (usado por build)
│
└── deploy          ← DESPLIEGUE (activado al completar TODOS los sprints)
```

**Regla de uso**: El comando `/plan2ship-build` activa `sprint-control` como punto de entrada. `sprint-control` invoca `build` internamente para la fase de construcción. Nunca se debe invocar `build` directamente para el ciclo completo — siempre a través de `sprint-control`.

## Por Qué Existe Este Skill

Sin este skill, los agentes:
- Declaran "terminado" después del primer sprint parcial
- No invocan Reviewer, QA ni Merge Supervisor
- Aceptan placeholders como entregas válidas
- No continúan con el siguiente sprint

Este skill **obliga** el ciclo correcto y **no permite** avanzar sin cumplir cada gate de calidad.

## Responsabilidades

1. Leer el Blueprint y extraer todos los sprints
2. Crear y mantener el archivo `SPRINT_STATUS.md` con el estado de cada sprint
3. Ejecutar el gate de calidad al cierre de cada sprint
4. Forzar: Reviewer → QA → Merge Supervisor antes de cerrar cualquier sprint
5. Iniciar el siguiente sprint automáticamente al aprobar el actual
6. Reportar bloqueos que requieran intervención humana

---

## Paso 1: Inicializar el Tablero de Sprints

Al activarse por primera vez, leer el Project Blueprint e identificar:

```markdown
# SPRINT_STATUS.md — Tablero de Control

| Sprint | Nombre | Estado | DoD Firmado | QA | Review | Merge |
|--------|--------|--------|-------------|----|--------|-------|
| 01 | Fundación | PENDIENTE | ❌ | ❌ | ❌ | ❌ |
| 02 | Catálogo | PENDIENTE | ❌ | ❌ | ❌ | ❌ |
| 03 | Carrito/Checkout | PENDIENTE | ❌ | ❌ | ❌ | ❌ |
| 04 | Panel Admin | PENDIENTE | ❌ | ❌ | ❌ | ❌ |
| 05 | Integraciones | PENDIENTE | ❌ | ❌ | ❌ | ❌ |
| 06 | Calidad/Despliegue | PENDIENTE | ❌ | ❌ | ❌ | ❌ |
```

Crear este archivo en la raíz del proyecto y actualizarlo en cada paso.

---

## Paso 2: Ciclo de Sprint Obligatorio

Para **cada sprint** del Blueprint, ejecutar este ciclo **sin excepción**:

```
┌─────────────────────────────────────────────────────────────┐
│ CICLO OBLIGATORIO POR SPRINT                                 │
│                                                             │
│  1. BUILD    → Subagentes implementan en ramas feature/*    │
│       ↓                                                     │
│  2. QA       → qa-agent prueba con datos REALES             │
│       ↓                                                     │
│  3. REVIEW   → reviewer-agent audita contra Blueprint       │
│       ↓                                                     │
│  4. MERGE    → merge-supervisor-agent aprueba o rechaza     │
│       ↓                                                     │
│  5. DOD      → Firmar Definition of Done del sprint         │
│       ↓                                                     │
│  6. NEXT     → Iniciar siguiente sprint automáticamente     │
└─────────────────────────────────────────────────────────────┘
```

**REGLA CRÍTICA**: Un sprint NO puede declararse terminado hasta que pasen los 5 gates (QA ✅, Review ✅, Merge ✅, DoD ✅, y todos los criterios de aceptación del Blueprint verificados).

---

## Paso 3: Definition of Done (DoD) por Sprint

El DoD es un checklist que TODOS los sprints deben cumplir antes de cerrarse:

### DoD Universal (aplica a TODOS los sprints)

```
[ ] Las páginas de UI muestran datos REALES (no placeholders ni hardcoded)
[ ] Los endpoints existen y responden correctamente con datos de BD
[ ] Las pruebas mínimas del Blueprint pasan (QA Agent aprobó)
[ ] El Reviewer firmó el cumplimiento del Blueprint
[ ] El Merge Supervisor aprobó la integración
[ ] No hay botones `disabled` sin funcionalidad en el mismo sprint
[ ] No hay texto tipo "Aquí se mostrarán los datos" en producción
[ ] No hay datos hardcoded en responses de API (arrays/objetos fijos)
```

### DoD por Sprint (específico del Blueprint)

Leer el Blueprint y extraer los criterios de aceptación del sprint actual. Cada criterio es un ítem del DoD específico.

---

## Paso 4: Gate Anti-Placeholder

Antes de aprobar cualquier sprint, ejecutar este escaneo:

```bash
# Detectar placeholders en frontend
grep -rn "se mostrar" web/src/ --include="*.tsx" --include="*.jsx"
grep -rn "Aquí" web/src/ --include="*.tsx" --include="*.jsx"
grep -rn "placeholder" web/src/app/ --include="*.tsx" --include="*.jsx"
grep -rn "TODO" web/src/app/ --include="*.tsx" --include="*.jsx"
grep -rn "hardcoded\|HARDCODED\|ficti" web/src/ --include="*.tsx"

# Detectar datos hardcoded en backend
grep -rn "\"ORD-001\"\|\"ORD-002\"\|\"Cliente ejemplo\"" api/ --include="*.py"
grep -rn "hardcoded\|ficti" api/ --include="*.py"

# Verificar que hay llamadas reales a BD/API
grep -rn "supabase\.\|fetch(\|axios\." web/src/app/ --include="*.tsx"
```

Si alguno de estos checks falla (encontró placeholders o NO encontró llamadas reales), el sprint se marca como **BLOQUEADO** y no se puede continuar.

---

## Paso 5: Protocolo de Continuación Automática

Cuando un sprint es aprobado por el Merge Supervisor:

```
1. Actualizar SPRINT_STATUS.md → sprint actual: COMPLETADO ✅
2. Leer el siguiente sprint del Blueprint
3. Si existe siguiente sprint:
   → Iniciar BUILD del siguiente sprint automáticamente
   → Actualizar SPRINT_STATUS.md → siguiente sprint: EN PROGRESO
4. Si no existe siguiente sprint:
   → Reportar: "Todos los sprints del Blueprint completados"
   → Activar skill `deploy` para el ciclo SHIP
```

**NUNCA** detenerse entre sprints sin razón explícita. Si hay bloqueo, reportarlo con detalle al usuario.

---

## Paso 6: Manejo de Bloqueos

Si un sprint no puede completarse, reportar exactamente:

```markdown
## 🚫 SPRINT BLOQUEADO: Sprint {N} — {Nombre}

### Motivo del Bloqueo
{descripción exacta del problema}

### Evidencia
- QA: {resultado}
- Review: {resultado}
- Merge: {resultado}

### Qué se Necesita para Continuar
- [ ] {acción concreta 1}
- [ ] {acción concreta 2}

### Sprints en Espera
- Sprint {N+1}: {nombre} — BLOQUEADO por Sprint {N}
- Sprint {N+2}: {nombre} — BLOQUEADO por Sprint {N}
```

---

## Reglas Críticas

- **NUNCA** declarar un sprint terminado sin DoD firmado
- **NUNCA** aceptar placeholders como entrega válida
- **NUNCA** saltarse el ciclo QA → Review → Merge
- **SIEMPRE** actualizar `SPRINT_STATUS.md` en cada cambio de estado
- **SIEMPRE** iniciar el siguiente sprint automáticamente si el actual fue aprobado
- **SIEMPRE** reportar el estado de avance al final de cada sprint

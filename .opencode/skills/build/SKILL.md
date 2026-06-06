---
name: build
description: Skill para la fase BUILD de Plan2Ship. Motor de construccion por sprint que coordina subagentes especializados en ramas paralelas segun el Project Blueprint. Es invocado por sprint-control durante la fase de construccion de cada sprint. Use ONLY when implementing features from a Project Blueprint sprint.
---

# Skill: Build

Este skill es el **motor de construccion** de la fase BUILD dentro del ciclo Plan2Ship.

> **Posicion en la arquitectura**: Este skill es invocado POR `sprint-control` en la fase de construccion de cada sprint. El punto de entrada del ciclo completo es `sprint-control` (activado por `/plan2ship-build`). Este skill es responsable exclusivamente de coordinar la implementacion del codigo en ramas feature/*, no del ciclo completo.

## Prerequisitos

Antes de ejecutar este skill, verifica:

1. **Existe un Project Blueprint**: Busca archivos como `Project_Blueprint.md`, `blueprint.md`, o similares en la raiz del proyecto. Si no existe, DETENTE y reporta a `sprint-control`.
2. **Existe un repositorio Git inicializado**: Si no, inicializar con `git init` y crear el commit inicial.
3. **Existe una rama principal**: Verificar que exista `main` o `master`.
4. **Sprint actual identificado**: `sprint-control` debe haber indicado que sprint se esta construyendo.

## Flujo de Build

### Paso 1: Analizar el Sprint del Blueprint

Leer el Project Blueprint y extraer para el sprint actual:

- Modulos a implementar (catalogo, usuarios, carrito, etc.)
- Dependencias entre modulos dentro del sprint
- Stack tecnologico definido
- Criterios de aceptacion del sprint (para guiar la implementacion)
- Contratos de API a implementar

### Paso 2: Planificar Ramas

Para cada modulo del sprint actual, definir la rama correspondiente:

```
feature/database-schema          <- Database Agent
feature/backend-{modulo}         <- Backend Agent (por modulo)
feature/frontend-{modulo}        <- Frontend Agent (por modulo)
feature/tests-{modulo}           <- QA Agent (por modulo)
feature/devops-setup             <- DevOps Agent
```

### Paso 3: Ejecutar en Orden de Dependencia

El orden obligatorio dentro de cada sprint es:

```
1. Database Agent    -> feature/database-schema    (sin dependencias)
2. Backend Agent     -> feature/backend-{modulo}   (depende de DB)
3. Frontend Agent    -> feature/frontend-{modulo}  (depende de Backend)
4. DevOps Agent      -> feature/devops-setup       (paralelo con Backend/Frontend)
5. QA Agent          -> feature/tests-{modulo}     (despues de cada modulo)
```

### Paso 4: Instruir a Cada Subagente

Al invocar un subagente, proporcionar:

1. **Contexto del Blueprint**: Las secciones relevantes para su tarea
2. **Rama asignada**: La rama feature/* correspondiente
3. **Scope claro**: Exactamente que debe implementar en este sprint
4. **Criterios de aceptacion**: Los que aplican a su tarea
5. **Prohibicion de placeholders**: Es obligatorio recordar a cada subagente

Ejemplo de instruccion al Frontend Agent:

```
Implementa la pagina /catalogo segun el Sprint 02 del Project Blueprint.
Trabaja en la rama feature/frontend-catalog.
OBLIGATORIO: consultar v_active_products de Supabase para mostrar datos reales.
PROHIBIDO: entregar placeholders como "Los productos se mostraran aqui".
Si no hay datos en BD, mostrar estado vacio real con UI adecuada.
Criterios de aceptacion del sprint aplicables: [lista del Blueprint].
```

Ejemplo de instruccion al Backend Agent:

```
Implementa los endpoints del Sprint 04 del Project Blueprint.
Trabaja en la rama feature/backend-admin-products.
OBLIGATORIO: implementar GET, POST, PUT, DELETE /api/v1/admin/products
PROHIBIDO: retornar arrays de datos ficticios hardcoded en los endpoints.
Los datos deben venir de la base de datos.
Criterios de aceptacion del sprint aplicables: [lista del Blueprint].
```

### Paso 5: Verificacion Post-Implementacion (Gate Anti-Placeholder)

Antes de reportar a `sprint-control` que el BUILD esta completo, ejecutar:

```bash
# Detectar placeholders en frontend
grep -rn "se mostrar\|Aqui iran\|TODO:\|placeholder" \
  --include="*.tsx" --include="*.jsx" web/src/app/

# Verificar integracion real con BD/API
grep -rn "supabase\.from\|fetch(\|axios\." \
  --include="*.tsx" web/src/app/

# Detectar datos hardcoded en backend
grep -rn "\[{.*}\]\|\"ejemplo\"\|\"ficticio\"" \
  --include="*.py" api/app/routers/

# Verificar que existen los endpoints del Blueprint
grep -rn "router\.\(get\|post\|put\|delete\|patch\)" \
  --include="*.py" api/app/routers/
```

Si cualquier check falla → reportar falla a `sprint-control`, identificar el subagente responsable, enviar correcciones.

### Paso 6: Reporte a sprint-control

Al completar la fase BUILD exitosamente, reportar:

```markdown
## Reporte BUILD — Sprint {N}: {nombre}

### Subagentes ejecutados
- database-agent: rama feature/database-schema — COMPLETADO / FALLO
- backend-agent: rama feature/backend-{modulo} — COMPLETADO / FALLO
- frontend-agent: rama feature/frontend-{modulo} — COMPLETADO / FALLO
- qa-agent: rama feature/tests-{modulo} — COMPLETADO / FALLO
- devops-agent: rama feature/devops-setup — COMPLETADO / FALLO

### Gate Anti-Placeholder
- Placeholders detectados: NINGUNO / [lista]
- Integracion real verificada: SI / NO [archivos sin integracion]
- Estado: APROBADO / BLOQUEADO

### Ramas creadas
- feature/database-schema
- feature/backend-{modulo}
- feature/frontend-{modulo}

### Listo para siguiente fase
BUILD completado. Listo para QA -> REVIEW -> MERGE.
```

## Reglas Criticas

- **NUNCA** hagas merge sin aprobacion del Merge Supervisor (eso lo gestiona `sprint-control`)
- **NUNCA** modifiques el Blueprint durante la implementacion
- **NUNCA** trabajes en la rama principal directamente
- **NUNCA** declares BUILD completo si el gate anti-placeholder falla
- **NUNCA** aceptes placeholders o datos hardcoded como entrega valida de un subagente
- **SIEMPRE** sigue el orden de dependencia: DB -> Backend -> Frontend
- **SIEMPRE** verifica la integracion real con BD/API antes de reportar BUILD listo
- **SIEMPRE** reporta el resultado a `sprint-control` al terminar (exitoso o con bloqueo)
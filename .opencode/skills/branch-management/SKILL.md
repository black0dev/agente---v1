---
name: branch-management
description: Skill para gestion de ramas paralelas en Plan2Ship. Crea, cambia y coordina ramas feature/* para cada subagente evitando conflictos. Use ONLY when creating or managing Git branches for Plan2Ship implementation workflow.
---

# Skill: Branch Management

Este skill guía la creacion y gestion de ramas paralelas para el flujo de trabajo de Plan2Ship.

## Convencion de Ramas

### Rama Principal
```
main (o master)
```
Rama estable. Solo se integra codigo revisado y aprobado.

### Ramas de Feature (por subagente)

| Subagente | Patron de Rama | Ejemplo |
|-----------|---------------|---------|
| Database Agent | `feature/database-schema` | `feature/database-schema` |
| Backend Agent | `feature/backend-{modulo}` | `feature/backend-catalog` |
| Frontend Agent | `feature/frontend-{modulo}` | `feature/frontend-catalog` |
| QA Agent | `feature/tests-{modulo}` | `feature/tests-catalog` |
| DevOps Agent | `feature/devops-setup` | `feature/devops-setup` |

## Flujo de Creacion de Ramas

### Paso 1: Asegurar que main esta actualizado

```bash
git checkout main
git pull origin main
```

### Paso 2: Crear rama feature desde main

```bash
git checkout -b feature/{tipo}-{modulo} main
```

### Paso 3: Trabajar en la rama

```bash
# Realizar cambios
git add .
git commit -m "feat({modulo}): {descripcion}"
```

### Paso 4: Mantener la rama actualizada

Antes de empezar cada sesion de trabajo o antes de un merge:

```bash
git checkout feature/{tipo}-{modulo}
git fetch origin
git rebase origin/main
```

Si hay conflictos durante el rebase:

```bash
# Resolver conflictos en los archivos marcados
git add <archivos resueltos>
git rebase --continue
```

## Reglas de Commits

Formato de mensajes:

```
feat({modulo}): agregar endpoint GET /api/products
feat({modulo}): crear tabla products en base de datos
feat({modulo}): implementar vista de catalogo
test({modulo}): agregar pruebas de integracion para productos
chore({modulo}): configurar Docker Compose
```

Convencion:
- `feat`: Nueva funcionalidad
- `fix`: Correccion de bug
- `test`: Pruebas
- `chore`: Configuracion, dependencias
- `docs`: Documentacion

## Integracion de Ramas

El merge SIEMPRE sigue el flujo supervisado:

```
1. Subagente implementa en su rama
2. QA Agent prueba la rama
3. Reviewer Agent audita contra el Blueprint
4. Merge Supervisor decide aprobar/rechazar
5. Si aprueba, merge --no-ff a main
6. Eliminar rama feature
```

### Merge Ejecucion (solo Merge Supervisor):

```bash
git checkout main
git merge --no-ff feature/{tipo}-{modulo} -m "Merge feature/{tipo}-{modulo}: {descripcion}"
git push origin main
git branch -d feature/{tipo}-{modulo}
git push origin --delete feature/{tipo}-{modulo}
```

## Orden de Integracion

Las ramas se integran en este orden:

```
1. feature/database-schema      (no tiene dependencias)
2. feature/devops-setup          (no tiene dependencias)
3. feature/backend-{modulo}      (depende de database-schema)
4. feature/frontend-{modulo}     (depende de backend-{modulo})
5. feature/tests-{modulo}        (puede integrarse despues de su modulo)
```

## Resolucion de Conflictos

Si hay conflictos entre ramas:

1. **Rebase sobre main** antes de intentar merge:
   ```bash
   git checkout feature/{tipo}-{modulo}
   git rebase origin/main
   ```

2. **Resolver conflictos manualmente**:
   - Priorizar la version de main
   - Fusionar cambios de la feature branch
   - No eliminar cambios de other branch sin justificacion

3. **Si el conflicto es entre features paralelas**:
   - El Implementation Super Agent decide la resolucion
   - Se puede pedir a los subagentes que ajusten su codigo

## Reglas Criticas

- **NUNCA** trabajes directamente en main
- **NUNCA** hagas force push a ramas ya integradas
- **SIEMPRE** crea ramas desde main actualizado
- **SIEMPRE** rebasea antes de solicitar merge
- **SIEMPRE** usa `--no-ff` para preservar historia
- **SIEMPRE** elimina ramas despues del merge
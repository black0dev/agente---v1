---
name: merge
description: Skill para el ciclo MERGE de Plan2Ship. Evalua si una rama puede integrarse a la rama principal basandose en reportes de QA y Review. Use ONLY when deciding whether to merge a feature branch.
---

# Skill: Merge

Este skill guía el proceso de merge supervisado dentro del ciclo Plan2Ship. **Queda estrictamente prohibido el merge automatico.**

## Prerequisitos

Antes de evaluar un merge, verifica:

1. **Reporte de QA disponible**: El QA Agent ya probo la rama
2. **Reporte de Review disponible**: El Reviewer Agent ya audito el codigo
3. **La rama feature/* existe y tiene commits**
4. **La rama principal (main/master) existe**

## Flujo de Merge Supervisado

### Paso 1: Recopilar Evidencia

Obtiene los tres insumos obligatorios:

1. **Reporte de QA**: Resultados de pruebas y cobertura de criterios
2. **Reporte de Review**: Resultados de la auditoria contra el Blueprint
3. **Estado de Git**: Conflictos, diferencia con main, commits en la rama

```bash
git log main..{rama} --oneline
git diff main...{rama} --stat
git merge-tree $(git merge-base main {rama}) main {rama}
```

### Paso 2: Evaluar Criterios de Aprobacion

Para aprobar el merge, TODOS estos criterios deben cumplirse:

| Criterio | Requisito | Estado |
|----------|-----------|--------|
| Reporte QA | Todas las pruebas pasando | ? |
| Criterios de Aceptacion | 100% cubiertos | ? |
| Reporte Review | APROBADO o APROBADO CON OBSERVACIONES | ? |
| Conflictos | Sin conflictos con main | ? |
| Rama actualizada | Rebaseada con main | ? |

### Paso 3: Tipos de Decision

#### APROBAR
- Todas las pruebas pasan
- Reviewuestra APROBADO
- Sin conflictos
- Criterios de aceptacion 100% cubiertos

#### APROBAR CON CAMBIOS MENORES
- Pruebas pasan (con一些 no criticas fallidas menores)
- Reviewuestra APROBADO CON OBSERVACIONES menores
- Cambios menores que no afectan la funcionalidad

#### RECHAZAR
- Pruebas criticas fallidas
- Reviewuestra RECHAZADO
- Desviaciones significativas del Blueprint
- Conflictos sin resolver

#### SOLICITAR CAMBIOS
- Pruebas fallidas pero recuperables
- Observaciones de Review que requieren correccion
- Criterios de aceptacion parcialmente cubiertos

### Paso 4: Ejecutar Merge (solo si APROBADO)

```bash
git checkout main
git pull origin main
git merge --no-ff {rama} -m "Merge {rama}: [descripcion del modulo]"
git push origin main
git branch -d {rama}
```

Si es APROBAR CON CAMBIOS MENORES, solicitar los cambios primero.

### Paso 5: Reporte de Merge

```markdown
## Decision de Merge - Rama: {nombre_rama}

### Reporte de QA
- Estado: APROBADO / RECHAZADO
- Pruebas: X pasaron, Y fallaron
- Criterios cubiertos: A/B

### Reporte de Review
- Estado: APROBADO / RECHAZADO / APROBADO CON OBSERVACIONES
- Cumplimiento del Blueprint: X%
- Desviaciones: [lista]

### Conflictos con Main
- SIN CONFLICTOS / X conflictos en [archivos]

### Decision Final: APROBAR / RECHAZAR / SOLICITAR CAMBIOS

### Justificacion
{detalles de la decision}

### Acciones Posteriores (si aplica)
- [ ] Merge ejecutado a main
- [ ] Rama eliminada
- [ ] Notificacion al Implementation Super Agent
```

## Reglas Criticas

- **NUNCA** hagas merge sin reportes de QA y Review
- **NUNCA** hagas merge si hay pruebas criticas fallidas
- **NUNCA** hagas merge si el Reviewer rechazo
- **SIEMPRE** usa `--no-ff` para preservar historia de la rama
- **SIEMPRE** elimina la rama feature despues del merge exitoso
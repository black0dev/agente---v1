---
name: review
description: Skill para el ciclo REVIEW de Plan2Ship. Audita codigo de una rama contra el Project Blueprint para verificar cumplimiento, arquitectura, pruebas y criterios de aceptacion. Use ONLY when reviewing code against a Project Blueprint.
---

# Skill: Review

Este skill guía el proceso de revision (REVIEW) dentro del ciclo Plan2Ship.

## Prerequisitos

Antes de ejecutar este skill, verifica:

1. **Existe un Project Blueprint**: Con toda la documentacion tecnica
2. **Existe una rama feature/* con codigo para revisar**
3. **Las pruebas ya fueron ejecutadas** (reporte de QA disponible)

## Flujo de Review

### Paso 1: Contexto Completo

Lee completamente el Project Blueprint y la rama a revisar:

1. Blueprint: Arquitectura, contratos API, diseno DB, interfaces, criterios
2. Rama: Todos los archivos modificados/agregados
3. Reporte de QA: Resultados de pruebas

### Paso 2: Checklist de Revision

Ejecuta el checklist completo:

#### 2.1 Cumplimiento del Blueprint

Para cada seccion del Blueprint, verifica:

```markdown
| Seccion Blueprint      | Implementado | Coincide | Observaciones |
|------------------------|--------------|----------|---------------|
| Contratos de API       | SI/NO        | SI/NO    | ...           |
| Diseno de BD           | SI/NO        | SI/NO    | ...           |
| Interfaces             | SI/NO        | SI/NO    | ...           |
| Reglas de negocio      | SI/NO        | SI/NO    | ...           |
| Validaciones           | SI/NO        | SI/NO    | ...           |
```

#### 2.2 Arquitectura

- [ ] Estructura de archivos sigue el patron definido
- [ ] Separacion de responsabilidades correcta
- [ ] Patrones de diseno correctos (MVC, Clean, etc.)
- [ ] No hay violaciones de capas

#### 2.3 Pruebas (segun reporte de QA)

- [ ] Los criterios de aceptacion tienen pruebas
- [ ] Las pruebas pasan
- [ ] Hay cobertura de errores y casos borde

#### 2.4 Criterios de Aceptacion

Para cada criterio:

```markdown
| CA ID  | Criterio                           | Cumplido | Evidencia              |
|--------|------------------------------------|----------|------------------------|
| CA-001 | El usuario puede ver productos    | SI/NO    | test_list_products    |
| CA-002 | El admin puede crear productos     | SI/NO    | test_create_product    |
```

#### 2.5 Errores Evidentes

- [ ] Sin errores de sintaxis/compilacion
- [ ] Sin vulnerabilidades de seguridad evidentes
- [ ] Sin datos sensibles expuestos
- [ ] Sin dependencias innecesarias

### Paso 3: Reporte de Revision

Genera un reporte completo:

```markdown
## Reporte de Revision - Rama: {nombre_rama}

### Veredicto: APROBADO / RECHAZADO / APROBADO CON OBSERVACIONES

### Cumplimiento del Blueprint: X%
- Desviaciones encontradas: [lista]
- Funcionalidad faltante: [lista]
- Funcionalidad extra no especificada: [lista]

### Arquitectura: CONFORME / NO CONFORME
- Violaciones: [lista]

### Pruebas: APROBADAS / FALLIDAS
- Criterios cubiertos: X/Y
- Pruebas fallidas: [lista]

### Criterios de Aceptacion: X/Y cumplidos
- No cumplidos: [lista]

### Errores Evidentes: NINGUNO / [lista]

### Recomendacion Final
[APROBAR para merge / RECHAZAR con motivos / APROBAR con cambios menores]
```

## Reglas Criticas

- **NUNCA** apruebes codigo que no cumpla el Blueprint
- **NUNCA** modifiques codigo durante la revision (solo lectura)
- **SIEMPRE** cita la seccion del Blueprint que se esta verificando
- Las desviaciones deben reportarse explicitamente, no justificarse
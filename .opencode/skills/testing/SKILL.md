---
name: testing
description: Skill para el ciclo de TESTING de Plan2Ship. Crea y ejecuta pruebas basadas en los criterios de aceptacion del Project Blueprint. Use ONLY when creating or running tests against Blueprint acceptance criteria.
---

# Skill: Testing

Este skill guía el proceso de pruebas (QA) dentro del ciclo Plan2Ship.

## Prerequisitos

Antes de ejecutar este skill, verifica:

1. **Existe un Project Blueprint**: Con criterios de aceptacion definidos
2. **Existe codigo para probar**: La rama feature/* tiene implementacion
3. **El entorno de testing esta configurado**: Framework de testing instalado

## Flujo de Testing

### Paso 1: Extraer Criterios de Aceptacion

Del Project Blueprint, extrae:

- Todos los criterios de aceptacion (CA) definidos
- Historias de usuario y sus escenarios
- Contratos de API con sus request/response esperados
- Flujos de usuario criticos
- Reglas de negocio y validaciones

### Paso 2: Plan de Pruebas

Crea un plan que mapee cada criterio de aceptacion a una o mas pruebas:

```
| CA ID  | Descripcion                    | Tipo Prueba      | Archivo                  |
|--------|--------------------------------|------------------|--------------------------|
| CA-001 | Listar productos del catalogo | Integracion API  | products.api.test.js     |
| CA-002 | Crear producto como admin     | Integracion API  | products.api.test.js     |
| CA-003 | Filtrar productos por categoria | E2E            | catalog.e2e.test.js     |
```

### Paso 3: Tipos de Pruebas

Segun el modulo, implementa:

#### Pruebas Unitarias (obligatorias)
- Logica de negocio (servicios, utils)
- Validaciones y transformaciones
- Funciones puras

#### Pruebas de Integracion (obligatorias para APIs)
- Cada endpoint del contrato de API
- Interaccion con base de datos
- Middleware y autenticacion

#### Pruebas E2E (obligatorias para flujos criticos)
- Flujos de usuario completos
- Casos de uso principales

### Paso 4: Ejecucion

1. Ejecutar las pruebas en la rama feature correspondiente
2. Registrar los resultados (pass/fail por cada prueba)
3. Mapear resultados a criterios de aceptacion

### Paso 5: Reporte

Generar reporte con el formato:

```markdown
## Reporte de Pruebas - {modulo}

### Resumen
- Total: X | Pasaron: Y | Fallaron: Z
- Cobertura de criterios: A/B

### Detalle
- [CA-001] OK - test_list_products (pass)
- [CA-002] OK - test_create_product (pass)
- [CA-003] FAIL - test_filter_by_category (fail: filtro no funciona)

### Criterios Sin Cubrir
- [CA-004] (sin prueba - requiere E2E)
```

## Reglas Criticas

- **CADA** criterio de aceptacion debe tener al menos una prueba
- Las pruebas deben probar el **comportamiento esperado**, no la implementacion
- Las pruebas fallidas deben reportarse con el **criterio de aceptacion** que fallan
- **NUNCA** se omitan criterios de aceptacion sin justificacion
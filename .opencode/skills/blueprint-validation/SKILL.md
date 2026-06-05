---
name: blueprint-validation
description: Skill para validar que una implementacion cumple con el Project Blueprint de Plan2Ship. Compara la implementacion contra la planificacion y reporta desviaciones. Use ONLY when validating implementation against Project Blueprint specifications.
---

# Skill: Blueprint Validation

Este skill permite validar que una implementacion cumple con lo estipulado en el Project Blueprint.

## Que Valida

### 1. Contratos de API

Para cada endpoint definido en el Blueprint:

```markdown
| Endpoint | Metodo | Request | Response | Status |
|----------|--------|---------|----------|--------|
| /api/products | GET | - | Product[] | OK/MISS/PARTIAL |
| /api/products | POST | CreateProductDTO | Product | OK/MISS/PARTIAL |
| /api/products/:id | GET | - | Product | OK/MISS/PARTIAL |
```

### 2. Diseno de Base de Datos

Para cada tabla definida en el Blueprint:

```markdown
| Tabla | Columnas | Tipos | Constraints | Status |
|-------|----------|-------|-------------|--------|
| products | id, name, price, category_id | UUID, VARCHAR, DECIMAL, UUID | PK, NOT NULL, FK | OK/MISS/PARTIAL |
```

### 3. Interfaces

Para cada vista/Componente definido en el Blueprint:

```markdown
| Vista | Componentes | Ruta | Status |
|-------|-------------|------|--------|
| CatalogPage | ProductList, FilterBar, SearchInput | /catalog | OK/MISS/PARTIAL |
```

### 4. Arquitectura

Para la estructura definida en el Blueprint:

```markdown
| Capa | Patron | Archivos | Status |
|------|--------|----------|--------|
| Controllers | MVC | *Controller.js | OK/MISS |
| Services | Service Layer | *Service.js | OK/MISS |
```

### 5. Criterios de Aceptacion

Para cada criterio de aceptacion:

```markdown
| CA ID | Descripcion | Implementado | Probado | Status |
|-------|-------------|-------------|---------|--------|
| CA-001 | Ver catalogo de productos | SI/NO | SI/NO | OK/FAIL |
```

## Flujo de Validacion

### Paso 1: Cargar el Blueprint

Leer el archivo `Project_Blueprint.md` (o similar) y extraer:

- Lista de endpoints con sus contratos
- Diagrama de base de datos
- Lista de interfaces y componentes
- Arquitectura del sistema
- Criterios de aceptacion

### Paso 2: Escanear la Implementacion

Buscar en el codigo:

```bash
# Buscar endpoints
grep -rn "router\.(get|post|put|delete|patch)" --include="*.js" --include="*.ts"

# Buscar modelos/tablas
grep -rn "CREATE TABLE\|createTable\|model\|schema" --include="*.js" --include="*.ts" --include="*.sql"

# Buscar componentes
glob("**/*.jsx") o glob("**/*.tsx")
glob("**/*.vue") o glob("**/*.svelte")
```

### Paso 3: Comparar

Para cada elemento del Blueprint:

1. Verificar si existe en la implementacion
2. Verificar si coincide exactamente (tipos, campos, etc.)
3. Marcar como OK, MISSING, o PARTIAL

### Paso 4: Generar Reporte

```markdown
## Reporte de Validacion contra Blueprint

### Resumen General
- Cumplimiento total: X%
- Endpoints: A/B implementados
- Tablas: C/D creadas
- Interfaces: E/F implementadas
- Criterios: G/H cumplidos

### Detalle de Desviaciones

#### Endpoints Faltantes
- POST /api/products: No implementado
- GET /api/products/:id: Parcialmente implementado (falta campo category)

#### Tablas Faltantes
- Tabla categories: No creada
- Columna products.image_url: Falta en la tabla products

#### Interfaces Faltantes
- CartPage: No implementada
- AdminPanel: Parcialmente implementada

#### Criterios de Aceptacion No Cumplidos
- [CA-005]: El usuario puede filtrar por categoria - NO CUMPLIDO

### Desviaciones Adicionales (no en Blueprint)
- Endpoint DELETE /api/products/:id (no estaba en el Blueprint)
- Tabla audit_logs (no estaba en el Blueprint)
```

## Reglas Criticas

- **NUNCA** consideres implementado algo que no coincide exactamente con el Blueprint
- Las desviaciones sospechosas (funcionalidad extra no especificada) se reportan como tales
- Las funcionalidades faltantes se reportan como MISSING
- Las funcionalidades parciales se reportan como PARTIAL con detalle de que falta
- El porcentaje de cumplimiento se calcula como: `(items OK / total items) * 100`
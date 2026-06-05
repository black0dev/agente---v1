# Plan2Ship - Instrucciones del Proyecto

## Filosofia

Plan2Ship es una arquitectura multiagente que separa la planificacion de la implementacion. La regla de oro es: **la IA no debe pasar directamente de una idea vaga al codigo**. Primero se planifica profundamente (Project Blueprint), luego se ejecuta con agentes guiados por esa planificacion.

## Flujo Obligatorio

```
PLAN -> BUILD -> REVIEW -> MERGE -> SHIP
```

1. **PLAN**: La fase de planificacion genera un Project Blueprint detallado
2. **BUILD**: Los subagentes implementan en ramas feature/* guiados por el Blueprint
3. **REVIEW**: El Reviewer Agent audita el codigo contra el Blueprint
4. **MERGE**: El Merge Supervisor aprueba o rechaza la integracion
5. **SHIP**: El DevOps Agent despliega segun la estrategia del Blueprint

## Agentes de Implementacion

| Agente | Rol | Permisos |
|--------|-----|----------|
| `implementation-super-agent` | Coordinador general | Lectura, escritura, bash |
| `backend-agent` | APIs y logica de negocio | Lectura, escritura, bash |
| `frontend-agent` | Interfaces de usuario | Lectura, escritura, bash |
| `database-agent` | Esquemas y tablas | Lectura, escritura, bash |
| `qa-agent` | Pruebas y validacion | Lectura, escritura, bash |
| `devops-agent` | Entornos y despliegue | Lectura, escritura, bash |
| `reviewer-agent` | Auditoria contra Blueprint | Solo lectura |
| `merge-supervisor-agent` | Decision de integracion | Solo lectura + git |

## Reglas de Oro

1. **No improvisar**: Todo debe estar en el Project Blueprint antes de implementarse
2. **No merge automatico**: Siempre requiere aprobacion del Merge Supervisor
3. **Trabajar en ramas**: Nunca en main/master directamente
4. **Orden de dependencias**: Database -> Backend -> Frontend
5. **Fidelidad al Blueprint**: Las desviaciones se reportan, no se implementan sin aprobacion

## Comandos Disponibles

- `/plan2ship-build` - Inicia el ciclo Build para un modulo
- `/plan2ship-review` - Audita una rama contra el Blueprint
- `/plan2ship-merge` - Evalua si una rama puede integrarse
- `/plan2ship-ship` - Despliega la aplicacion
- `/plan2ship-db` - Crea/actualiza el esquema de base de datos
- `/plan2ship-test` - Ejecuta pruebas contra criterios de aceptacion

## Convencion de Ramas

```
feature/database-schema          <- Database Agent
feature/backend-{modulo}         <- Backend Agent
feature/frontend-{modulo}         <- Frontend Agent
feature/tests-{modulo}            <- QA Agent
feature/devops-setup              <- DevOps Agent
```

## Estructura del Project Blueprint

El Project Blueprint debe contener:

- Vision general del proyecto
- Alcance
- Requisitos funcionales y no funcionales
- Historias de usuario
- Arquitectura del sistema
- Diseno de base de datos
- Contratos de API
- Interfaces principales
- Flujos de usuario
- Plan de sprints
- Criterios de aceptacion
- Riesgos y decisiones importantes

## Convencion de Commits

```
feat({modulo}): descripcion
fix({modulo}): descripcion
test({modulo}): descripcion
chore({modulo}): descripcion
docs({modulo}): descripcion
```
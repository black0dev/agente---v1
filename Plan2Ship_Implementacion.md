# Plan2Ship - Arquitectura de Implementacion

## Vision General

La Arquitectura de Implementacion es la segunda fase del sistema Plan2Ship. Toma el **Project Blueprint** (generado por la fase de Planificacion) y lo convierte en software funcional y desplegable, evitando la improvisacion y asegurando el control de calidad en cada paso.

## Flujo Principal

```
BUILD -> REVIEW -> MERGE -> SHIP
```

Cada fase tiene un flujo estricto que no puede omitirse.

## Arquitectura de Agentes

```
Implementation Super Agent (coordinador)
├── Backend Agent        -> feature/backend-{modulo}
├── Frontend Agent       -> feature/frontend-{modulo}
├── Database Agent       -> feature/database-schema
├── QA Agent             -> feature/tests-{modulo}
├── DevOps Agent         -> feature/devops-setup
├── Reviewer Agent       -> (solo lectura, auditoria)
└── Merge Supervisor Agent -> (solo lectura + git, decision)
```

### Implementation Super Agent

- **Rol**: Coordinador general del ciclo completo
- **Permisos**: Completos (lectura, escritura, bash)
- **Responsabilidades**:
  - Leer el Project Blueprint completo antes de actuar
  - Dividir el trabajo entre subagentes
  - Orquestar el flujo Build -> Review -> Merge -> Ship
  - Reportar desviaciones del Blueprint

### Backend Agent

- **Rol**: Desarrollo de APIs y logica de negocio
- **Permisos**: Lectura, escritura, bash
- **Responsabilidades**:
  - Implementar endpoints segun contratos de API del Blueprint
  - Crear modelos, servicios y controladores
  - Seguir la arquitectura definida

### Frontend Agent

- **Rol**: Desarrollo de interfaces de usuario
- **Permisos**: Lectura, escritura, bash
- **Responsabilidades**:
  - Implementar vistas y componentes segun el Blueprint
  - Integrar con APIs del backend
  - Seguir los flujos de usuario definidos

### Database Agent

- **Rol**: Creacion de esquemas y tablas
- **Permisos**: Lectura, escritura, bash
- **Responsabilidades**:
  - Crear migraciones segun el diseno de BD del Blueprint
  - Definir relaciones, constraints e indices
  - Crear seeds si el Blueprint los especifica

### QA Agent

- **Rol**: Creacion y ejecucion de pruebas
- **Permisos**: Lectura, escritura, bash
- **Responsabilidades**:
  - Mapear criterios de aceptacion a pruebas
  - Crear pruebas unitarias, de integracion y E2E
  - Reportar resultados y cobertura

### DevOps Agent

- **Rol**: Configuracion de entornos y despliegue
- **Permisos**: Lectura, escritura, bash
- **Responsabilidades**:
  - Configurar Docker y docker-compose
  - Configurar CI/CD
  - Preparar scripts de despliegue

### Reviewer Agent

- **Rol**: Auditoria contra el Project Blueprint
- **Permisos**: Solo lectura + git (no puede editar codigo)
- **Responsabilidades**:
  - Verificar cumplimiento del Blueprint
  - Verificar arquitectura
  - Verificar pruebas y criterios de aceptacion
  - Generar reporte de revision

### Merge Supervisor Agent

- **Rol**: Decision final de integracion
- **Permisos**: Solo lectura + git merge/push
- **Responsabilidades**:
  - Evaluar reportes de QA y Review
  - Verificar conflictos
  - Aprobar o rechazar merges
  - Ejecutar merges aprobados

## Skills Disponibles

| Skill | Descripcion |
|-------|-------------|
| `build` | Orquesta la implementacion de modulos del Blueprint |
| `testing` | Crea y ejecuta pruebas contra criterios de aceptacion |
| `review` | Audita codigo contra el Project Blueprint |
| `merge` | Evalua y ejecuta merges supervisados |
| `deploy` | Configura y ejecuta despliegue |
| `branch-management` | Gestiona ramas feature/* para cada subagente |
| `blueprint-validation` | Valida implementacion contra el Blueprint |

## Comandos

| Comando | Descripcion |
|---------|-------------|
| `/plan2ship-build` | Inicia ciclo Build |
| `/plan2ship-review` | Inicia ciclo Review |
| `/plan2ship-merge` | Inicia ciclo Merge |
| `/plan2ship-ship` | Inicia ciclo Ship |
| `/plan2ship-db` | Crea/actualiza esquema de BD |
| `/plan2ship-test` | Ejecuta pruebas |

## Convencion de Ramas

```
main (o master)                          <- Rama estable
├── feature/database-schema               <- Database Agent
├── feature/backend-catalog               <- Backend Agent (catalogo)
├── feature/backend-users                 <- Backend Agent (usuarios)
├── feature/frontend-catalog              <- Frontend Agent (catalogo)
├── feature/frontend-users                <- Frontend Agent (usuarios)
├── feature/tests-catalog                 <- QA Agent (catalogo)
├── feature/tests-users                   <- QA Agent (usuarios)
└── feature/devops-setup                  <- DevOps Agent
```

## Orden de Integracion

```
1. feature/database-schema    (sin dependencias)
2. feature/devops-setup       (sin dependencias)
3. feature/backend-{modulo}   (depende de database)
4. feature/frontend-{modulo}  (depende de backend)
5. feature/tests-{modulo}     (despues de cada modulo)
```

## Flujo de Merge Supervisado

```
Agente implementa en su rama
        ↓
QA Agent prueba la rama
        ↓
Reviewer Agent audita contra Blueprint
        ↓
Merge Supervisor aprueba o rechaza
        ↓
Merge controlado (--no-ff) a main
        ↓
Rama feature eliminada
```

## Archivos de Configuracion

```
C:\ope\
├── opencode.json                              <- Configuracion principal
├── AGENTS.md                                   <- Instrucciones del proyecto
├── .opencode/
│   ├── agents/
│   │   ├── implementation-super-agent.md       <- Agente coordinador
│   │   ├── backend-agent.md                    <- Agente backend
│   │   ├── frontend-agent.md                  <- Agente frontend
│   │   ├── database-agent.md                  <- Agente base de datos
│   │   ├── qa-agent.md                         <- Agente QA
│   │   ├── devops-agent.md                     <- Agente DevOps
│   │   ├── reviewer-agent.md                   <- Agente revisor
│   │   └── merge-supervisor-agent.md           <- Agente supervisor de merge
│   └── skills/
│       ├── build/SKILL.md                      <- Skill de implementacion
│       ├── testing/SKILL.md                    <- Skill de pruebas
│       ├── review/SKILL.md                     <- Skill de revision
│       ├── merge/SKILL.md                      <- Skill de merge
│       ├── deploy/SKILL.md                      <- Skill de despliegue
│       ├── branch-management/SKILL.md          <- Skill de gestion de ramas
│       └── blueprint-validation/SKILL.md        <- Skill de validacion Blueprint
```
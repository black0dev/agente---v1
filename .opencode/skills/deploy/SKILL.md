---
name: deploy
description: Skill para el ciclo SHIP (despliegue) de Plan2Ship. Configura y ejecuta el despliegue segun la estrategia del Project Blueprint. Use ONLY when deploying an application based on Blueprint deployment strategy.
---

# Skill: Deploy

Este skill guía el proceso de despliegue (SHIP) dentro del ciclo Plan2Ship.

## Prerequisitos

Antes de ejecutar este skill, verifica:

1. **Todo el codigo esta mergueado en la rama principal**: No hay ramas feature pendientes
2. **Todas las pruebas pasan**: Reporte de QA aprobado
3. **El Reviewer aprobo el codigo**: Reporte de Review aprobado
4. **Existe una estrategia de despliegue en el Blueprint**

## Estrategias de Despliegue

La estrategia se determina segun lo definido en el Project Blueprint:

### Opcion 1: Docker Compose Local (por defecto)

Para desarrollo local y demos:

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  db_data:
```

### Opcion 2: Plataforma Cloud (Render, Railway, Vercel)

Para despliegue en produccion:

1. Configurar variables de entorno en la plataforma
2. Conectar repositorio Git
3. Configurar build command y start command
4. Configurar base de datos gestionada

### Opcion 3: VPS con Docker

Para despliegue en VPS:

1. SSH al servidor
2. Clonar el repositorio
3. Configurar .env
4. `docker compose up -d`

## Flujo de Despliegue

### Paso 1: Verificar Prerequisitos

```bash
git status
git log --oneline -5
```

- Confirmar que estamos en la rama principal
- Confirmar que no hay cambios sin commitear

### Paso 2: Preparar Configuracion

Segun la estrategia del Blueprint:

1. Crear/verificar `.env.production` (usando `.env.example` como referencia)
2. Verificar `Dockerfile` existe y esta correcto
3. Verificar `docker-compose.yml` existe y esta correcto
4. Verificar scripts de despliegue

### Paso 3: Ejecutar Despliegue

#### Para Docker Compose:
```bash
docker compose down
docker compose build --no-cache
docker compose up -d
docker compose ps
```

#### Para Cloud Platform:
```bash
git push origin main
# La plataforma detecta el push y despliega automaticamente
```

### Paso 4: Verificacion Post-Despliegue

Despues del despliegue, verificar:

1. La aplicacion responde (health check)
2. La base de datos esta conectada
3. Las migraciones se ejecutaron
4. Los endpoints principales funcionan

```bash
curl http://localhost:3000/health
```

### Paso 5: Reporte de Despliegue

```markdown
## Reporte de Despliegue

### Estrategia Utilizada
[Docker Compose / Render / Vercel / VPS]

### Estado del Despliegue
- Fecha: {fecha}
- Commit: {hash}
- Plataforma: {plataforma}
- URL: {url}

### Verificaciones Post-Despliegue
- [ ] Health check: OK / FAIL
- [ ] Base de datos conectada: OK / FAIL
- [ ] Migraciones ejecutadas: OK / FAIL
- [ ] Endpoints principales: OK / FAIL

### Variables de Entorno Configuradas
- {lista de variables (sin valores)}

### Comandos de Gestion
- Iniciar: `{comando}`
- Detener: `{comando}`
- Ver logs: `{comando}`
- Reiniciar: `{comando}`
```

## Reglas Criticas

- **NUNCA** despliegues sin que todos los merges esten aprobados
- **NUNCA** incluyas secrets/contrasenas en el codigo o docker-compose
- **SIEMPRE** usa variables de entorno para configuracion sensible
- **SIEMPRE** verifica el health check despues del despliegue
- **SIEMPRE** prefer Docker Compose local como primera opcion
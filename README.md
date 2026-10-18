# Nexo-Escolar

Sistema de administración escolar construido como proyecto de aprendizaje de arquitectura hexagonal con ASP.NET Core, PostgreSQL y React. Ver [docs/DECISIONES.md](docs/DECISIONES.md) para el contexto y [docs/ROADMAP.md](docs/ROADMAP.md) para el plan por sprints.

## Stack

- **Backend:** ASP.NET Core Web API sobre .NET 10 LTS, arquitectura hexagonal, ADO.NET + Npgsql (sin ORM)
- **Base de datos:** PostgreSQL 17 (Docker Compose)
- **Frontend:** React + TypeScript + Vite
- **Pruebas:** NUnit

## Estructura

```
.
├── backend/
│   ├── NexoEscolar.slnx
│   ├── docker-compose.yml          # PostgreSQL local
│   └── src/
│       ├── School.Api/             # adaptadores driving: HTTP, DI, endpoints
│       ├── School.Application/     # casos de uso y puertos
│       ├── School.Domain/          # entidades y reglas de negocio
│       ├── School.Infrastructure/  # adaptadores: Npgsql, SQL, autenticación
│       └── Tests/School.UnitTests/
├── frontend/                       # React + TypeScript (Sprint 8)
└── docs/
    ├── DECISIONES.md               # decisiones de negocio y técnicas
    └── ROADMAP.md                  # plan de sprints 0–8
```

## Requisitos

- .NET SDK 10 (ver `backend/global.json`)
- Docker
- Bun o Node.js (para el frontend)

## Cómo levantar el proyecto

```bash
# 1. Base de datos
cd backend
docker compose up -d

# 2. API
dotnet run --project src/School.Api

# 3. Pruebas
dotnet test

# 4. Frontend (cuando exista)
cd ../frontend
bun install
bun run dev
```

La API queda disponible en el puerto indicado en `launchSettings.json`; OpenAPI en `/openapi/v1.json` (Swagger UI en desarrollo).

La cadena de conexión por defecto está en `backend/src/School.Api/appsettings.json` (`ConnectionStrings:NexoEscolarDb`) y apunta al Postgres del compose.

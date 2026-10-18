# Decisiones del proyecto

Documento de decisiones técnicas y de negocio de Nexo-Escolar. Se actualiza cuando cambie una decisión; no es un diario de cambios.

## Contexto

Nexo-Escolar nace como el siguiente paso de aprendizaje después de Warehouse API. El objetivo es practicar ASP.NET Core, ADO.NET/PostgreSQL y React aplicando arquitectura hexagonal de verdad, con un sistema escolar que sirva además de proyecto de portafolio.

| Decisión | Elección | Motivo |
| --- | --- | --- |
| Tipo de escuela | Genérico | No atarse a privada/pública. El modelo cubre alumnos, tutores, grupos, inscripciones, calificaciones y asistencia; pagos quedan fuera del MVP |
| Objetivo principal | Aprender + portafolio | Practicar las tecnologías y poder mostrar el proyecto en entrevistas |
| Alcance MVP | Alumnos, grupos e inscripciones | Módulo mínimo para construir una funcionalidad útil de extremo a extremo; el resto llega después |

## Stack

| Área | Tecnología |
| --- | --- |
| Lenguaje backend | C# |
| Framework | ASP.NET Core Web API sobre .NET 10 LTS (fijado en `backend/global.json`) |
| Arquitectura | Hexagonal (puertos y adaptadores) |
| Base de datos | PostgreSQL 17 (Docker Compose local) |
| Acceso a datos | ADO.NET con Npgsql (sin ORM, SQL explícito) |
| Frontend | React + TypeScript + Vite |
| Datos remotos | TanStack Query (Sprint 8) |
| Formularios | React Hook Form + Zod (Sprint 8) |
| Pruebas backend | NUnit (se mantuvo en lugar de xUnit; cumple el mismo objetivo) |
| Entorno local | Docker Compose |
| CI | GitHub Actions (pendiente, sprint futuro) |

## Arquitectura hexagonal: cómo la aplicamos

La regla central: **las dependencias apuntan hacia adentro**. El dominio no depende de nadie; la aplicación define los contratos; los detalles técnicos viven afuera y se conectan al final.

```
School.Api            → adaptador "driving": HTTP, DTOs de entrada/salida, composición (DI)
School.Application    → casos de uso + PUERTOS (interfaces como IStudentRepository)
School.Domain         → entidades, value objects, reglas de negocio (cero dependencias)
School.Infrastructure → adaptadores "driven": PostgreSQL con Npgsql/ADO.NET, autenticación
```

### Responsabilidades por capa

- **Domain**: entidades (`Student`), objetos de valor, invariantes y errores de negocio. No conoce `NpgsqlConnection`, `HttpContext`, JSON ni ASP.NET.
- **Application**: casos de uso (ej. `RegisterStudent`), contratos de entrada/salida y los puertos que define para operaciones externas. No sabe *cómo* se guarda, solo *qué* necesita guardar.
- **Infrastructure**: implementa los puertos de Application con ADO.NET/Npgsql, configuración de conexión, transacciones y scripts SQL versionados.
- **Api**: recibe HTTP, valida el formato de la solicitud, invoca el caso de uso y traduce el resultado (o el error) a un código HTTP. Registra los adaptadores en el contenedor de dependencias.

### Reglas prácticas

1. Tipos de Npgsql (`NpgsqlConnection`, `NpgsqlDataReader`, etc.) o SQL **nunca** cruzan hacia Application o Domain.
2. Las interfaces (puertos) se declaran en Application y se implementan en Infrastructure. Domain no define interfaces para infraestructura.
3. No se crea un `Repository<T>` genérico para ocultar consultas: cada puerto expone las operaciones que el caso de uso necesita.
4. No se agregan abstracciones "por si acaso" (MediatR, CQRS completo, una interfaz por clase). Se introduce abstracción cuando resuelve un problema real.
5. Distinguir errores: una solicitud HTTP mal formada no es lo mismo que una violación de regla de negocio.

### Ejemplo: cómo se verá el módulo Students

```
School.Domain/Students/
  Student.cs                  # entidad + invariantes (nombre obligatorio, matrícula única...)
  StudentErrors.cs            # errores de negocio del dominio

School.Application/Students/
  IStudentRepository.cs       # puerto: Add, GetById, ExistsByEnrollment...
  RegisterStudent.cs          # caso de uso (o StudentService si prefieres servicios)
  StudentDtos.cs              # contratos de entrada/salida del caso de uso

School.Infrastructure/Persistence/
  NpgsqlConnectionFactory.cs  # adaptador: obtiene conexiones
  StudentRepository.cs        # adaptador: implementa IStudentRepository con SQL

School.Api/Endpoints/
  StudentsEndpoints.cs        # adaptador driving: mapea HTTP ↔ caso de uso
```

Referencia teórica: [Hexagonal Architecture (Alistair Cockburn)](https://alistarkhurst.com/articles/ports-adapters.html)

### Regla personal

> Antes de crear una clase, debo poder explicar qué responsabilidad tiene, qué problema resuelve y por qué pertenece a esa capa.

Si no puedo explicarlo, la clase no debería existir todavía.

## Versiones y soporte

- .NET 10 LTS, fijado en `backend/global.json` con `rollForward: latestFeature` para que la imagen Docker del SDK pueda compilar aunque el patch cambie.
- Revisar el soporte de paquetes al añadir dependencias nuevas; preferir versiones LTS estables.

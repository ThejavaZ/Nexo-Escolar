# Roadmap de Nexo-Escolar

Proyecto de aprendizaje estructurado por sprints, como un proyecto profesional. No hay duración fija: algunos sprints son una tarde, otros varios días. Lo importante es cerrar los criterios de aceptación antes de avanzar.

**Regla de arquitectura hexagonal:** no se empieza creando todas las entidades ni todos los repositorios. Se define primero el dominio, los límites de la aplicación y cómo se comunican las capas. Después se construye una funcionalidad completa de extremo a extremo.

---

## Flujo de Git

- Un **rama por sprint**: `sprint/0-preparacion`, `sprint/1-dominio`, etc.
- Trabajo sobre la rama del sprint con commits pequeños y frecuentes.
- Cuando se cumplen todos los criterios de aceptación del sprint:
  1. Merge a `main`
  2. Tag: `sprint-N-done`
  3. Push de `main` y del tag
- Convención de mensajes: `feat:`, `fix:`, `docs:`, `chore:`, `test:`
- **Sin criterios cumplidos y sin merge+push, no se avanza al siguiente sprint.**

```bash
git switch -c sprint/0-preparacion
# ... trabajo y commits ...
git switch main && git merge sprint/0-preparacion
git tag sprint-0-done
git push origin main sprint-0-done
```

---

## Sprints

### Sprint 0 — Preparación arquitectónica

**Estado:** [x] Completado

**Objetivo:** que la solución compile y tenga una estructura consistente antes de programar funcionalidad escolar.

- [x] Verificar que `NexoEscolar.slnx` contiene todos los proyectos correctos
- [x] Confirmar que todos los proyectos apuntan a `net10.0`
- [x] Verificar las referencias entre proyectos
- [x] Eliminar las clases `Class1.cs` generadas por las plantillas
- [x] Revisar el código inicial generado en `School.Api`
- [x] Configurar nullable reference types y convenciones básicas
- [x] Crear un `.gitignore` adecuado para .NET y frontend
- [x] Limpiar artefactos de build (`bin/`, `obj/`) del repositorio
- [x] Documentar las responsabilidades de cada proyecto (`docs/DECISIONES.md`)
- [x] Docker: `Dockerfile` multi-stage, `docker-compose.yml` con PostgreSQL

No agregar todavía JWT, migraciones complejas ni decenas de paquetes.

**Criterio de aceptación:** la solución compila, las pruebas se ejecutan y las dependencias están correctamente configuradas.

---

### Sprint 1 — Dominio y reglas de negocio

**Estado:** [ ] Por comenzar

**Objetivo:** definir el primer modelo de negocio sin depender de ASP.NET, SQL ni frameworks externos.

Empieza con alumnos; deja docentes, pagos y calificaciones para después.

- [ ] Identificar los conceptos del dominio escolar
- [ ] Definir qué representa un alumno dentro del sistema
- [ ] Determinar qué información es obligatoria y cuál es opcional
- [ ] Identificar las invariantes y restricciones del negocio
- [ ] Diseñar las entidades y objetos de valor que realmente se necesiten
- [ ] Definir cómo se representan los errores de negocio
- [ ] Escribir pruebas unitarias para las reglas implementadas

Todo esto vive principalmente en `School.Domain`.

**Regla:** el dominio no debe conocer `NpgsqlConnection`, `HttpContext`, controladores, JSON ni servicios de ASP.NET. No diseñar entidades para todos los módulos futuros: modelo mínimo que resuelva el problema actual.

**Criterio de aceptación:** las reglas de negocio del alumno están implementadas y cubiertas por pruebas unitarias en `School.Domain`.

---

### Sprint 2 — Aplicación y puertos

**Estado:** [ ] Por comenzar

**Objetivo:** definir qué puede hacer el sistema y qué necesita del exterior para hacerlo.

En `School.Application`:

- [ ] Casos de uso
- [ ] Contratos de entrada y salida
- [ ] Puertos para operaciones externas (persistencia)
- [ ] DTO específicos de los casos de uso, cuando sean necesarios
- [ ] Coordinación y validación de las operaciones

Pensar en la operación «registrar alumno» sin implementarla todavía: ¿qué información recibe? ¿qué reglas debe cumplir? ¿qué necesita guardar? ¿qué resultado devuelve si hay conflicto?

No hace falta MediatR, CQRS completo ni una interfaz por cada clase. Empezar con clases y contratos explícitos; más abstracciones solo cuando aporten valor.

**Criterio de aceptación:** se puede describir y probar el caso de uso sin conectarse a PostgreSQL ni iniciar un servidor HTTP.

---

### Sprint 3 — Infraestructura y PostgreSQL

**Estado:** [ ] Por comenzar

**Objetivo:** implementar los detalles técnicos que la aplicación necesita.

En `School.Infrastructure`:

- [ ] Definir la configuración de PostgreSQL
- [ ] Crear el mecanismo para obtener conexiones con Npgsql
- [ ] Implementar los puertos de persistencia definidos en Application
- [ ] Usar consultas SQL parametrizadas con ADO.NET
- [ ] Gestionar correctamente conexiones, comandos y lectores
- [ ] Definir transacciones para operaciones que lo requieran
- [ ] Versionar los scripts de creación y evolución de la base de datos

Aquí entra lo aprendido en Warehouse API. Sin ORM: SQL explícito. Sin `Repository<T>` genérico que oculte todas las consultas.

**Criterio de aceptación:** los adaptadores pueden acceder a PostgreSQL y ejecutar las operaciones requeridas sin trasladar detalles de la base de datos al dominio.

---

### Sprint 4 — API y composición

**Estado:** [ ] Por comenzar

**Objetivo:** conectar el mundo HTTP con la lógica de aplicación.

En `School.Api`:

- [ ] Configurar la inyección de dependencias
- [ ] Registrar los adaptadores de infraestructura
- [ ] Exponer los endpoints necesarios para el caso de uso
- [ ] Validar el formato de las solicitudes
- [ ] Traducir resultados y errores a códigos HTTP apropiados
- [ ] Configurar OpenAPI
- [ ] Probar los endpoints con un cliente HTTP

Distinguir dos validaciones: una solicitud HTTP mal formada no es lo mismo que una operación que viola una regla de negocio.

**Criterio de aceptación:** se puede invocar el endpoint y recorrer toda la cadena hasta la persistencia.

---

### Sprint 5 — Pruebas y calidad

**Estado:** [ ] Por comenzar

Se mantiene NUnit.

- [ ] Pruebas unitarias para las reglas del dominio
- [ ] Pruebas de aplicación con dobles de prueba para los puertos
- [ ] Pruebas de integración para los adaptadores PostgreSQL
- [ ] Pruebas de integración HTTP para los endpoints críticos
- [ ] Verificar errores, casos límite y resultados inesperados

Las pruebas unitarias no deben necesitar una base de datos real. Las de integración sí pueden usar una base de prueba aislada.

**Criterio de aceptación:** se puede demostrar que las reglas y el flujo principal funcionan, y detectar regresiones automáticamente.

---

### Sprint 6 — Primer módulo escolar completo

**Estado:** [ ] Por comenzar

El sprint que convierte la arquitectura en una funcionalidad útil.

- [ ] Registrar alumnos
- [ ] Consultar el detalle de un alumno
- [ ] Listar alumnos con paginación
- [ ] Filtrar y buscar alumnos
- [ ] Actualizar sus datos
- [ ] Definir cómo se gestionan las bajas, si el negocio las necesita
- [ ] Documentar los endpoints y sus respuestas

Reutilizar lo aprendido en Warehouse sobre filtros y consultas paginadas, adaptándolo a las necesidades escolares.

**Criterio de aceptación:** el módulo de alumnos funciona de extremo a extremo y está documentado.

---

### Sprint 7 — Seguridad y auditoría

**Estado:** [ ] Por comenzar

- [ ] Implementar autenticación
- [ ] Definir roles y políticas de autorización
- [ ] Proteger los endpoints en el servidor
- [ ] Registrar acciones relevantes en una bitácora
- [ ] No almacenar contraseñas en texto plano
- [ ] Evitar exponer datos personales innecesarios

No construir todos los permisos antes de tener claros los usuarios y los módulos del sistema.

**Criterio de aceptación:** los endpoints están protegidos y las acciones relevantes quedan registradas.

---

### Sprint 8 — Frontend administrativo

**Estado:** [ ] Por comenzar

El frontend consume la API existente; no duplica sus reglas de negocio.

- [ ] Inicializar React, TypeScript y Vite (ya hecho a nivel de scaffold)
- [ ] Configurar navegación y estructura visual
- [ ] Crear el cliente HTTP
- [ ] Implementar las pantallas de alumnos
- [ ] Gestionar estados de carga, errores y formularios
- [ ] Integrar autenticación y permisos cuando el backend esté preparado
- [ ] Verificar que las operaciones funcionan desde la interfaz

**Criterio de aceptación:** se puede administrar el módulo de alumnos desde el navegador contra la API real.

---

## Regla de avance

> Antes de crear una clase, debo poder explicar qué responsabilidad tiene, qué problema resuelve y por qué pertenece a esa capa.

Así se aprende arquitectura hexagonal de verdad, en lugar de distribuir archivos entre cuatro carpetas.

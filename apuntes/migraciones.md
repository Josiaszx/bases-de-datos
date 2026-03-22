# Migraciones de Bases de Datos

Las migraciones de bases de datos son el proceso de gestionar cambios en el esquema o los datos de una base de datos de forma controlada y reproducible. Aquí te explico los conceptos clave:

## ¿Qué son?

Una migración es un script o archivo que describe una transformación en la base de datos: crear una tabla, agregar una columna, cambiar un tipo de dato, poblar datos iniciales, etc. El objetivo es que estos cambios sean **versionados, rastreables y reversibles**.

## ¿Por qué son importantes?

Sin migraciones, los cambios en la base de datos se hacen manualmente en cada entorno (desarrollo, staging, producción), lo que genera inconsistencias, errores humanos y dificultad para trabajar en equipo. Las migraciones resuelven esto porque:

- Todos los desarrolladores aplican exactamente los mismos cambios
- Se puede reproducir el estado de la base de datos en cualquier punto del historial
- Los cambios se integran al flujo de control de versiones (Git)
- Permiten rollback si algo sale mal

## Conceptos fundamentales

**Up / Down (o migrate / rollback):** Cada migración típicamente tiene dos operaciones: `up` para aplicar el cambio y `down` para revertirlo.

**Historial de migraciones:** Las herramientas guardan un registro de qué migraciones ya fueron aplicadas (generalmente en una tabla especial en la propia base de datos, como `schema_migrations`).

**Orden de ejecución:** Las migraciones se ejecutan en orden cronológico o numérico, garantizando consistencia.

## Herramientas populares

Depende mucho del ecosistema:

- **Flyway** y **Liquibase** → agnósticas al lenguaje, muy usadas en Java
- **Alembic** → Python / SQLAlchemy
- **Django Migrations** → integrado en Django
- **ActiveRecord Migrations** → Ruby on Rails
- **Prisma Migrate** → Node.js
- **Knex.js** → Node.js
- **Entity Framework Migrations** → .NET

## Buenas prácticas

- Cada migración debe ser **atómica**: hace una sola cosa bien definida
- Nunca editar una migración ya aplicada en producción; si hay un error, se crea una nueva migración que lo corrige
- Siempre probar el `rollback` antes de aplicar en producción
- Incluir las migraciones en el repositorio de código junto con la aplicación
- En equipos grandes, coordinar para evitar conflictos entre migraciones simultáneas

## Migraciones de datos y de esquema

Es importante distinguir:

- **Migraciones de esquema:** cambian la *estructura* (tablas, columnas, índices, restricciones)
- **Migraciones de datos:** transforman los *datos existentes* (normalizar valores, rellenar nuevas columnas, limpiar registros) o combiar de 

Las migraciones de datos suelen ser más delicadas en producción porque pueden tardar mucho tiempo en tablas grandes y bloquear operaciones.

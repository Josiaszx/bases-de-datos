## Indexación de Bases de Datos

La indexación es una técnica utilizada para optimizar el rendimiento de una base de datos.

### ¿Qué es un índice?

Un índice es una **estructura de datos auxiliar** que se crea sobre una o más columnas de una tabla, con el objetivo de acelerar las consultas. Es conceptualmente similar al índice de un libro: en lugar de leer página por página para encontrar un tema, vas directo a la referencia.

Sin índice, una base de datos hace un **full table scan** — recorre cada fila una por una. Con un índice, puede saltar directamente a los registros relevantes.

### ¿Para qué sirven?

- Acelerar consultas (`SELECT`, `WHERE`, `JOIN`, `ORDER BY`)
- Garantizar unicidad (`UNIQUE INDEX`)
- Optimizar ordenamientos y agrupaciones
- Mejorar el rendimiento en tablas con millones de filas


### ¿Cómo funcionan internamente?

La estructura más común es el **B-Tree (árbol balanceado)**:
- Los datos se organizan en nodos jerárquicos ordenados
- Cada búsqueda tarda **O(log n)** en vez de O(n)
- Es eficiente para rangos (`BETWEEN`, `>`, `<`) y búsquedas exactas

### Tipos de índices

**B-Tree:** el más común. Funciona para igualdad (`=`), rangos (`BETWEEN`, `>`) y ordenamiento. Es el índice por defecto en PostgreSQL, MySQL, etc.

**Hash:** ultra rápido para búsquedas exactas (`=`), pero inútil para rangos. Cada clave se mapea a una posición fija mediante una función hash.

**Índice compuesto:** sobre múltiples columnas, como `(apellido, nombre)`. Es muy eficiente cuando las consultas filtran por ambas columnas, pero el orden importa.

**Índice de texto completo (Full-text):** para búsquedas en texto libre, como en motores de búsqueda. Tokeniza y almacena lemas de palabras.

**Índice parcial:** cubre solo un subconjunto de filas (ej: `WHERE activo = true`). Ocupa menos espacio y es más rápido.


### Ventajas y desventajas

| | Sin índice | Con índice |
|---|---|---|
| Lectura (`SELECT`) | Lento en tablas grandes | Muy rápido |
| Escritura (`INSERT/UPDATE`) | Normal | Un poco más lento (hay que actualizar el índice tambien) |
| Espacio en disco | Menos | Más (la estructura ocupa espacio) |

### ¿Cuándo crear índices?

Conviene indexar columnas que aparecen frecuentemente en:
- cláusulas `WHERE`
- `JOIN ON`
- `ORDER BY` / `GROUP BY`
- Columnas con alta cardinalidad (muchos valores únicos)

No conviene indexar columnas muy pocas veces consultadas, columnas con muy pocos valores distintos (ej: `booleano`), o tablas pequeñas donde el full scan ya es rápido.
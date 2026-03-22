# Arquitectura y Funcionamiento Interno de Bases de Datos

## Arquitectura en capas

Una base de datos relacional clásica (como PostgreSQL o MySQL) tiene una arquitectura en capas bastante bien definida:

### 1. Capa de interfaz / parser
Recibe la consulta SQL como texto y la convierte en una estructura interna (un árbol de sintaxis abstracta, AST). Verifica que la sintaxis sea válida.

### 2. Query Optimizer
El optimizador toma el AST y decide **cómo** ejecutar la consulta. Para `SELECT * FROM A JOIN B ON ...`, hay múltiples formas de hacerlo: ¿empiezo por A o por B? ¿uso un índice? ¿hago un hash join o un nested loop?

El optimizador construye un **plan de ejecución** basado en estadísticas sobre los datos (cuántas filas tiene cada tabla, qué tan selectivo es cada índice, etc.) y elige el plan de menor costo estimado.

### 3. Execution Engine
Ejecuta el plan generado por el optimizador. Opera sobre **operadores relacionales**: scan, filter, join, sort, aggregate. Estos operadores se componen como un árbol donde los datos "fluyen" de abajo hacia arriba.

### 4. Storage Engine
Gestiona cómo los datos se guardan y recuperan del disco. Aquí viven estructuras como los **B-Trees** (para índices), las **heap files** (para los datos en sí), y los **buffer pools**.

### 5. Transaction Manager + Lock Manager
Garantiza las propiedades **ACID**. Controla qué transacciones pueden ejecutarse en paralelo y cuándo deben esperar o abortarse.

### 6. Write-Ahead Log (WAL)
Antes de modificar cualquier dato en disco, se escribe primero en un log secuencial. Esto permite recuperarse de fallos: si el sistema cae, el log permite reconstruir el estado consistente.

## Cómo se almacenan los datos físicamente

Los datos no se guardan "fila por fila" de forma ingenua. Todo se organiza en **páginas** (típicamente 8KB o 16KB). El disco se divide en páginas, y el motor trabaja siempre en términos de páginas, no de bytes individuales.

El **Buffer Pool** es una zona en RAM que cachea páginas del disco. Leer del disco es ~100.000x más lento que leer de RAM, por lo que el buffer pool es crítico para el rendimiento. Usa algoritmos como **LRU** (Least Recently Used) para decidir qué páginas mantener en memoria.

### Índices: B-Trees

El índice más común es el **B-Tree** (árbol B balanceado). Es una estructura de árbol donde:
- Los nodos internos guardan claves y punteros a hijos
- Las hojas guardan los datos (o punteros a ellos)
- El árbol siempre está balanceado: todas las hojas están a la misma profundidad

Esto garantiza búsquedas, inserciones y eliminaciones en **O(log n)**. Para una tabla con 1 millón de filas, un B-Tree necesita solo aproximadamente 20 comparaciones para encontrar cualquier registro.

## Transacciones y ACID

Las propiedades ACID son garantías formales:

- **Atomicidad**: una transacción es todo o nada. Si falla a la mitad, se deshace completamente (rollback).
- **Consistencia**: la base de datos pasa de un estado válido a otro estado válido. Las restricciones (constraints, foreign keys) nunca se violan.
- **Aislamiento**: las transacciones concurrentes no se "ven" entre sí en estados intermedios. Esto tiene niveles: Read Committed, Repeatable Read, Serializable.
- **Durabilidad**: una vez confirmada (commit), la transacción sobrevive a cualquier fallo del sistema.

El aislamiento es el más costoso de implementar. Los mecanismos principales son dos: **locking** (bloquear filas/tablas) y **MVCC** (Multi-Version Concurrency Control), donde cada transacción ve una "snapshot" del momento en que empezó.

## Modelos de datos alternativos

El modelo relacional no es el único. Cada modelo tiene una teoría distinta detrás:

| Modelo | Idea central | Ejemplo |
|---|---|---|
| Relacional | Datos como tablas, álgebra relacional | PostgreSQL |
| Documental | Datos como documentos JSON anidados | MongoDB |
| Grafo | Datos como nodos y aristas | Neo4j |
| Clave-valor | Diccionario distribuido | Redis |
| Columnar | Almacenamiento por columnas, no por filas | ClickHouse |

El modelo **columnar** es particularmente interesante para analytics: si una tabla tiene 100 columnas pero solo consultas 3, el motor solo lee esas 3 columnas del disco en lugar de filas completas. Enorme ahorro de I/O.

## El teorema CAP y bases de datos distribuidas

Cuando una base de datos se distribuye en múltiples nodos, aparece el **Teorema CAP**: es imposible garantizar simultáneamente las tres propiedades:

- **C**onsistency: todos los nodos ven los mismos datos al mismo tiempo
- **A**vailability: el sistema siempre responde
- **P**artition tolerance: el sistema funciona aunque haya fallas de red entre nodos

En la práctica, la partición de red es inevitable, así que el diseño elige entre consistencia y disponibilidad. Cassandra elige AP (disponible pero eventualmente consistente); HBase elige CP (consistente pero puede volverse no disponible).

En esencia, una base de datos es un sistema que resuelve con elegancia la tensión entre **rendimiento** (acceso rápido a datos), **correctitud** (ACID, integridad) y **concurrencia** (múltiples usuarios simultáneos). Cada decisión de diseño es un trade-off entre estos tres ejes.


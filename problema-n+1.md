## El Problema N+1

Es uno de los problemas de rendimiento más comunes en aplicaciones que usan ORMs (como Django, ActiveRecord, Hibernate, Prisma, etc.).

## ¿Qué es?

Ocurre cuando para obtener una lista de N registros, la aplicación termina ejecutando **1 consulta inicial + N consultas adicionales** (una por cada registro), en lugar de resolver todo en una sola consulta eficiente.

### Ejemplo concreto

Si tenemos `Autores` y `Libros`, y querermos mostrar cada autor con sus libros:

```python
# Django ORM
autores = Autor.objects.all()  # 1 consulta: SELECT * FROM autores

for autor in autores:
    print(autor.libros.all())  # N consultas: SELECT * FROM libros WHERE autor_id = ?
    # se hicieron n + 1 consultas para mostrar la información
```

Si hay 100 autores entonces se realizan **101 consultas** al servidor. Si hay 1000 se realizan **1001 consultas**. La base de datos se aplasta.

## ¿Por qué ocurre?

Porque los ORMs cargan las relaciones de forma **lazy (perezosa)** por defecto. No buscan los datos relacionados hasta que realmente los necesitemos. Esto parece conveniente, pero en un loop puede ser perjudicial.

## ¿Cómo se soluciona?

La solución central es la **carga anticipada (eager loading)**: decirle al ORM que traiga los datos relacionados desde el principio, en la misma consulta o en una consulta adicional optimizada.

### En Django: `select_related` y `prefetch_related`

```python
# select_related hace un JOIN (para relaciones 1:1 o N:1)
autores = Autor.objects.select_related('editorial').all()

# prefetch_related → hace una segunda consulta y une en memoria (para 1:N o N:M)
autores = Autor.objects.prefetch_related('libros').all()
# Resultado: 2 consultas en total, sin importar cuántos autores haya
```

### En SQL puro: JOINs

```sql
-- En lugar de múltiples SELECT separados:
SELECT autores.*, libros.*
FROM autores
LEFT JOIN libros ON libros.autor_id = autores.id;
```

## ¿Cómo detectarlo?

El problema es que **el código se ve limpio** y solo se nota cuando medís el rendimiento. Algunas formas de detectarlo:

- **Django Debug Toolbar** muestra todas las queries ejecutadas en cada request
- **Hibernate Statistics** / **p6spy** en Java
- **Logs de la base de datos** activar query logging y buscar patrones repetidos
- **APM tools** como Datadog, New Relic o Sentry Performance

## Implicaciones en producción

| Situación | Consultas ejecutadas |
|---|---|
| 10 registros, sin optimizar | 11 queries |
| 100 registros, sin optimizar | 101 queries |
| 1.000 registros, sin optimizar | 1.001 queries |
| 1.000 registros, con eager loading | 2 queries |

Esto se traduce en **mayor latencia**, **sobrecarga en la base de datos**, **timeouts**, y en casos extremos, caídas del servicio completo.


## Consideraciones importantes

- El eager loading tampoco es gratis: traer demasiados datos de golpe puede consumir mucha memoria. Hay que encontrar el equilibrio.
- En APIs paginadas, el N+1 es más controlable porque trabajás con conjuntos pequeños, pero sigue siendo un problema.
- Algunos ORMs modernos como **Prisma** o **SQLAlchemy 2.0** tienen modos más estrictos que ayudan a evitarlo por diseño.

# PruebaTecnicaTuya
La solución técnica debe crearse en un repositorio de GitHub (público), el cual debe tener un documento README en el cual explique el desarrollo que dio a cada ejercicio.

# Prueba Técnica - Analista de Datos

Este repositorio contiene la solución a la prueba técnica de Analista de Datos. El proyecto está dividido en fases, siguiendo la estructura del documento original.

## Fase 1: Extracción y Combinación de Datos (SQL)

En esta fase se presentan las consultas SQL solicitadas, basadas en el diagrama entidad-relación proporcionado.

### 1. Número total de órdenes registradas.

Explicación: Esta parte de la consultaa cuent el número de filas en la columna id_orden. Como id_orden es la clave principal y esta no puede ser null, tomando count(id_orden) o count(*) se obtiene el mismo resultado para la totalidad de órdenes. Se nombra este conteo con el alias de "total_ordenes" a la columna resultante haciendo que el resultado sea más legible.
Y por ultimo el From orden nos indica que se esta realizando una operación sobre la tabla "orden"

### 2. Número de clientes que han realizado órdenes entre el 01-01-2021 y la fecha actual.

Explicación: Se toma el identificador de los clientes disntitos (cedula) para asegurarnos de que cada cliente se cuente solo una ve, incluso si ha realizado múltiples ordenes en el periodo. Con el from cliente c, se inicia la consultada desde la respectiv tabla y se abrevia con el alias "c". Se continua con un INNER JOIN orden o ON c.cedula = o.cedula. Al unir esta tabla cliente con la tabla orden usando la columna cedula, esta es la clave foránea que las relaciona. Se le asigno a orden el alias "o".
Con el WHERE o.fecha_orden >= '2021-01-010' AND o.fecha_orden <= CURRENT_DATE(); Se filtra las órdenes para incluir solo aquellas cuya fecha_orden se encuentre en el rango especificado. Se usa CURRENT_DATE() Porquue es el estandar para traer la fecha actual del sistema, no obstante para otros sistemas varia ligeramente usando GETDATE() o NOW()

### 3.  Listado total de clientes con la cantidad total de órdenes realizadas (conteo), el valor total gastado en todas sus órdenes (suma), y el nombre del producto que más ha comprado (nombre del producto más frecuente).

Explicación:

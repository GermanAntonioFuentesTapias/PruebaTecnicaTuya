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

Explicación: Con el FROM cliente c: se parte de la tabla cliente con alias c para que con el LEFT JOIN orden o ON c.cedula = o.cedula: se une con la tabla de órdenes para contar cuántas hizo cada cliente. Se usa LEFT JOIN para no excluir a los que no han comprado nada. Se realiza otro LEFT JOIN ClienteProductoMasComprado cpmc ON c.cedula = cpmc.cedula: se conecta la CTE que contiene el producto más comprado. Teniendo un valor adicional para asi con COUNT(DISTINCT o.id_orden): asegurando que se cuenten solo órdenes distintas por cliente. Con SUM(o.total_pedido) se suma el total gastado por cliente. para luego agrupar el MAX(CASE WHEN cpmc.rn = 1 THEN cpmc.nombre_producto END) con el que se retorna el producto más comprado por el grupo de interes cliente, ya que rn = 1 representa el top 1 y MAX ignora NULL. Se agrupa con GROUP BY c.cedula, c.nombre por cliente y se ordena ORDER BY cantidad_total_ordenes DESC: se ordena desde el que más órdenes hizo al que menos.

##4.  Detalle completo (datos del cliente, fecha, nombre producto, cantidad) del pedido cuyo monto fue el más grande (en valor, no en unidades) en el año 2020.

Explicación:
SELECT c.nombre AS nombre_cliente, c.telefono AS telefono_cliente, o.fecha_orden, o.total_pedido, p.nombre_producto, do.cantidad: Selecciona todas las columnas de detalle solicitadas.
Luego con FROM cliente c INNER JOIN orden o ... INNER JOIN detalle_orden do ... INNER JOIN producto p ...: Realiza los INNER JOIN necesarios para conectar todas las tablas y obtener la información completa del cliente, la orden, los productos y las cantidades.
WHERE o.id_orden = (...): Aquí es donde la consulta principal utiliza el resultado de la subconsulta para filtrar y obtener los detalles de la orden específica que tiene el total_pedido más grande en 2020.

##5. Valor total vendido por mes y año.
Explicación:

Con SELECT STRFTIME('%Y', fecha_orden) AS año, STRFTIME('%m', fecha_orden) AS mes: Extrae el año (%Y) y el mes (%m) de la columna fecha_orden. Nota de compatibilidad de fechas:
SQLite: YEAR(fecha_orden) y MONTH(fecha_orden). Con SUM(total_pedido) AS valor_total_vendido: Calcula la suma del total_pedido para cada grupo. Luego FROM orden: Indica que estamos operando sobre la tabla orden. Se agrupa con GROUP BY año, mes:  Esto asegura que la suma (SUM) se calcule para cada período mensual/anual.
ORDER BY año, mes: Ordena el resultado cronológicamente por año y luego por mes, lo que facilita la lectura y el análisis de la tendencia.


##6.Para el cliente con cédula 123456, especificar para cada producto, el número de veces que lo ha comprado y el valor total gastado en dicho producto. Ordenar el resultado de mayor a menor.
Explicación:
Con SELECT pr.nombre_producto: Selecciona el nombre del producto. COUNT(DISTINCT o.id_orden) AS numero_veces_comprado:
COUNT(DISTINCT o.id_orden) cuenta el número de órdenes distintas en las que este cliente ha comprado el producto. Si un cliente compra el mismo producto en la misma orden varias veces , esta cuenta solo una vez esa orden. Si la intención es contar cuántas líneas de pedido para ese producto ha tenido el cliente , entonces podrías usar COUNT(do.id_producto) o COUNT(*). La formulación COUNT(DISTINCT o.id_orden) suele ser más útil para "número de veces que lo ha comprado" a nivel de orden.
SUM(do.cantidad * do.precio_unitario) AS valor_total_gastado_en_producto: Calcula el valor total gastado en cada producto. Multiplica la cantidad comprada de ese producto en la detalle_orden por su precio_unitario (que está en detalle_orden para reflejar el precio en el momento de la compra, lo cual es una buena práctica).
FROM cliente c ... INNER JOIN orden o ... INNER JOIN detalle_orden do ... INNER JOIN producto p ...: Realiza los INNER JOIN necesarios para conectar todas las tablas y acceder a la información del cliente, sus órdenes, los detalles de esas órdenes y los productos.
WHERE c.cedula = '123456': Filtra los resultados para incluir solo las compras realizadas por el cliente con la cedula '123456'. Importante: Confirma el tipo de dato de la columna cedula. Si es numérico (INT), quita las comillas (c.cedula = 123456). Si es de texto (VARCHAR), las comillas son necesarias. Asumo VARCHAR por la naturaleza de los números de identificación.
GROUP BY p.nombre_producto: Agrupa los resultados por el nombre del producto para que las funciones COUNT y SUM operen sobre cada producto individualmente.
ORDER BY valor_total_gastado_en_producto DESC: Ordena el resultado final de mayor a menor valor total gastado en cada producto.

##7. Si necesitas actualizar una tabla histórica con los datos del último mes, y en este nuevo mes has incluido una nueva columna para la ciudad del cliente, ¿qué proceso seguirías para evitar conflictos por diferencia de dimensiones, considerando que no tienes acceso a los comandos ADD COLUMN o ALTER TABLE?

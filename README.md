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

El proceso con mas detalle de la consulta o pregunta 7 se encuentra en fase1_sql con mayor claridad.

**Fase 2. Se desarrollo en el archivo fase2, no obstante traigo por facilidad.*

### Se extrae la informacion de la base de datos dada para hacer un analisis exploratorio que se observa en el Dashboard.
Se analizo Ejecutivo por su peso , logrando inicialmente estos resultados.

1. Análisis con Python

Hemos identificado los siguientes cargos y su productividad promedio:

cargobase	capt_tot (Productividad Promedio)
EJECUTIVO COMERCIAL	8.58947
EJECUTIVO COMERCIAL MEDIO TIEMPO	4.6162
EJECUTIVO COMERCIAL FIN DE SEMANA	4.28547

El enunciado nos proporciona los costos asociados a cada tipo de empleo:

100% tiempo completo
50% medio tiempo
40% fines de semana

Se determina el cargo más conveniente calculando la productividad por unidad de costo. Cuanta productividad se obtiene por cada 'unidad de costo'.

Cargo	 Productividad Promedio (capt_tot)	Costo (%)	Productividad / Costo
EJECUTIVO COMERCIAL               |	8.58947	|100|	8.58947 / 100 = 0.08589
EJECUTIVO COMERCIAL  MEDIO TIEMPO |	4.6162	|50 |	4.6162 / 50 = 0.09232
EJECUTIVO COMERCIAL FIN DE SEMANA |	4.28547	|40 |	4.28547 / 40 = 0.10714

Al comparar la "Productividad / Costo", el EJECUTIVO COMERCIAL FIN DE SEMANA muestra el valor más alto (0.10714), seguido por el EJECUTIVO COMERCIAL MEDIO TIEMPO (0.09232) y finalmente el EJECUTIVO COMERCIAL (0.08589).

Desde esta perspectiva de eficiencia costo- productividad, se enfocaria en contratar EJECUTIVO COMERCIAL FIN DE SEMANA. Aunque su productividad total individual es menor, su costo proporcionalmente reducido los hace más eficientes en términos de retorno de inversión por unidad de costo.

**Fase 3*

El objetivo es aplicar técnicas de aprendizaje no supervisado para identificar patrones y generar segmentos. El clustering, específicamente K-Means, es una excelente opción para esto.
Análisis y Preprocesamiento de Datos (Python) :Primero, necesitamos preparar los datos del df_catt para el clustering.
Identificar Variables Numéricas: Las columnas relevantes para el clustering son las métricas numéricas. Excluiremos negocio, region (categóricas) y canal (identificador).
Manejo de Valores Faltantes: Observamos que aprobacion_tarjetas y aprobacion_creditos tienen algunos valores nulos. Los imputaremos con la media.
Escalado de Características: Es fundamental escalar las características numéricas antes de aplicar K-Means, ya que el algoritmo es sensible a la escala de las variables.

La primera salida de codigo fue esta:
![image](https://github.com/user-attachments/assets/bf540a35-613e-4877-a849-0415eb0b077e)

![image](https://github.com/user-attachments/assets/1c5a13e2-3c1c-4541-a7f0-7da77b35fd37)

En la imagen anterior está el siguiente paso para la Fase 3, usando el método del codo para determinar el número óptimo de clusters. Determinación del Número Óptimo de Clusters (Método del Codo)
Interpretación del Gráfico del Codo:

Observando el gráfico, podemos identificar un "codo" claro alrededor de K = 3 o K = 4. La disminución en la Suma de los Cuadrados de los Errores (SSE) se desacelera significativamente a partir de estos puntos. Para obtener segmentos significativos y manejables, un número de clusters entre 3 y 4 parece apropiado.

Para este análisis, optaremos por K = 3 clusters. Este número suele ofrecer un buen equilibrio entre la granularidad de los segmentos y la facilidad de interpretación.

Clasificación de los puntos de venta en segmentos (Continuación)
Hemos aplicado el algoritmo K-Means con 3 clusters a los datos escalados y hemos obtenido las características promedio para cada clúster.

Interpretación de los Segmentos (Clustering):

Basándonos en las medias de cada característica por cluster, podemos describir los segmentos de la siguiente manera:

Cluster 0: "Puntos de Venta de Bajo Rendimiento General"

Características: Este segmento representa los puntos de venta con los valores más bajos en casi todas las métricas, incluyendo capturas de tarjetas y créditos, cantidad y monto de créditos, seguros, tráfico y contribución. Su aprovechamiento de tráfico es relativamente bajo.
Estrategia Sugerida: Necesitan programas de mejora intensiva, capacitación, y posiblemente reevaluación de su ubicación o estrategia de marketing. Son los puntos con mayor oportunidad de crecimiento si se les da el soporte adecuado.
Cluster 1: "Puntos de Venta de Alto Volumen de Tarjetas / Solo Tarjetas"

Características: Se destacan por tener las más altas capturas_tarjetas y tarjetas en comparación con los otros clusters. Sin embargo, lo más llamativo es que tienen valores cercanos a cero en todas las métricas de creditos, seguros, trafico_transaccional, trafico_clientes y aprovechamiento_de_trafico, así como una contribucion alta solo por las tarjetas. Esto podría indicar que son puntos de venta altamente especializados o enfocados casi exclusivamente en la colocación de tarjetas, o bien que los datos de otras métricas están incompletos/ausentes para este grupo. Su aprobación de tarjetas es similar a los demás.
Estrategia Sugerida: Reconocer su fortaleza en tarjetas. Si su especialización es intencional, se puede buscar cómo maximizar aún más este nicho. Si es una oportunidad para diversificar, se deberían introducir estrategias para impulsar la venta de créditos y seguros, así como el tráfico, aunque el tráfico cero es un punto a investigar.
Cluster 2: "Puntos de Venta de Rendimiento Medio y Balanceado"

Características: Representan un grupo intermedio en términos de volumen general de capturas de tarjetas y créditos, tráfico y contribución. Tienen los valores más altos en aprobacion_creditos y aprovechamiento_de_trafico, lo que sugiere que son eficientes en convertir el tráfico en ventas, especialmente en créditos.
Estrategia Sugerida: Estos puntos de venta son un buen modelo a seguir y pueden ser un "piso" de rendimiento para otros. Se puede buscar escalar sus mejores prácticas. También son candidatos para estrategias de crecimiento que les permitan pasar al siguiente nivel de volumen, basándose en su buena tasa de conversión.

**Fase 4.*

Fase 2 (Análisis de Datos para Productividad):
Variables: cargobase (tipo de ejecutivo), capt_tot (productividad, asumiendo "capturas totales" como proxy), y los costos asociados (100% full-time, 50% part-time, 40% weekend) provistos en el enunciado.
Metodología: Análisis descriptivo y cálculo de una métrica de eficiencia: productividad_por_costo (Productividad Promedio / Costo Asociado). Esto permitió identificar el cargo más eficiente a nivel global.
Fase 3 (Clasificación de los Puntos de Venta en Segmentos):
Variables: Métricas numéricas del dataset Segmentacion_CATT.xlsx - CATT.csv (ej. capturas_tarjetas, aprobacion_tarjetas, trafico_transaccional, contribucion, etc.).
Metodología: Se utilizó un modelo de aprendizaje no supervisado (K-Means Clustering).
Preprocesamiento: Imputación de valores nulos con la media y escalado de características (StandardScaler) para preparar los datos para el clustering.
Determinación de K: Se empleó el Método del Codo para identificar el número óptimo de clusters (se eligió K=3).
Clustering: Se aplicó K-Means para agrupar los puntos de venta en 3 segmentos distintivos.
Análisis de Clusters: Se calculó el promedio de las características originales para cada cluster para interpretar sus perfiles.
Fase 4 (Integración de Fases):
Variables: Se combinaron las columnas canal y Cluster del dataset de segmentación con la tabla de productividad (cargobase, capt_tot).
Metodología: Se realizó un merge de las tablas y se recalculó la productividad_por_costo por cargobase dentro de cada Cluster.
2. Hallazgos Importantes y Justificación de Contratación

Hallazgos Clave:

Productividad General por Costo (Fase 2):

Ejecutivo Comercial Fin de Semana: 0.10714
Ejecutivo Comercial Medio Tiempo: 0.09232
Ejecutivo Comercial: 0.08589
Conclusión Inicial: A nivel global, el Ejecutivo Comercial Fin de Semana es el más eficiente en términos de productividad por unidad de costo.
### 📈 Determinación del número óptimo de clusters:
Se usó el **método del codo**, que sugirió **K = 3**.

### 🧩 Segmentos encontrados:

- **Cluster 0:** Bajo rendimiento general. Necesitan intervención o reducción de costos.
- **Cluster 1:** Alta especialización en tarjetas. Muy eficientes en fines de semana.
- **Cluster 2:** Buen rendimiento balanceado. Alta eficiencia en conversión de tráfico.

---
Rendimiento intermedio pero con buena eficiencia en la conversión (alta aprobacion_creditos, aprovechamiento_de_trafico).
Justificación de Contratación (¿Qué cargo y dónde?):

Ahora, al combinar la productividad por costo con la segmentación de los puntos de venta, obtenemos recomendaciones más granulares y estratégicas:

Productividad por Costo por Cargo y por Cluster:

Recomendaciones Específicas por Segmento de Puntos de Venta:

Para Puntos de Venta del Cluster 0 (Bajo Rendimiento General): Se recomienda priorizar la contratación de Ejecutivos Comerciales de Medio Tiempo. Su eficiencia por costo es la más alta en este segmento, lo que sugiere que una inversión controlada con un perfil de medio tiempo es la forma más efectiva de iniciar la mejora de la productividad en estos puntos, sin incurrir en altos costos iniciales que quizás no justifiquen el retorno.
Para Puntos de Venta del Cluster 1 (Alto Volumen de Tarjetas / Solo Tarjetas): Se recomienda enfocar la contratación en Ejecutivos Comerciales de Fin de Semana. Estos ejecutivos muestran una eficiencia notablemente superior en este segmento. Esto podría indicar que el público objetivo para tarjetas está más activo durante los fines de semana, o que este perfil de ejecutivo se adapta mejor a la dinámica de venta de tarjetas en estos canales.
Para Puntos de Venta del Cluster 2 (Rendimiento Medio y Balanceado): La mejor opción es contratar Ejecutivos Comerciales de Medio Tiempo. Estos puntos ya son eficientes y balanceados, y una contratación a medio tiempo ofrece el mejor retorno por unidad de costo para mantener y potencialmente escalar su buen desempeño sin una inversión desproporcionada.
3. Conclusiones Generales y Próximos Pasos

Conclusiones para el Incremento de Productividad:

La clave para incrementar los niveles de productividad de la compañía radica en una estrategia de contratación diferenciada y segmentada, en lugar de un enfoque único para todos los puntos de venta. La eficiencia no solo se trata de la productividad bruta, sino de la productividad en relación con el costo de la fuerza laboral.

Los Ejecutivos de Fin de Semana son excepcionalmente eficientes en puntos de venta especializados en tarjetas.
Los Ejecutivos de Medio Tiempo son la opción más rentable para puntos de venta con bajo rendimiento y aquellos con un desempeño ya balanceado.
La contratación a tiempo completo es la menos eficiente en términos de productividad por costo en todos los segmentos, aunque individualmente producen más. Esto sugiere que su alto costo requiere un retorno proporcionalmente más alto, lo cual no se observa en esta métrica.

### Recomendaciones por segmento:

| Cluster | Cargo Recomendado                    | Justificación |
|---------|---------------------------------------|---------------|
| 0       | Ejecutivo Medio Tiempo               | Ideal para zonas de bajo rendimiento por su bajo costo |
| 1       | Ejecutivo Fin de Semana              | Alta eficiencia donde dominan las tarjetas |
| 2       | Ejecutivo Medio Tiempo               | Balance costo/beneficio ideal para seguir escalando |

---


Próximos Pasos para Mejorar o Expandir el Análisis:

Validación de Segmentos: Realizar una validación de los clusters con expertos de negocio. ¿Estos segmentos tienen sentido desde una perspectiva operativa?
Análisis de Atributos Cualitativos: Si se dispone de datos cualitativos (ej. tipo de ubicación del punto de venta, tamaño, demografía del área), incorporarlos al análisis para enriquecer la descripción de los clusters.
Análisis de Estacionalidad: La columna mes en la tabla de productividad podría usarse para ver si la productividad y la eficiencia por cargo varían estacionalmente, lo que podría informar sobre momentos óptimos para campañas de contratación.
Optimización del Número de Clusters: Explorar otros métodos para determinar el número óptimo de clusters (ej. Silhouette Score) para confirmar o refinar la elección de K=3.
Análisis de Contribución Marginal: Estudiar el impacto incremental de añadir un ejecutivo de cada tipo a cada cluster, en lugar de solo promedios, para entender el potencial de crecimiento.
Simulación de Escenarios: Desarrollar un modelo que permita simular el impacto en la productividad total y en los costos al ajustar la composición de la fuerza laboral en cada tipo de punto de venta.
Considerar la Retención: ¿Qué tipo de cargo tiene mayor o menor rotación? Esto podría influir en la decisión de contratación a largo plazo.





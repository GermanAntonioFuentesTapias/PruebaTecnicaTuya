# PruebaTecnicaTuya
La solución técnica debe crearse en un repositorio de GitHub (público), el cual debe tener un documento README en el cual explique el desarrollo que dio a cada ejercicio.

# Prueba Técnica - Analista de Datos

Este repositorio contiene la solución a la prueba técnica de Analista de Datos. El proyecto está dividido en fases, siguiendo la estructura del documento original.

## Fase 1: Extracción y Combinación de Datos (SQL)

En esta fase se presentan las consultas SQL solicitadas, basadas en el diagrama entidad-relación proporcionado.

### 1. Número total de órdenes registradas.

Explicación: Esta parte de la consultaa cuent el número de filas en la columna id_orden. Como id_orden es la clave principal y esta no puede ser null, tomando count(id_orden) o count(*) se obtiene el mismo resultado para la totalidad de órdenes. Se nombra este conteo con el alias de "total_ordenes" a la columna resultante haciendo que el resultado sea más legible.
Y por ultimo el From orden nos indica que se esta realizando una operación sobre la tabla "orden"

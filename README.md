# Modelo de Datos Transaccional y Dimensional – BCP

Este repositorio documenta la definición de **modelos transaccionales y dimensionales**, así como los procesos **ETL** correspondientes, con el propósito de construir un **Data Warehouse** orientado al análisis de operaciones bancarias y la detección de fraude.

---

## Estructura del Proyecto

El proyecto se organiza en tres niveles principales:

1. **Transaccional (`transaccional`)**

   * Modelo **OLTP** que representa el sistema bancario en operación.
   * Tablas principales:

     * `cliente`
     * `cuenta`
     * `dispositivo`
     * `ubicacion`
     * `transaccion`
     * `alerta_fraude`

2. **Dimensional (`dimensional`)**

   * Modelo **OLAP** diseñado para análisis en inteligencia de negocios.
   * Dimensiones: `dim_cliente`, `dim_producto`, `dim_tiempo`, `dim_canal`.
   * Tabla de hechos: `fact_transaccion`.
   * Incluye índices de optimización para acelerar consultas de auditoría y análisis.

3. **Staging y Master (`stg_bcp` y `dim_master_bcp`)**

   * **Staging (`stg_bcp`)**: espacio intermedio de carga de datos provenientes de `tr_raw_bcp`.
   * **Master (`dim_master_bcp`)**: modelo dimensional enriquecido con métricas adicionales (ejemplo: valor de vida del cliente, probabilidad de deserción, riesgo).

---

## Proceso ETL

El proceso ETL comprende las siguientes fases:

1. **Extracción**

   * Obtención de datos desde el esquema `tr_raw_bcp`.

2. **Transformación**

   * Limpieza de datos mediante funciones como `COALESCE`, `MAX` y `SUM`.
   * Segmentación de clientes (Premium, Estándar, Básico).
   * Clasificación de riesgo en función de montos acumulados.
   * Identificación de deserción por inactividad mayor a 90 días.
   * Generación de métricas:

     * `clv` (Customer Lifetime Value).
     * `prob_desercion` y `prob_riesgo` (valores simulados con `RANDOM()`).

3. **Carga**

   * Inserción de datos transformados en `stg_bcp`.
   * Transferencia hacia `dim_master_bcp`, con manejo de duplicados mediante `ON CONFLICT`.
   * Consolidación final en la tabla de hechos `fact_transaccion`, vinculando todas las dimensiones.

---

## Tablas Principales

### Transaccional

* `cliente`: datos básicos de clientes.
* `cuenta`: información de cuentas bancarias.
* `transaccion`: movimientos financieros vinculados a dispositivos y ubicaciones.
* `alerta_fraude`: registros de alertas generadas por reglas o modelos de análisis.

### Dimensional

* `dim_cliente`: información enriquecida de clientes.
* `dim_producto`: catálogo de productos financieros.
* `dim_tiempo`: calendario con granularidad diaria, mensual, trimestral y anual.
* `dim_canal`: canales de interacción (web, móvil, agencia).
* `fact_transaccion`: tabla de hechos con métricas financieras y banderas de irregularidad.

---

## Optimización

* Creación de índices en llaves foráneas de `fact_transaccion`.
* Índices adicionales en columnas clave para consultas frecuentes (`dni`, `tipo_producto`, `fecha`).
* Clasificación de transacciones por `tipo_transaccion` y `estado` para auditoría eficiente.

---

## Mantenimiento

Incluye scripts de reinicio para pruebas e integraciones:

```sql
TRUNCATE TABLE stg_bcp.* RESTART IDENTITY CASCADE;
TRUNCATE TABLE dim_master_bcp.* RESTART IDENTITY CASCADE;
```

---

## Posibles Mejoras

* Implementación de **Slowly Changing Dimensions (SCD Tipo 2)** para mantener histórico de cambios en clientes y productos.
* Incorporación de un catálogo de divisas para soportar operaciones en múltiples monedas.
* Derivación de la dimensión `dim_canal` a partir de reglas de negocio basadas en `tipo_dispositivo`.
* Inclusión de métricas de fraude avanzadas, como puntajes provenientes de modelos de aprendizaje automático.

---

## Referencias

* Banco de Crédito del Perú. (n.d.). *Historia*. Recuperado de: [https://www.viabcp.com/nosotros](https://www.viabcp.com/nosotros)
* Banco de Crédito del Perú. (2023). *Memoria integrada*. Wikipedia. Recuperado de: [https://en.wikipedia.org/wiki/Banco_de_Cr%C3%A9dito_de](https://en.wikipedia.org/wiki/Banco_de_Cr%C3%A9dito_de)


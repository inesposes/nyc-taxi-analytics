# 🚖 Proceso ETL y Visualización de Datos - NYC Taxi Data (2020–2024)

##  1️⃣ Introducción

Este proyecto implementa un pipeline ETL completo sobre los datos históricos de los taxis de la ciudad de Nueva York (*NYC Taxi Data*) correspondientes al período **2020–2024**. Utilizando **Databricks** como plataforma de procesamiento, se desarrolló una arquitectura en capas (**Bronze**, **Silver** y **Gold**) basada en el enfoque *Delta Lake*, para preparar los datos y generar tablas optimizadas para análisis y visualización de KPIs a través de dashboards interactivos.

---

## 2️⃣ Arquitectura

### Capa Bronze – Ingesta Cruda

En esta capa se realiza la **ingesta de los archivos CSV originales** mensuales del NYC Taxi Data. Los archivos se leen desde un volumen en Databricks, se combinan en un único DataFrame y se almacenan en **formato Parquet**.

---

### Capa Silver – Limpieza y Transformación

En esta etapa se aplican transformaciones para mejorar la **calidad y tulidad** de los datos. 

#### Transformaciones realizadas:
- Eliminación de **duplicados** y **valores nulos**.
- Filtrado de **viajes sin pasajeros**.
- Imputación de **distancias igual a cero** con la **mediana**.
- Conversión de fechas (`tpep_pickup_datetime`) a tipo **timestamp**.
- Creación de nuevas variables temporales:  
  `pickup_year`, `pickup_month`, `pickup_day`, `pickup_hour`.
- Traducción de día y mes a formato **texto en español**.
- Corrección de valores negativos en `fare_amount` y `tips_amount`.
- Filtrado de registros fuera del rango **2020–2024**.
- Enriquecimiento de datos con zonas geográficas:
  - Unión (`JOIN`) con dataset de ubicaciones para obtener:  
    `PU_Borough`, `PU_Zone`, `DO_Borough`, `DO_Zone`.
  - Creación del campo `Borough_Concat`, combinando origen y destino de los distritos.

---

### Capa Gold – Agregaciones y Modelado Analítico

Se generan **tablas modelo** orientadas aa la visualización en el dashboards.

> Todas las tablas de esta capa se almacenan en **formato Delta**, lo que permite mejoras en rendimiento, integridad de datos, y funcionalidades como actualizaciones, control de versiones y soporte para transacciones ACID.


#### Tablas generadas:

##### 📊 `generalInsights`
Contiene información general del comportamiento de los viajes.

| Campo              | Descripción                         |
|--------------------|-------------------------------------|
| `pickup_year`      | Año de la recogida del viaje        |
| `pickup_month`     | Mes de la recogida                  |
| `pickup_hour`      | Hora de recogida                    |
| `avg_fare`         | Tarifa promedio                     |
| `avg_distance`     | Distancia promedio de viajes        |
| `total_passengers` | Total de pasajeros                  |

---

##### 👥 `passengersBehaviour`
Permite analizar los patrones de comportamiento de los pasajeros.

| Campo              | Descripción                         |
|--------------------|-------------------------------------|
| `total_passengers` | Total de pasajeros                  |
| `avg_tips`         | Propinas promedio                   |
| `pickup_hour`      | Hora de recogida                    |
| `pickup_day`       | Día de la semana de recogida        |

---

##### 🌍 `tripsLocation`
Información geográfica sobre los viajes entre distritos y zonas.

| Campo             | Descripción                          |
|-------------------|--------------------------------------|
| `Borough_Concat`  | Concatenación de distrito origen-destino |
| `DO_Zone`         | Zona de destino                      |
| `PU_Zone`         | Zona de origen                       |
| `DO_Borough`      | Distrito de destino                  |
| `PU_Borough`      | Distrito de origen                   |
| `pickup_day`      | Día de la semana de recogida         |

---

## 3️⃣ Automatización del Pipeline

El proceso ETL está completamente **automatizado** mediante un **pipeline de Databricks Workflows**, que ejecuta mensualmente el notebook de procesamiento.

- **Frecuencia:** 5 de cada mes  
- **Hora de ejecución:** 3:00 AM  
- **Notificaciones:** Envío de correos electrónicos en caso de **éxito ✅** o **error ❌** en la ejecución.

---

## 4️⃣ Dashboards e Indicadores

A partir de las tablas Gold, se desarrollan dashboards interactivos con KPIs clave como:

- Tarifa promedio por hora y mes.
- Distribución de propinas por día de la semana.
- Rutas más frecuentes entre distritos.
- Número de pasajeros por franja horaria.
- Comparativa de zonas de mayor recogida y destino.

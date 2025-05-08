# 🚖 Proceso ETL y Visualización de Datos - NYC Taxi Data (2020–2024)

##  1️⃣ Introducción

Este proyecto implementa un pipeline ETL completo sobre los datos históricos de los taxis de la ciudad de Nueva York (*NYC Taxi Data*) correspondientes al período **2020–2024**. Utilizando **Databricks** como plataforma de procesamiento, se desarrolló una arquitectura en capas (**Bronze**, **Silver** y **Gold**) para preparar los datos y generar tablas optimizadas para análisis y visualización de KPIs a través de un dashboard interactivo.

---

## 2️⃣ Arquitectura

### Capa Bronze 

En esta capa se realiza la **ingesta de los archivos CSV originales** mensuales del NYC Taxi Data. Los archivos se leen desde un volumen en Databricks, se combinan en un único DataFrame y se almacenan en **formato Parquet** en un volumen.

> La extracción de los datos se realizó mediante web scraping y se subieron los .csv manualmente a Databricks. Lo ideal hubiese sido realizar el web scraping en el notebook de la capa bronze y guardarlos dinámicamente en un volumen de Databricks, pero al usar una **versión de prueba**, Databricks no lo permitía.

---

### Capa Silver 

En esta etapa se aplican transformaciones para mejorar la **calidad y tulidad** de los datos. 

#### Transformaciones realizadas:
- Eliminación de **duplicados** y **valores nulos**.
- Filtrado de **viajes sin pasajeros**.
- Imputación de **distancias igual a cero** con la **mediana**.
- Conversión de fechas a tipo **timestamp**.
- Creación de nuevas variables temporales:  
  `pickup_year`, `pickup_month`, `pickup_day`, `pickup_hour`.
- Traducción de día y mes a formato **texto en español**.
- Corrección de valores negativos en `fare_amount` y `tips_amount`.
- Filtrado de registros fuera del rango **2020–2024** (~1000 registros).
- Enriquecimiento de datos con las zonas geográficas a las que corresponden los LocationID (PULocationID, DOLocationID) del dataset original:
  - Unión (`JOIN`) con dataset de ubicaciones para obtener:  
    `PU_Borough`, `PU_Zone`, `DO_Borough`, `DO_Zone`.
  - Creación del campo `Borough_Concat`, combinando origen y destino de los distritos.

---

### Capa Gold 

Se generan **tablas** orientadas a la visualización en el dashboard.

> Todas las tablas de esta capa se almacenan en **formato Delta**, lo que permite mejoras en rendimiento, integridad de datos, y funcionalidades como actualizaciones, control de versiones y soporte para transacciones ACID.


#### Tablas generadas:

##### 📊 `generalInsights`
Contiene información general de los viajes.

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

A partir de las tablas Gold, el dashboard se centra en tres aspectos principales de los que se extraen los siguientes KPIs:

### **Información General**
- **Media de tarifa ($):** $16.07  
- **Media de distancia (km):** 3.56 km  
- **Total de pasajeros:** 970.36 millones  

---

### **Comportamiento de los pasajeros**
- **Pasajeros por hora del día**:  
  Se detecta un pico destacado entre las **17h y 19h**, lo que impulsa la creación de una visualización más específica.
  
- **Pasajeros por hora y día de la semana**:  
  Se analiza si ese pico se repite todos los días o si varía. Se observa que en algunos días, ese rango horario no es tan relevante.

- **Pasajeros de 17h a 19h por día de la semana**:  
  Se filtra solo ese tramo horario para entender **qué días concentran más demanda**, lo que puede servir para asignar más taxis en momentos clave.

- **Propinas por hora del día**:  
  Aparece un comportamiento inesperado: las **propinas son mayores a las 5am**. Este dato puede incentivar a los conductores a cubrir turnos tempranos.

---

### **Desplazamientos**
- **Top 3 viajes más frecuentes entre distritos**:  
  - Manhattan → Manhattan: 343M
  - Manhattan → Queens: 11,4M 
  - Queens → Manhattan: 18,3M 
  Se concluye que **Manhattan concentra la mayoría de los viajes**.

- **Zonas de recogida más frecuentes en Manhattan**:  
  Dado el alto volumen de viajes dentro de Manhattan, se identifican las áreas de recogida más comunes.

- **Viajes por zona y día de la semana en Manhattan**:  
  Se analiza la demanda por zona según el día, para detectar **patrones semanales** y **optimizar la distribución de flotas**.
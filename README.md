# ✈️ Azure Data Engineering Project – Flight Operations Analytics (Argentina 2024–2025)

## 📖 Descripción general

Este proyecto fue desarrollado como trabajo integrador del curso **Data Engineering en Azure**.  
El objetivo fue diseñar e implementar un **pipeline de datos completo** utilizando **Azure Data Factory, Databricks y Data Lake**, aplicando la **arquitectura Medallion (Bronze–Silver–Gold)** para procesar y analizar información sobre vuelos comerciales en el norte argentino.

El sistema permite integrar datos provenientes de una **API REST** (vuelos y feriados), realizar **transformaciones con PySpark**, y generar **tablas Delta Lake** preparadas para análisis de estacionalidad, cancelaciones y demoras.

---

## 🧱 Arquitectura y flujo de datos

La solución implementa el enfoque **Medallion Architecture**, dividiendo el flujo en tres capas:

| Capa | Descripción | Salida |
|------|--------------|--------|
| **Bronze** | Ingesta cruda de archivos JSON desde APIs REST (vuelos y feriados). | `bronze/vuelos/`, `bronze/feriados/` |
| **Silver** | Limpieza, tipado y normalización. Creación de columnas derivadas (`delay_minutes`, `origin`, `destination`, `is_holiday`). | `silver/vuelos/`, `silver/feriados/` |
| **Gold** | Enriquecimiento analítico: join entre vuelos y feriados, cálculo de métricas operativas (`delay_mean`, `cancel_rate`), filtrado de rutas y períodos definidos. | `gold/vuelos_enriquecidos/` |

📊 **Pipeline orquestado con Azure Data Factory:**
1. **Copy Activity (REST → Data Lake)**: descarga de JSON de vuelos y feriados.  
2. **ForEach**: procesamiento por año y carpeta.  
3. **Databricks Notebook Activity**: ejecución de notebooks de transformación (bronze → silver → gold).  
4. **Output Delta Tables**: almacenamiento final en contenedor `tpfinal/silver` y `tpfinal/gold`.

---

## ⚙️ Tecnologías utilizadas

- **Azure Data Factory (ADF)** → orquestación y automatización del pipeline.  
- **Azure Databricks (PySpark)** → limpieza, enriquecimiento y creación de tablas Delta.  
- **Azure Data Lake Gen2** → almacenamiento de datos en capas.  
- **Delta Lake** → formato de almacenamiento transaccional y escalable.  
- **Python / PySpark** → procesamiento de datos.  
- **Power BI (opcional)** → visualización analítica (futuro).  

---

## 📂 Estructura del proyecto

```css
📁 azure-flight-pipeline/
│
├── notebooks/
│ ├── bronze.ipynb
│ ├── silver.ipynb
│ └── gold.ipynb
│
├── docs/
│ ├── pipeline_diagram.png
│ └── Informe_Final_DataEngineering_Vuelos.pdf
│
├── data-samples/
│ ├── feriados_2024.json
│ └── vuelos_sample.json
│
└── README.md
```


---

## 🧩 Transformaciones principales

### **Capa Bronze**
- Ingesta cruda desde endpoints REST.  
- Almacenamiento directo de archivos `.json` en contenedores del Data Lake.  
- Organización por carpetas: `year=2024/`, `year=2025/`.

### **Capa Silver**
- Limpieza y normalización de datos:
  - Tipado de fechas (`to_date`, `to_timestamp`).
  - Creación de columnas derivadas: `delay_minutes`, `origin`, `destination`.
  - Eliminación de duplicados y valores nulos.  
- Validaciones de calidad (IATA de 3 caracteres, fechas válidas).

### **Capa Gold**
- Filtro de rutas: **AEP ↔ JUJ/SLA/TUC**.  
- Período analizado: **diciembre 2024 – abril 2025**.  
- Join con tabla de feriados.  
- Cálculo de variables analíticas:
  - `is_holiday`
  - `weekday_name`
  - `delay_minutes` (limpieza de outliers)
  - `cancel_flag` (vuelos cancelados)

El resultado es un dataset consolidado que permite estudiar:
- Estacionalidad y microestacionalidad de vuelos.  
- Impacto de feriados y fines de semana largos.  
- Desempeño operativo por aerolínea.  

---

## ✅ Controles de calidad implementados

- Validación de estructura y tipado en Bronze y Silver.  
- Eliminación de nulos en campos clave (`flight_date`, `origin`, `destination`, `airline`).  
- Deduplicado de feriados por fecha y nombre.  
- Revisión de demoras con outliers (>24h).  
- Validación de joins en Gold (sin duplicados por fecha).

---

## 🧠 Resultados

El pipeline entrega una tabla Delta lista para análisis en Power BI o Databricks SQL.  
Cada vuelo contiene información enriquecida sobre:
- Aerolínea, origen, destino y fecha.  
- Estado (demorado, cancelado, puntual).  
- Día de la semana y si fue feriado o no.  

Esto permite realizar estudios posteriores sobre:
- Estacionalidad de vuelos.  
- Efecto de los feriados en la operación.  
- Comparación de desempeño entre aerolíneas.

---

## 🚀 Próximos pasos y mejoras

- Incorporar datos de todo el período **2023–2025** para obtener tendencias históricas.  
- Implementar **Databricks Autoloader** para ingestas incrementales.  
- Integrar métricas meteorológicas o de tráfico aeroportuario.  
- Conectar la capa Gold a **Power BI** para reportes automáticos.

---

## 👩‍💻 Autor

**Verónica Valdez**  
📍 Argentina  
💼 Desarrolladora Backend & Data Engineering  
📧 _contacto_: [LinkedIn](https://www.linkedin.com/in/vmvaldez/)

---

> _“Este proyecto consolida habilidades clave en ingeniería de datos con Azure: integración de fuentes heterogéneas, transformación con PySpark, control de calidad y modelado analítico en Delta Lake.”_

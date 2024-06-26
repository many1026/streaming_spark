# Proyecto de Análisis de Colisiones de Vehículos con PySpark Streaming

Este proyecto tiene como objetivo realizar un análisis de colisiones de vehículos utilizando PySpark Streaming. Se incluye un proceso de limpieza de datos, balanceo de clases y entrenamiento de un modelo de regresión logística para predecir la severidad de las colisiones. A continuación se detallan los aspectos generales del proyecto.

## Descripción del Proyecto

El análisis de datos de colisiones de vehículos es fundamental para entender y prevenir accidentes en la ciudad. Este proyecto procesa datos en tiempo real, realiza un análisis exhaustivo y entrena un modelo de machine learning para predecir la severidad de las colisiones.

## Componentes del Proyecto

### 1. Configuración de la Sesión de Spark

Se configura una sesión de Spark con memoria asignada y configuraciones adicionales para optimizar el procesamiento de datos.

### 2. Preparación de los Datos

Se leen los datos desde un directorio CSV y se crean nuevas columnas necesarias para el análisis, como la columna `Severity` basada en el número de personas lesionadas y muertas.

### 3. Balanceo de los Datos

Para manejar el desbalance de clases en el conjunto de datos, se generan datos sintéticos para asegurar una distribución equilibrada de la variable objetivo `Severity`.

### 4. Preparación de los Datos para el Modelo

Se utiliza `VectorAssembler` para combinar las columnas de características en una sola columna de vectores llamada `features`.

### 5. Entrenamiento del Modelo de Regresión Logística

Se entrena un modelo de regresión logística para predecir la severidad de las colisiones utilizando las características seleccionadas.

### 6. Evaluación del Modelo

El modelo se evalúa utilizando el área bajo la curva ROC (AUC-ROC) y se genera una gráfica ROC para visualizar el rendimiento del modelo.

## Archivos del Proyecto

- **[spark_local](./spark_local.md):** Código principal que ejecuta todo el proceso de lectura de datos, limpieza, balanceo, entrenamiento y evaluación del modelo.
- **[data_source](./data_source.md):** Directorio que contiene los archivos CSV de datos de colisiones.
- **[README.md](./README.md):** Documento de introducción y descripción general del proyecto (este archivo).

## Instalación y Ejecución

### Requisitos Previos

- Apache Spark
- PySpark
- Python 3.x
- Jupyter Notebook (opcional, para pruebas y exploración de datos)


## Resultados del Proyecto

El modelo de regresión logística entrenado se evaluó utilizando el área bajo la curva ROC (AUC-ROC). A continuación se muestra la gráfica ROC obtenida:

![ROC Curve](https://github.com/many1026/streaming_spark/assets/73008381/15f95caa-98f8-47b3-94ac-73e6a3bf4e10)

La gráfica muestra el rendimiento del modelo con un área bajo la curva de aproximadamente 0.66, lo que indica un rendimiento moderado del modelo en la predicción de la severidad de las colisiones.


## Contacto

Para preguntas o sugerencias, puede contactarme a través de [mi perfil de GitHub](https://github.com/many1026).

---

¡Gracias por visitar este proyecto y contribuir al análisis y prevención de colisiones de vehículos!

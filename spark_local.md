# Streaming en PySpark

Este documento proporciona una guía paso a paso para solucionar problemas de memoria en el heap al cargar archivos CSV con PySpark.

## Crear la Sesión de Spark con Más Memoria Asignada

Primero, crea una sesión de Spark y asigna más memoria a la JVM (Java Virtual Machine).

``` python
from pyspark.sql import SparkSession
from pyspark.sql.types import *

# Crear la sesión de Spark con más memoria asignada y configuraciones adicionales
spark = SparkSession.builder \
    .appName("TuApp") \
    .config("spark.driver.memory", "8g") \
    .config("spark.executor.memory", "8g") \
    .config("spark.sql.shuffle.partitions", "200") \
    .config("spark.sql.streaming.forceDeleteTempCheckpointLocation", "true") \
    .config("spark.memory.fraction", "0.8") \
    .config("spark.memory.storageFraction", "0.2") \
    .getOrCreate()

# Define el esquema para el CSV
csvSchema = StructType([
    StructField("Index", IntegerType(), True),
    StructField("CRASH DATE", StringType(), True),
    StructField("CRASH TIME", StringType(), True),
    StructField("BOROUGH", StringType(), True),
    StructField("ZIP CODE", StringType(), True),
    StructField("LATITUDE", DoubleType(), True),
    StructField("LONGITUDE", DoubleType(), True),
    StructField("NUMBER OF PERSONS INJURED", DoubleType(), True),
    StructField("NUMBER OF PERSONS KILLED", DoubleType(), True)
])

# Configurar el DataFrame de entrada de streaming
inputPath = "/Users/mny_1026/Downloads/csv_directory"

streamingInputDF = (
    spark
        .readStream
        .schema(csvSchema)
        .option("maxFilesPerTrigger", 1)
        .option("header", "true")
        .format("csv")
        .load(inputPath)
)



```
En este paso, estamos configurando PySpark para cargar un archivo por cada trigger (maxFilesPerTrigger)
y estamos indicando que los archivos CSV tienen una fila de encabezado (header).
``` python
# Iniciar la Consulta de Escritura en la Consola
query = streamingInputDF.writeStream.outputMode("append").format("console").start()
``` 
Este codigo inicia la consulta y espera a que termine para mostrar los datos en la consola en modo append.
# Resultado:
<img width="1227" alt="image" src="https://github.com/many1026/streaming_spark/assets/73008381/e5a467d2-2efb-4be6-9669-709ddeb6021d">



# Analisis Interesantes
## Agrupa los datos por ventanas de tiempo de 1 hora y cuenta el número de elementos en cada grupo.
``` python
from pyspark.sql.functions import window

streamingCountsDF = (
  streamingInputDF
    .groupBy(
      streamingInputDF.BOROUGH,
      window(streamingInputDF["CRASH DATE"], "1 hour"))
    .count()
)
```
## Contar el Número Total de Accidentes por Borough
``` python
totalAccidentsByBorough = streamingInputDF.groupBy("BOROUGH").count()
totalAccidentsByBorough.writeStream.outputMode("complete").format("console").start().awaitTermination()
```
<img width="1227" alt="image" src="https://github.com/many1026/streaming_spark/assets/73008381/c0bd93e8-5b68-4c94-8d0e-301be4effed2">


## Distribución de Accidentes por Código Postal
```python
accidentsByZipCode = streamingInputDF.groupBy("ZIP CODE").count()
accidentsByZipCode.writeStream.outputMode("complete").format("console").start().awaitTermination()
```
# Resultado:
<img width="432" alt="image" src="https://github.com/many1026/streaming_spark/assets/73008381/75e24773-3bf2-417d-9f8e-ab3f7c8143b5">

## Accidentes por Hora del Día
```python
from pyspark.sql.functions import hour

accidents_by_hour = streamingInputDF.withColumn("Hour", hour("CRASH TIME")).groupBy("Hour").count()
accidents_by_hour.writeStream.outputMode("complete").format("console").start().awaitTermination()

```
# Resultado
<img width="406" alt="image" src="https://github.com/many1026/streaming_spark/assets/73008381/508e418c-40a5-4627-b56a-1eba9e9885a4">


# MODELO DE MACHINE LEARNING

<img width="1440" alt="image" src="https://github.com/many1026/streaming_spark/assets/73008381/9d0be754-5a6c-4522-8cc2-c327a33d4d34">
# Análisis de la Curva ROC para la Regresión Logística
## Descripción General
La gráfica muestra la curva ROC (Receiver Operating Characteristic) resultante de aplicar un modelo de regresión logística a los datos disponibles. La ROC es una herramienta útil para evaluar el desempeño de un clasificador binario. En el eje horizontal (X) se encuentra la Tasa de Falsos Positivos (FPR), mientras que en el eje vertical (Y) se encuentra la Tasa de Verdaderos Positivos (TPR).

# Área Bajo la Curva (AUC):
## Interpretación de la Curva
![image](https://github.com/many1026/streaming_spark/assets/73008381/15f95caa-98f8-47b3-94ac-73e6a3bf4e10)

El área bajo la curva ROC (AUC) es de 0.66. Este valor indica la capacidad del modelo para distinguir entre las clases positivas y negativas.
Una AUC de 0.66 sugiere que el modelo tiene una habilidad moderada para distinguir entre las clases. En términos generales, una AUC de 0.5 indica un modelo sin capacidad de discriminación (equivalente a un clasificador aleatorio), mientras que una AUC de 1.0 indica un modelo perfecto.
Forma de la Curva:

La curva ROC muestra un comportamiento ascendente con algunas fluctuaciones, lo cual es esperado en modelos que no son perfectos.
La curva se mantiene por encima de la línea roja diagonal, que representa un clasificador aleatorio. Esto indica que el modelo tiene un desempeño mejor que un clasificador aleatorio.

# Puntos Clave:
Al inicio (cerca de (0, 0)), la curva muestra una pendiente pronunciada, indicando que el modelo logra capturar una buena cantidad de verdaderos positivos con pocos falsos positivos.
Conforme la FPR aumenta, la TPR también lo hace, pero con fluctuaciones, lo cual indica que el modelo tiene ciertos errores en la clasificación.
Evaluación del Modelo:
Aunque el modelo tiene una AUC mayor a 0.5, lo cual es positivo, el valor de 0.66 sugiere que hay espacio para mejorar.
Dependiendo del contexto y la aplicación, una AUC de 0.66 puede ser suficiente, pero generalmente se buscaría mejorar este valor ajustando el modelo, incorporando más características, o utilizando técnicas avanzadas de modelado.

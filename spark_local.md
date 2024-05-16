# Solucionando Problemas de Memoria en el Heap con PySpark

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


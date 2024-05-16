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
    StructField("CRASH DATE", StringType(), True),
    StructField("CRASH TIME", StringType(), True),
    StructField("BOROUGH", StringType(), True),
    StructField("ZIP CODE", StringType(), True),
    StructField("LATITUDE", DoubleType(), True),
    StructField("LONGITUDE", DoubleType(), True),
    StructField("NUMBER OF PERSONS INJURED", IntegerType(), True),
    StructField("NUMBER OF PERSONS KILLED", IntegerType(), True)
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
query.awaitTermination()
``` 
Este codigo inicia la consulta y espera a que termine para mostrar los datos en la consola en modo append.


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
<img width="1440" alt="image" src="https://github.com/many1026/streaming_spark/assets/73008381/0ec88f5d-44df-44d2-b517-1d77b4bdb934">

## Calcular el Número Promedio de Lesiones por Accidentes
``` python
from pyspark.sql.functions import avg

averageInjuries = streamingInputDF.groupBy("BOROUGH").agg(avg("NUMBER OF PERSONS INJURED"))
averageInjuries.writeStream.outputMode("complete").format("console").start().awaitTermination()
```
## Distribución de Accidentes por Código Postal
```python
accidentsByZipCode = streamingInputDF.groupBy("ZIP CODE").count()
accidentsByZipCode.writeStream.outputMode("complete").format("console").start().awaitTermination()
```

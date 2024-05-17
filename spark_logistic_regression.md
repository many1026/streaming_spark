# Guía para la Regresión Logística con PySpark

## Paso 1: Configurar la Sesión de Spark

```python
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

staticInputDF = (
    spark
        .read
        .schema(csvSchema)
        .option("header", "true")
        .format("csv")
        .load(inputPath)
)
```
# Paso 2: Preparar los Datos

```python
from pyspark.sql.functions import when, col, length, concat, lit, to_timestamp

# Crear la columna "Severity" basada en las columnas de lesiones y muertes
staticInputDF = staticInputDF.withColumn(
    "Severity",
    when(col("NUMBER OF PERSONS KILLED") > 0, 1).
    when(col("NUMBER OF PERSONS INJURED") > 0, 1).
    otherwise(0)
)

# Verificar la distribución de las etiquetas en el conjunto de datos
staticInputDF.groupBy("Severity").count().show()
```
# Paso 3: Balancear los Datos
```python
from pyspark.sql import Row
import random

# Generar filas sintéticas con variación
def generate_synthetic_row(severity):
    return Row(
        Index=random.randint(0, 1000000),
        **{
            "CRASH DATE": None,
            "CRASH TIME": None,
            "BOROUGH": None,
            "ZIP CODE": None,
            "LATITUDE": random.uniform(40.0, 41.0),
            "LONGITUDE": random.uniform(-74.0, -73.0),
            "NUMBER OF PERSONS INJURED": random.randint(0, 10) if severity == 1 else random.randint(0, 5),
            "NUMBER OF PERSONS KILLED": random.randint(0, 5) if severity == 1 else random.randint(0, 2),
            "Severity": severity
        }
    )

num_synthetic_rows = 10000
synthetic_rows = [generate_synthetic_row(0) for _ in range(num_synthetic_rows // 2)] + \
                 [generate_synthetic_row(1) for _ in range(num_synthetic_rows // 2)]

synthetic_df = spark.createDataFrame(synthetic_rows, schema=staticInputDF.schema)

# Unir los datos sintéticos con el conjunto de datos original
balanced_data = staticInputDF.unionByName(synthetic_df)

# Verificar la distribución de las etiquetas en el conjunto de datos balanceado
balanced_data.groupBy("Severity").count().show()

``
# Paso 4: Preparar los Datos para el Modelo
```python
```

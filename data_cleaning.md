# Limpieza de Datos de Accidentes de Tráfico
```python
#%%
import pandas as pd
df = pd.read_csv('Motor_Vehicle_Collisions_-_Crashes.csv')
# %%
df.columns

# %%
df.columns
# %%
import pandas as pd

# Definir las columnas relevantes
relevant_columns = [
    'CRASH DATE', 'CRASH TIME', 'BOROUGH', 'ZIP CODE', 
    'LATITUDE', 'LONGITUDE', 'NUMBER OF PERSONS INJURED', 'NUMBER OF PERSONS KILLED'
]
# Conservar solo las columnas relevantes
df_relevant = df[relevant_columns]

# Mostrar el DataFrame resultante
print("DataFrame con columnas relevantes:")
print(df_relevant.head())
# %%
total_na = df_relevant.isna().sum().sum()
total_na
# %%
# INTERPOLAR
# 4. Interpolación lineal
df_interpolate = df_relevant.copy()
df_interpolate['LATITUDE'] = df_interpolate['LATITUDE'].interpolate(method='linear')
df_interpolate['LONGITUDE'] = df_interpolate['LONGITUDE'].interpolate(method='linear')
df_interpolate['NUMBER OF PERSONS INJURED'] = df_interpolate['NUMBER OF PERSONS INJURED'].interpolate(method='linear')
df_interpolate['NUMBER OF PERSONS KILLED'] = df_interpolate['NUMBER OF PERSONS KILLED'].interpolate(method='linear')
print("\nDataFrame después de la interpolación lineal:")
print(df_interpolate.head())
# %%
total_na = df_interpolate.isna().sum().sum()
total_na
# %%
columns_of_interest = ['LATITUDE', 'LONGITUDE']
na_counts_specific = df[columns_of_interest].isna().sum()
print("\nNúmero de NA en columnas específicas:")
print(na_counts_specific)
# %%
columns_of_interest = ['LATITUDE', 'LONGITUDE']
na_counts_specific = df_interpolate[columns_of_interest].isna().sum()
print("\nNúmero de NA en columnas específicas:")
print(na_counts_specific)
# %%
columns_of_interest = ['BOROUGH', 'NUMBER OF PERSONS KILLED']
na_counts_specific = df_interpolate[columns_of_interest].isna().sum()
print("\nNúmero de NA en columnas específicas:")
print(na_counts_specific)
# %%
import numpy as np
df_interpolate
# %%
# Convertir la columna 'ZIP CODE' a valores numéricos, forzando errores a NaN
df_interpolate['ZIP CODE'] = pd.to_numeric(df_interpolate['ZIP CODE'], errors='coerce')

# Imputar NA en BOROUGH con la moda
borough_mode = df_interpolate['BOROUGH'].mode()[0]
df_interpolate['BOROUGH'] = df_interpolate['BOROUGH'].fillna(borough_mode)

# Imputar NA en ZIP CODE con la media (redondeada hacia abajo)
zip_code_mean = np.floor(df_interpolate['ZIP CODE'].mean())
df_interpolate['ZIP CODE'] = df_interpolate['ZIP CODE'].fillna(zip_code_mean)
# %%
df_interpolate
# %%
df_interpolate.to_csv('Burren.csv')
```

# Pandas & Manejo de Archivos en Python 🐍

---

## 1. Instalación e importación

```python
pip install pandas openpyxl  # openpyxl para Excel

import pandas as pd
import numpy as np            # casi siempre va junto con pandas
import json
```

---

## 2. Leer archivos

### CSV
```python
df = pd.read_csv("datos.csv")
df = pd.read_csv("datos.csv", encoding="utf-8")       # con acentos
df = pd.read_csv("datos.csv", sep=";")                # separador distinto
df = pd.read_csv("datos.csv", header=None)            # sin encabezado
df = pd.read_csv("datos.csv", names=["a","b","c"])    # asignar nombres
df = pd.read_csv("datos.csv", skiprows=2)             # saltar filas
df = pd.read_csv("datos.csv", nrows=100)              # leer solo 100 filas
df = pd.read_csv("datos.csv", usecols=["col1","col2"])# solo ciertas columnas
```

### Excel
```python
df = pd.read_excel("datos.xlsx")
df = pd.read_excel("datos.xlsx", sheet_name="Hoja1") # hoja específica
df = pd.read_excel("datos.xlsx", sheet_name=0)       # primera hoja
df = pd.read_excel("datos.xlsx", sheet_name=None)    # todas las hojas → dict
```

### JSON
```python
# Con pandas (JSON tabular)
df = pd.read_json("datos.json")

# Con json nativo (JSON anidado o complejo)
with open("datos.json", "r", encoding="utf-8") as f:
    datos = json.load(f)    # → dict o list

# JSON desde string
datos = json.loads('{"nombre": "Ana", "edad": 25}')
```

### TXT
```python
# Leer todo de una vez
with open("datos.txt", "r", encoding="utf-8") as f:
    contenido = f.read()

# Leer línea por línea
with open("datos.txt", "r", encoding="utf-8") as f:
    lineas = f.readlines()         # lista con \n al final
    lineas = [l.strip() for l in f] # lista limpia sin \n
```

---

## 3. Explorar el DataFrame

```python
df.head(10)          # primeras 10 filas (default 5)
df.tail(10)          # últimas 10 filas
df.sample(5)         # 5 filas aleatorias
df.shape             # (filas, columnas)
df.columns           # nombres de columnas
df.dtypes            # tipo de cada columna
df.index             # índices

df.info()            # resumen: tipos, nulos, memoria
df.describe()        # estadísticas: media, std, min, max, percentiles
df.describe(include="all")  # incluye columnas de texto también

df.isnull().sum()    # cantidad de nulos por columna
df.nunique()         # valores únicos por columna
df["col"].value_counts()    # frecuencia de cada valor en una columna
df["col"].unique()          # valores únicos de una columna
```

---

## 4. Acceder a datos

```python
# Por columna
df["nombre"]                    # Series (una columna)
df[["nombre", "edad"]]          # DataFrame (varias columnas)

# Por fila
df.iloc[0]                      # fila por índice numérico
df.iloc[0:5]                    # filas 0 a 4
df.iloc[0, 2]                   # fila 0, columna 2

df.loc[5]                       # fila por etiqueta de índice
df.loc[0:5, "nombre"]           # filas 0 a 5, columna nombre
df.loc[0:5, ["nombre", "edad"]] # filas 0 a 5, varias columnas

# Filtrar filas con condiciones
df[df["edad"] > 18]
df[df["ciudad"] == "CDMX"]
df[(df["edad"] > 18) & (df["ciudad"] == "CDMX")]   # AND
df[(df["edad"] < 18) | (df["ciudad"] == "MTY")]    # OR
df[df["ciudad"].isin(["CDMX", "MTY", "GDL"])]      # IN
df[~df["ciudad"].isin(["CDMX"])]                   # NOT IN
df[df["nombre"].str.contains("Ana")]               # contiene texto
```

---

## 5. Limpiar datos

### Valores nulos
```python
df.isnull().sum()            # contar nulos por columna
df.dropna()                  # eliminar filas con cualquier nulo
df.dropna(subset=["edad"])   # eliminar filas con nulo en columna específica
df.dropna(axis=1)            # eliminar columnas con nulos

df.fillna(0)                         # rellenar todos los nulos con 0
df.fillna({"edad": 0, "nombre": "Desconocido"})  # por columna
df["edad"].fillna(df["edad"].mean())  # rellenar con la media
df.ffill()                           # rellenar con el valor anterior
df.bfill()                           # rellenar con el valor siguiente
```

### Duplicados
```python
df.duplicated().sum()           # cuántos duplicados hay
df.drop_duplicates()            # eliminar duplicados
df.drop_duplicates(subset=["email"])  # duplicados por columna específica
df.drop_duplicates(keep="last")       # quedarse con el último
```

### Tipos de datos
```python
df["edad"] = df["edad"].astype(int)
df["precio"] = df["precio"].astype(float)
df["fecha"] = pd.to_datetime(df["fecha"])
df["fecha"] = pd.to_datetime(df["fecha"], format="%d/%m/%Y")
```

### Texto
```python
df["nombre"] = df["nombre"].str.upper()
df["nombre"] = df["nombre"].str.lower()
df["nombre"] = df["nombre"].str.strip()          # quitar espacios
df["nombre"] = df["nombre"].str.replace("-", "") # reemplazar
df["nombre"] = df["nombre"].str.split(",")       # dividir en lista
df[["nombre", "apellido"]] = df["nombre"].str.split(" ", expand=True)
```

---

## 6. Transformar datos

### Agregar / modificar columnas
```python
df["nueva_col"] = 0                              # columna con valor fijo
df["total"] = df["precio"] * df["cantidad"]      # columna calculada
df["es_adulto"] = df["edad"] >= 18               # columna booleana

# Aplicar función a una columna
df["nombre"] = df["nombre"].apply(lambda x: x.upper())

# Aplicar función compleja
def categorizar(edad):
    if edad < 18: return "menor"
    elif edad < 65: return "adulto"
    else: return "senior"

df["categoria"] = df["edad"].apply(categorizar)

# Aplicar función a toda la fila (axis=1)
df["resumen"] = df.apply(lambda row: f"{row['nombre']} - {row['edad']}", axis=1)
```

### Renombrar y eliminar
```python
df.rename(columns={"nombre": "name", "edad": "age"}, inplace=True)
df.drop(columns=["col_innecesaria"])
df.drop(columns=["col1", "col2"])
df.drop(index=0)              # eliminar fila por índice
```

### Ordenar
```python
df.sort_values("edad")                          # ascendente
df.sort_values("edad", ascending=False)         # descendente
df.sort_values(["ciudad", "edad"])              # múltiples columnas
df.reset_index(drop=True)                       # resetear índice
```

---

## 7. Agrupar y agregar

```python
# groupby — el más poderoso
df.groupby("ciudad")["ventas"].sum()            # suma por ciudad
df.groupby("ciudad")["edad"].mean()             # promedio por ciudad
df.groupby("ciudad")["ventas"].max()            # máximo por ciudad
df.groupby("ciudad")["ventas"].count()          # conteo por ciudad

# Múltiples agregaciones a la vez
df.groupby("ciudad").agg({
    "ventas": ["sum", "mean", "max"],
    "edad": "mean"
})

# Agrupar por múltiples columnas
df.groupby(["ciudad", "categoria"])["ventas"].sum()

# pivot table — como Excel
pd.pivot_table(df,
    values="ventas",
    index="ciudad",
    columns="categoria",
    aggfunc="sum",
    fill_value=0
)
```

---

## 8. Combinar DataFrames

```python
# Merge — como JOIN en SQL
pd.merge(df1, df2, on="id")                     # inner join
pd.merge(df1, df2, on="id", how="left")         # left join
pd.merge(df1, df2, on="id", how="right")        # right join
pd.merge(df1, df2, on="id", how="outer")        # outer join
pd.merge(df1, df2, left_on="id", right_on="user_id")  # columnas distintas

# Concat — apilar DataFrames
pd.concat([df1, df2])                           # apilar filas
pd.concat([df1, df2], ignore_index=True)        # resetear índice
pd.concat([df1, df2], axis=1)                   # unir columnas
```

---

## 9. Fechas y tiempos

```python
df["fecha"] = pd.to_datetime(df["fecha"])

# Extraer partes de la fecha
df["año"] = df["fecha"].dt.year
df["mes"] = df["fecha"].dt.month
df["dia"] = df["fecha"].dt.day
df["dia_semana"] = df["fecha"].dt.day_name()   # "Monday", "Tuesday"...

# Filtrar por fecha
df[df["fecha"] > "2024-01-01"]
df[df["fecha"].between("2024-01-01", "2024-12-31")]

# Diferencia entre fechas
df["dias_transcurridos"] = (pd.Timestamp.now() - df["fecha"]).dt.days
```

---

## 10. Exportar resultados

```python
# CSV
df.to_csv("resultado.csv", index=False)
df.to_csv("resultado.csv", index=False, encoding="utf-8-sig")  # Excel compatible

# Excel
df.to_excel("resultado.xlsx", index=False)
df.to_excel("resultado.xlsx", index=False, sheet_name="Reporte")

# JSON
df.to_json("resultado.json", orient="records", indent=4)

# JSON nativo
with open("resultado.json", "w", encoding="utf-8") as f:
    json.dump(datos, f, indent=4, ensure_ascii=False)

# TXT
with open("resultado.txt", "w", encoding="utf-8") as f:
    f.write(contenido)
```

---

## 11. Flujo completo típico en hackathon

```python
import pandas as pd
import json

# 1. Cargar datos
df = pd.read_csv("datos.csv", encoding="utf-8")

# 2. Explorar
print(df.shape)
print(df.info())
print(df.isnull().sum())

# 3. Limpiar
df = df.drop_duplicates()
df = df.dropna(subset=["columna_importante"])
df["fecha"] = pd.to_datetime(df["fecha"])
df["precio"] = df["precio"].astype(float)

# 4. Transformar
df["total"] = df["precio"] * df["cantidad"]
df["categoria"] = df["tipo"].apply(lambda x: "A" if x > 100 else "B")

# 5. Analizar
resumen = df.groupby("categoria").agg({
    "total": ["sum", "mean"],
    "cantidad": "count"
}).round(2)

top_10 = df.nlargest(10, "total")

# 6. Exportar
resumen.to_csv("resumen.csv")
top_10.to_excel("top10.xlsx", index=False)
```

---

## 12. Trucos rápidos para hackathon

```python
# Top N valores
df.nlargest(10, "ventas")
df.nsmallest(5, "precio")

# Contar valores únicos rápido
df["ciudad"].value_counts()
df["ciudad"].value_counts(normalize=True)   # en porcentaje

# Correlación entre columnas numéricas
df.corr()

# Columnas numéricas solamente
df.select_dtypes(include="number")
df.select_dtypes(include="object")   # columnas de texto

# Reemplazar valores
df["estado"].replace({"activo": 1, "inactivo": 0})

# Condicional tipo if-else en columna
import numpy as np
df["descuento"] = np.where(df["total"] > 1000, 0.1, 0.0)

# Múltiples condiciones
condiciones = [
    df["total"] > 1000,
    df["total"] > 500,
    df["total"] <= 500
]
opciones = ["Premium", "Estándar", "Básico"]
df["nivel"] = np.select(condiciones, opciones)

# Aplanar JSON anidado
from pandas import json_normalize
df = json_normalize(datos, record_path="items", meta=["id", "nombre"])
```

---

## 13. Errores comunes ⚠️

```python
# ❌ SettingWithCopyWarning — modificar una copia sin querer
filtrado = df[df["edad"] > 18]
filtrado["nueva_col"] = 1   # ⚠️ puede no funcionar

# ✅ Usar .copy() explícitamente
filtrado = df[df["edad"] > 18].copy()
filtrado["nueva_col"] = 1   # ✅ funciona bien

# ❌ Encoding — error al leer archivos con acentos
df = pd.read_csv("datos.csv")  # puede fallar

# ✅ Siempre especificar encoding
df = pd.read_csv("datos.csv", encoding="utf-8")
# Si falla, probar: encoding="latin-1" o encoding="utf-8-sig"

# ❌ pop(0) en lista para simular queue — es O(n)
cola = []
cola.pop(0)  # lento

# ✅ Usar deque
from collections import deque
cola = deque()
cola.popleft()  # O(1)
```

---

## Referencia rápida

```
¿Quiero ver los datos?              → df.head() / df.info() / df.describe()
¿Quiero filtrar filas?              → df[df["col"] > valor]
¿Quiero una columna calculada?      → df["nueva"] = df["a"] * df["b"]
¿Quiero agrupar y sumar?            → df.groupby("col")["val"].sum()
¿Quiero unir dos DataFrames?        → pd.merge(df1, df2, on="id")
¿Quiero eliminar nulos?             → df.dropna()
¿Quiero los 10 mayores?             → df.nlargest(10, "col")
¿Quiero contar frecuencias?         → df["col"].value_counts()
¿Quiero cambiar tipo de dato?       → df["col"].astype(int)
¿Quiero exportar?                   → df.to_csv() / df.to_excel()
```

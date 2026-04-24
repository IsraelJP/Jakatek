# Conexión a Bases de Datos en Python 🐍🗄️

---

## PARTE 1 — PostgreSQL (SQL Relacional)

### Instalación
```bash
pip install psycopg2-binary sqlalchemy pandas
```

### Conexión básica con psycopg2
```python
import psycopg2

conn = psycopg2.connect(
    host="localhost",
    port=5432,
    database="mi_base",
    user="usuario",
    password="contraseña"
)

cursor = conn.cursor()
```

### Conexión con variables de entorno (buena práctica)
```python
import psycopg2
import os
from dotenv import load_dotenv

load_dotenv()  # carga el archivo .env

conn = psycopg2.connect(
    host=os.getenv("DB_HOST", "localhost"),
    port=os.getenv("DB_PORT", 5432),
    database=os.getenv("DB_NAME"),
    user=os.getenv("DB_USER"),
    password=os.getenv("DB_PASSWORD")
)
```

Archivo `.env`:
```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=mi_base
DB_USER=usuario
DB_PASSWORD=contraseña
```

---

### CRUD con psycopg2

#### SELECT — Leer datos
```python
cursor = conn.cursor()

# Todos los registros
cursor.execute("SELECT * FROM usuarios")
filas = cursor.fetchall()          # lista de tuplas
for fila in filas:
    print(fila)

# Con filtro
cursor.execute("SELECT * FROM usuarios WHERE edad > %s", (18,))  # ⚠️ usar %s, nunca f-string
fila = cursor.fetchone()           # un solo registro
filas = cursor.fetchmany(10)       # N registros

# Obtener nombres de columnas
columnas = [desc[0] for desc in cursor.description]
```

#### INSERT — Insertar datos
```python
cursor.execute(
    "INSERT INTO usuarios (nombre, edad, ciudad) VALUES (%s, %s, %s)",
    ("Ana", 25, "CDMX")
)
conn.commit()   # ⚠️ siempre hacer commit después de escribir

# Insertar múltiples registros
datos = [("Luis", 30, "MTY"), ("Sara", 22, "GDL")]
cursor.executemany(
    "INSERT INTO usuarios (nombre, edad, ciudad) VALUES (%s, %s, %s)",
    datos
)
conn.commit()
```

#### UPDATE — Actualizar datos
```python
cursor.execute(
    "UPDATE usuarios SET ciudad = %s WHERE nombre = %s",
    ("MTY", "Ana")
)
conn.commit()
```

#### DELETE — Eliminar datos
```python
cursor.execute("DELETE FROM usuarios WHERE edad < %s", (18,))
conn.commit()
```

---

### Conexión con SQLAlchemy + Pandas (recomendado para hackathon)
```python
from sqlalchemy import create_engine
import pandas as pd

# Crear engine
engine = create_engine("postgresql://usuario:contraseña@localhost:5432/mi_base")

# Leer tabla completa como DataFrame — muy útil
df = pd.read_sql("SELECT * FROM usuarios", engine)
df = pd.read_sql("SELECT * FROM usuarios WHERE edad > 18", engine)
df = pd.read_sql_table("usuarios", engine)    # tabla completa

# Escribir DataFrame a tabla
df.to_sql("nueva_tabla", engine, if_exists="replace", index=False)
df.to_sql("nueva_tabla", engine, if_exists="append", index=False)
# if_exists: "replace" sobreescribe, "append" agrega, "fail" lanza error
```

---

### Patrón recomendado — context manager
```python
import psycopg2
from contextlib import contextmanager

@contextmanager
def get_connection():
    conn = psycopg2.connect(
        host="localhost",
        database="mi_base",
        user="usuario",
        password="contraseña"
    )
    try:
        yield conn
        conn.commit()
    except Exception as e:
        conn.rollback()    # revertir si hay error
        raise e
    finally:
        conn.close()       # siempre cerrar

# Uso
with get_connection() as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM usuarios")
    print(cursor.fetchall())
```

---

### Crear tablas
```python
cursor.execute("""
    CREATE TABLE IF NOT EXISTS usuarios (
        id SERIAL PRIMARY KEY,
        nombre VARCHAR(100) NOT NULL,
        edad INTEGER,
        ciudad VARCHAR(100),
        email VARCHAR(200) UNIQUE,
        creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
""")
conn.commit()
```

---

### Flujo completo PostgreSQL
```python
import psycopg2
import pandas as pd
from sqlalchemy import create_engine

# 1. Conectar
engine = create_engine("postgresql://usuario:contraseña@localhost:5432/mi_base")

# 2. Leer datos
df = pd.read_sql("SELECT * FROM ventas", engine)

# 3. Procesar con pandas
resumen = df.groupby("ciudad")["total"].sum().reset_index()

# 4. Guardar resultado en nueva tabla
resumen.to_sql("resumen_ventas", engine, if_exists="replace", index=False)

print("✅ Proceso completado")
```

---

## PARTE 2 — MongoDB (NoSQL)

### Instalación
```bash
pip install pymongo pandas
```

### Conexión básica
```python
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017/")
db = client["mi_base"]                  # base de datos
coleccion = db["usuarios"]              # colección (como tabla)
```

### Conexión con URI (Atlas o remoto)
```python
from pymongo import MongoClient
import os

# MongoDB Atlas
uri = "mongodb+srv://usuario:contraseña@cluster.mongodb.net/mi_base"
client = MongoClient(uri)

# Con variable de entorno
client = MongoClient(os.getenv("MONGO_URI"))
db = client["mi_base"]
```

---

### CRUD con pymongo

#### INSERT — Insertar documentos
```python
# Un documento
usuario = {"nombre": "Ana", "edad": 25, "ciudad": "CDMX"}
resultado = coleccion.insert_one(usuario)
print(resultado.inserted_id)    # → ObjectId generado

# Múltiples documentos
usuarios = [
    {"nombre": "Luis", "edad": 30, "hobbies": ["fútbol", "música"]},
    {"nombre": "Sara", "edad": 22, "activo": True}
]
resultado = coleccion.insert_many(usuarios)
print(resultado.inserted_ids)
```

#### FIND — Leer documentos
```python
# Todos los documentos
for doc in coleccion.find():
    print(doc)

# Con filtro
coleccion.find_one({"nombre": "Ana"})               # uno solo
coleccion.find({"ciudad": "CDMX"})                  # varios
coleccion.find({"edad": {"$gt": 18}})               # edad > 18
coleccion.find({"edad": {"$gte": 18, "$lte": 65}})  # entre 18 y 65

# Seleccionar campos (proyección)
coleccion.find({}, {"nombre": 1, "edad": 1, "_id": 0})  # solo nombre y edad

# Ordenar y limitar
coleccion.find().sort("edad", -1).limit(10)         # top 10 más viejos

# Contar
coleccion.count_documents({"ciudad": "CDMX"})
coleccion.count_documents({})                       # total
```

#### UPDATE — Actualizar documentos
```python
# Actualizar uno
coleccion.update_one(
    {"nombre": "Ana"},                    # filtro
    {"$set": {"ciudad": "MTY"}}           # cambio
)

# Actualizar varios
coleccion.update_many(
    {"activo": False},
    {"$set": {"activo": True}}
)

# Otros operadores de actualización
{"$inc": {"edad": 1}}                    # incrementar
{"$push": {"hobbies": "lectura"}}        # agregar a array
{"$pull": {"hobbies": "fútbol"}}         # quitar de array
{"$unset": {"campo": ""}}               # eliminar campo
```

#### DELETE — Eliminar documentos
```python
coleccion.delete_one({"nombre": "Ana"})        # eliminar uno
coleccion.delete_many({"activo": False})       # eliminar varios
coleccion.delete_many({})                      # eliminar todos ⚠️
```

---

### Operadores de consulta más usados
```python
# Comparación
{"edad": {"$gt": 18}}      # mayor que
{"edad": {"$gte": 18}}     # mayor o igual
{"edad": {"$lt": 65}}      # menor que
{"edad": {"$lte": 65}}     # menor o igual
{"edad": {"$ne": 18}}      # diferente
{"ciudad": {"$in": ["CDMX", "MTY"]}}   # en lista
{"ciudad": {"$nin": ["CDMX"]}}         # no en lista

# Lógicos
{"$and": [{"edad": {"$gt": 18}}, {"ciudad": "CDMX"}]}
{"$or":  [{"ciudad": "CDMX"}, {"ciudad": "MTY"}]}
{"$not": {"ciudad": "CDMX"}}

# Texto
{"nombre": {"$regex": "^Ana"}}         # empieza con Ana
{"nombre": {"$regex": "ana", "$options": "i"}}  # case insensitive
```

---

### Aggregation Pipeline — el más poderoso
```python
# Equivalente a GROUP BY en SQL
pipeline = [
    {"$match": {"activo": True}},                   # WHERE
    {"$group": {
        "_id": "$ciudad",                           # GROUP BY ciudad
        "total_ventas": {"$sum": "$ventas"},        # SUM(ventas)
        "promedio_edad": {"$avg": "$edad"},         # AVG(edad)
        "conteo": {"$sum": 1}                       # COUNT(*)
    }},
    {"$sort": {"total_ventas": -1}},                # ORDER BY DESC
    {"$limit": 10}                                  # LIMIT 10
]

resultados = list(coleccion.aggregate(pipeline))
for r in resultados:
    print(r)
```

---

### MongoDB a DataFrame pandas
```python
import pandas as pd

# Convertir consulta a DataFrame
datos = list(coleccion.find({}, {"_id": 0}))   # excluir _id
df = pd.DataFrame(datos)

# Con filtro
datos = list(coleccion.find({"edad": {"$gt": 18}}, {"_id": 0}))
df = pd.DataFrame(datos)

# Aggregation a DataFrame
resultados = list(coleccion.aggregate(pipeline))
df = pd.DataFrame(resultados)
```

---

### Índices — para búsquedas rápidas
```python
# Crear índice simple
coleccion.create_index("email")

# Índice único
coleccion.create_index("email", unique=True)

# Índice compuesto
coleccion.create_index([("ciudad", 1), ("edad", -1)])  # 1=asc, -1=desc

# Ver índices existentes
coleccion.index_information()
```

---

### Flujo completo MongoDB
```python
from pymongo import MongoClient
import pandas as pd

# 1. Conectar
client = MongoClient("mongodb://localhost:27017/")
db = client["hackathon"]
coleccion = db["ventas"]

# 2. Insertar datos de prueba
ventas = [
    {"producto": "A", "ciudad": "CDMX", "total": 500},
    {"producto": "B", "ciudad": "MTY",  "total": 300},
    {"producto": "A", "ciudad": "CDMX", "total": 200},
]
coleccion.insert_many(ventas)

# 3. Aggregation — ventas por ciudad
pipeline = [
    {"$group": {"_id": "$ciudad", "total": {"$sum": "$total"}}},
    {"$sort": {"total": -1}}
]
resultados = list(coleccion.aggregate(pipeline))

# 4. Convertir a DataFrame y analizar
df = pd.DataFrame(resultados)
df.rename(columns={"_id": "ciudad"}, inplace=True)

# 5. Cerrar conexión
client.close()
print(df)
```

---

## Resumen comparativo

| | PostgreSQL | MongoDB |
|---|---|---|
| **Tipo** | Relacional (SQL) | NoSQL (documentos) |
| **Estructura** | Tablas y filas | Colecciones y documentos |
| **Esquema** | Fijo | Flexible |
| **Librería** | psycopg2 / SQLAlchemy | pymongo |
| **Consultas** | SQL estándar | JSON-like |
| **Joins** | ✅ Nativo | ⚠️ Con `$lookup` |
| **Escala horizontal** | Limitada | ✅ Muy buena |
| **Ideal para** | Datos estructurados | Datos flexibles/anidados |

---

## Referencia rápida

```
# PostgreSQL con pandas — lo más rápido
engine = create_engine("postgresql://user:pass@host:5432/db")
df = pd.read_sql("SELECT * FROM tabla", engine)
df.to_sql("nueva_tabla", engine, if_exists="replace", index=False)

# MongoDB con pandas — lo más rápido
client = MongoClient("mongodb://localhost:27017/")
df = pd.DataFrame(list(client["db"]["coleccion"].find({}, {"_id": 0})))
```

---

> 💡 **Tips para hackathon:**
> - Usa **SQLAlchemy + pandas** para PostgreSQL — leer/escribir tablas es trivial
> - Usa `if_exists="replace"` para sobreescribir tablas sin borrarlas manualmente
> - En MongoDB excluye siempre `_id` con `{"_id": 0}` para evitar problemas con pandas
> - Usa variables de entorno `.env` para no hardcodear contraseñas
> - Siempre hacer `conn.commit()` en psycopg2 después de INSERT/UPDATE/DELETE
> - Cierra la conexión con `client.close()` o usa context manager `with`

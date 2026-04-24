# Arquitectura para Hackathon 🏗️🐍

> Principio clave: **velocidad > perfección**. Archivos pequeños, un solo lugar para cambiar cosas, todo retorna DataFrame.

---

## PARTE 1 — PostgreSQL (Repository Pattern)

### Estructura de carpetas
```
proyecto/
├── .env               # variables de entorno (credenciales)
├── main.py            # punto de entrada
├── database.py        # conexión a la BD
├── models.py          # dataclasses (estructura de datos)
└── repository.py      # todo el CRUD en un solo lugar
```

---

### `.env` — credenciales
```
DATABASE_URL=postgresql://usuario:contraseña@localhost:5432/mi_base
```

---

### `database.py` — conexión
```python
from sqlalchemy import create_engine
import os
from dotenv import load_dotenv

load_dotenv()

engine = create_engine(os.getenv("DATABASE_URL"))
```

---

### `models.py` — estructura de datos
```python
from dataclasses import dataclass, field
from typing import Optional
from datetime import datetime

@dataclass
class Usuario:
    nombre: str
    edad: int
    ciudad: str
    id: Optional[int] = None
    creado_en: Optional[datetime] = None

@dataclass
class Venta:
    producto: str
    cantidad: int
    precio: float
    ciudad: str
    id: Optional[int] = None
    total: float = field(init=False)

    def __post_init__(self):
        self.total = self.cantidad * self.precio
```

---

### `repository.py` — todo el CRUD
```python
import pandas as pd
from sqlalchemy import text
from database import engine

class UsuarioRepository:

    # ── Leer ──────────────────────────────────────────
    def get_all(self) -> pd.DataFrame:
        return pd.read_sql("SELECT * FROM usuarios", engine)

    def get_by_id(self, id: int) -> pd.DataFrame:
        return pd.read_sql(
            "SELECT * FROM usuarios WHERE id = %(id)s",
            engine, params={"id": id}
        )

    def get_by_ciudad(self, ciudad: str) -> pd.DataFrame:
        return pd.read_sql(
            "SELECT * FROM usuarios WHERE ciudad = %(ciudad)s",
            engine, params={"ciudad": ciudad}
        )

    # ── Escribir ───────────────────────────────────────
    def save(self, df: pd.DataFrame):
        """Guarda un DataFrame en la tabla."""
        df.to_sql("usuarios", engine, if_exists="append", index=False)

    def replace(self, df: pd.DataFrame):
        """Sobreescribe la tabla completa."""
        df.to_sql("usuarios", engine, if_exists="replace", index=False)

    # ── Comodín ────────────────────────────────────────
    def query(self, sql: str) -> pd.DataFrame:
        """Ejecuta cualquier SELECT y retorna DataFrame."""
        return pd.read_sql(sql, engine)

    def execute(self, sql: str):
        """Ejecuta cualquier INSERT/UPDATE/DELETE."""
        with engine.connect() as conn:
            conn.execute(text(sql))
            conn.commit()


class VentaRepository:

    def get_all(self) -> pd.DataFrame:
        return pd.read_sql("SELECT * FROM ventas", engine)

    def resumen_por_ciudad(self) -> pd.DataFrame:
        return pd.read_sql("""
            SELECT ciudad,
                   SUM(total)  AS total_ventas,
                   AVG(total)  AS promedio,
                   COUNT(*)    AS cantidad
            FROM ventas
            GROUP BY ciudad
            ORDER BY total_ventas DESC
        """, engine)

    def save(self, df: pd.DataFrame):
        df.to_sql("ventas", engine, if_exists="append", index=False)

    def query(self, sql: str) -> pd.DataFrame:
        return pd.read_sql(sql, engine)
```

---

### `main.py` — punto de entrada
```python
from repository import UsuarioRepository, VentaRepository
import pandas as pd

usuario_repo = UsuarioRepository()
venta_repo   = VentaRepository()

# Leer todos los usuarios
df_usuarios = usuario_repo.get_all()
print(df_usuarios.head())

# Filtrar por ciudad
df_cdmx = usuario_repo.get_by_ciudad("CDMX")

# Guardar un DataFrame nuevo
nuevos = pd.DataFrame([
    {"nombre": "Ana", "edad": 25, "ciudad": "CDMX"},
    {"nombre": "Luis", "edad": 30, "ciudad": "MTY"},
])
usuario_repo.save(nuevos)

# Consulta personalizada
df_custom = usuario_repo.query("""
    SELECT ciudad, AVG(edad) as edad_promedio
    FROM usuarios
    GROUP BY ciudad
""")

# Resumen de ventas
df_ventas = venta_repo.resumen_por_ciudad()
print(df_ventas)
```

---

## PARTE 2 — MongoDB (Repository Pattern)

### Estructura de carpetas
```
proyecto/
├── .env               # variables de entorno
├── main.py            # punto de entrada
├── database.py        # conexión a MongoDB
├── models.py          # dataclasses con to_dict()
└── repository.py      # todo el CRUD en un solo lugar
```

---

### `.env` — credenciales
```
MONGO_URI=mongodb://localhost:27017/
MONGO_DB=hackathon
```

---

### `database.py` — conexión
```python
from pymongo import MongoClient
import os
from dotenv import load_dotenv

load_dotenv()

client = MongoClient(os.getenv("MONGO_URI", "mongodb://localhost:27017/"))
db     = client[os.getenv("MONGO_DB", "hackathon")]
```

---

### `models.py` — estructura de datos
```python
from dataclasses import dataclass, asdict, field
from typing import Optional, List

@dataclass
class Usuario:
    nombre: str
    edad: int
    ciudad: str
    hobbies: List[str] = field(default_factory=list)
    activo: bool = True
    id: Optional[str] = None

    def to_dict(self):
        """Convierte a dict limpio para MongoDB."""
        return {k: v for k, v in asdict(self).items() if v is not None and k != "id"}

@dataclass
class Venta:
    producto: str
    cantidad: int
    precio: float
    ciudad: str
    total: float = field(init=False)

    def __post_init__(self):
        self.total = self.cantidad * self.precio

    def to_dict(self):
        return asdict(self)
```

---

### `repository.py` — todo el CRUD
```python
import pandas as pd
from database import db

class UsuarioRepository:
    def __init__(self):
        self.col = db["usuarios"]

    # ── Leer ──────────────────────────────────────────
    def get_all(self) -> pd.DataFrame:
        return pd.DataFrame(list(self.col.find({}, {"_id": 0})))

    def get_by(self, filtro: dict) -> pd.DataFrame:
        """Filtro flexible: get_by({"ciudad": "CDMX"})"""
        return pd.DataFrame(list(self.col.find(filtro, {"_id": 0})))

    def get_one(self, filtro: dict) -> dict:
        return self.col.find_one(filtro, {"_id": 0})

    def count(self, filtro: dict = {}) -> int:
        return self.col.count_documents(filtro)

    # ── Escribir ───────────────────────────────────────
    def save(self, usuario):
        """Guarda un solo objeto."""
        return self.col.insert_one(usuario.to_dict())

    def save_many(self, lista: list):
        """Guarda una lista de objetos."""
        return self.col.insert_many([u.to_dict() for u in lista])

    def update(self, filtro: dict, cambios: dict):
        """update({"nombre": "Ana"}, {"$set": {"ciudad": "MTY"}})"""
        return self.col.update_many(filtro, cambios)

    def delete(self, filtro: dict):
        return self.col.delete_many(filtro)

    # ── Comodín ────────────────────────────────────────
    def aggregate(self, pipeline: list) -> pd.DataFrame:
        """Ejecuta un pipeline y retorna DataFrame."""
        return pd.DataFrame(list(self.col.aggregate(pipeline)))


class VentaRepository:
    def __init__(self):
        self.col = db["ventas"]

    def get_all(self) -> pd.DataFrame:
        return pd.DataFrame(list(self.col.find({}, {"_id": 0})))

    def save_many(self, lista: list):
        return self.col.insert_many([v.to_dict() for v in lista])

    def resumen_por_ciudad(self) -> pd.DataFrame:
        pipeline = [
            {"$group": {
                "_id": "$ciudad",
                "total_ventas": {"$sum": "$total"},
                "promedio":     {"$avg": "$total"},
                "cantidad":     {"$sum": 1}
            }},
            {"$sort": {"total_ventas": -1}},
            {"$project": {
                "ciudad": "$_id",
                "total_ventas": 1,
                "promedio": 1,
                "cantidad": 1,
                "_id": 0
            }}
        ]
        return self.aggregate(pipeline)

    def aggregate(self, pipeline: list) -> pd.DataFrame:
        return pd.DataFrame(list(self.col.aggregate(pipeline)))
```

---

### `main.py` — punto de entrada
```python
from repository import UsuarioRepository, VentaRepository
from models import Usuario, Venta

usuario_repo = UsuarioRepository()
venta_repo   = VentaRepository()

# Guardar usuarios
usuario_repo.save_many([
    Usuario("Ana",  25, "CDMX", hobbies=["música"]),
    Usuario("Luis", 30, "MTY"),
    Usuario("Sara", 22, "CDMX", hobbies=["lectura", "fútbol"]),
])

# Leer todos como DataFrame
df = usuario_repo.get_all()
print(df.head())

# Filtrar por ciudad
df_cdmx = usuario_repo.get_by({"ciudad": "CDMX"})

# Filtros avanzados
df_jovenes = usuario_repo.get_by({"edad": {"$lt": 25}})

# Actualizar
usuario_repo.update({"nombre": "Ana"}, {"$set": {"ciudad": "GDL"}})

# Resumen de ventas con aggregation
df_ventas = venta_repo.resumen_por_ciudad()
print(df_ventas)

# Pipeline personalizado
pipeline = [
    {"$match": {"activo": True}},
    {"$group": {"_id": "$ciudad", "total": {"$sum": 1}}},
    {"$sort": {"total": -1}}
]
df_custom = usuario_repo.aggregate(pipeline)
```

---

## Comparativa de arquitecturas

| | PostgreSQL | MongoDB |
|---|---|---|
| **Conexión** | SQLAlchemy engine | MongoClient |
| **Leer** | `pd.read_sql()` | `pd.DataFrame(col.find())` |
| **Escribir** | `df.to_sql()` | `col.insert_many()` |
| **Consulta flexible** | `.query(sql)` | `.get_by(filtro)` |
| **Agrupaciones** | SQL GROUP BY | Aggregation Pipeline |
| **Retorna** | DataFrame | DataFrame |

---

> 💡 **Tips para hackathon:**
> - Ambas arquitecturas **siempre retornan DataFrames** — conecta directo con pandas y tus gráficos
> - Empieza por `database.py` y `models.py` — son los más rápidos de escribir
> - Usa `.query()` y `.aggregate()` como comodín para no perder tiempo creando métodos nuevos
> - Guarda las credenciales en `.env` desde el inicio — evita errores de seguridad
> - Si el tiempo aprieta, trabaja todo desde `main.py` y refactoriza después

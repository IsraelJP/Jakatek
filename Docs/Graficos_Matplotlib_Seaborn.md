# Gráficos con Matplotlib y Seaborn 🐍📊

---

## 1. Instalación e importación

```python
pip install matplotlib seaborn

import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
import numpy as np
```

---

## 2. Estructura base de cualquier gráfico

```python
# Siempre seguir este flujo:
plt.figure(figsize=(10, 6))   # 1. Tamaño del gráfico
# ... código del gráfico ...  # 2. Tipo de gráfico
plt.title("Mi Gráfico")       # 3. Título
plt.xlabel("Eje X")           # 4. Etiqueta eje X
plt.ylabel("Eje Y")           # 5. Etiqueta eje Y
plt.legend()                  # 6. Leyenda (si hay varias series)
plt.tight_layout()            # 7. Ajustar márgenes automáticamente
plt.savefig("grafico.png")    # 8. Guardar (opcional)
plt.show()                    # 9. Mostrar
```

---

## 3. Gráficos con Matplotlib

### Líneas — tendencias en el tiempo
```python
plt.figure(figsize=(10, 6))
plt.plot(df["fecha"], df["ventas"], color="blue", linewidth=2, label="Ventas")
plt.plot(df["fecha"], df["gastos"], color="red", linestyle="--", label="Gastos")
plt.title("Ventas vs Gastos")
plt.xlabel("Fecha")
plt.ylabel("Monto")
plt.legend()
plt.tight_layout()
plt.show()
```

### Barras verticales
```python
plt.figure(figsize=(10, 6))
plt.bar(df["ciudad"], df["ventas"], color="steelblue", edgecolor="black")
plt.title("Ventas por Ciudad")
plt.xlabel("Ciudad")
plt.ylabel("Ventas")
plt.xticks(rotation=45)       # rotar etiquetas si son largas
plt.tight_layout()
plt.show()
```

### Barras horizontales
```python
plt.figure(figsize=(10, 6))
plt.barh(df["ciudad"], df["ventas"], color="steelblue")
plt.title("Ventas por Ciudad")
plt.tight_layout()
plt.show()
```

### Dispersión (Scatter)
```python
plt.figure(figsize=(8, 6))
plt.scatter(df["precio"], df["ventas"], alpha=0.6, color="green", s=50)
plt.title("Precio vs Ventas")
plt.xlabel("Precio")
plt.ylabel("Ventas")
plt.show()
```

### Histograma
```python
plt.figure(figsize=(8, 6))
plt.hist(df["edad"], bins=20, color="steelblue", edgecolor="black")
plt.title("Distribución de Edades")
plt.xlabel("Edad")
plt.ylabel("Frecuencia")
plt.show()
```

### Pastel (Pie)
```python
plt.figure(figsize=(8, 8))
plt.pie(
    df["ventas"],
    labels=df["ciudad"],
    autopct="%1.1f%%",          # mostrar porcentajes
    startangle=90,
    colors=["steelblue", "orange", "green", "red"]
)
plt.title("Distribución de Ventas")
plt.show()
```

### Múltiples gráficos en una figura
```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))   # 2 filas, 2 columnas

axes[0, 0].plot(df["fecha"], df["ventas"])
axes[0, 0].set_title("Ventas en el tiempo")

axes[0, 1].bar(df["ciudad"], df["ventas"])
axes[0, 1].set_title("Ventas por ciudad")

axes[1, 0].hist(df["edad"], bins=20)
axes[1, 0].set_title("Distribución de edades")

axes[1, 1].scatter(df["precio"], df["ventas"])
axes[1, 1].set_title("Precio vs Ventas")

plt.tight_layout()
plt.show()
```

---

## 4. Gráficos con Seaborn

> Seaborn trabaja directamente con DataFrames y genera gráficos más bonitos con menos código.

### Barras
```python
plt.figure(figsize=(10, 6))
sns.barplot(data=df, x="ciudad", y="ventas", hue="categoria", palette="Blues_d")
plt.title("Ventas por Ciudad y Categoría")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### Líneas
```python
plt.figure(figsize=(10, 6))
sns.lineplot(data=df, x="fecha", y="ventas", hue="categoria", linewidth=2)
plt.title("Ventas en el tiempo")
plt.tight_layout()
plt.show()
```

### Dispersión
```python
plt.figure(figsize=(8, 6))
sns.scatterplot(data=df, x="precio", y="ventas", hue="categoria", size="cantidad")
plt.title("Precio vs Ventas")
plt.show()
```

### Histograma + curva de densidad
```python
plt.figure(figsize=(8, 6))
sns.histplot(data=df, x="edad", bins=20, kde=True, color="steelblue")
plt.title("Distribución de Edades")
plt.show()
```

### Boxplot — detectar outliers
```python
plt.figure(figsize=(10, 6))
sns.boxplot(data=df, x="categoria", y="ventas", palette="Set2")
plt.title("Distribución de Ventas por Categoría")
plt.show()
```

### Violinplot — como boxplot pero más detallado
```python
plt.figure(figsize=(10, 6))
sns.violinplot(data=df, x="categoria", y="ventas", palette="muted")
plt.title("Distribución de Ventas")
plt.show()
```

### Heatmap de correlación — el más impresionante 🔥
```python
plt.figure(figsize=(10, 8))
sns.heatmap(
    df.corr(),
    annot=True,           # mostrar valores
    fmt=".2f",            # 2 decimales
    cmap="coolwarm",      # colores rojo-azul
    vmin=-1, vmax=1,      # rango fijo
    linewidths=0.5
)
plt.title("Correlación entre Variables")
plt.tight_layout()
plt.show()
```

### Countplot — contar categorías
```python
plt.figure(figsize=(10, 6))
sns.countplot(data=df, x="ciudad", hue="categoria", palette="Set1")
plt.title("Conteo por Ciudad")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### Pairplot — relación entre todas las columnas numéricas
```python
# Muy útil para exploración rápida
sns.pairplot(df, hue="categoria", diag_kind="kde")
plt.show()
```

### Regresión lineal (con línea de tendencia)
```python
plt.figure(figsize=(8, 6))
sns.regplot(data=df, x="precio", y="ventas", color="steelblue")
plt.title("Tendencia Precio vs Ventas")
plt.show()
```

---

## 5. Personalización

### Colores
```python
# Paletas de Seaborn
palette="Blues"        # tonos azules
palette="Reds"         # tonos rojos
palette="coolwarm"     # azul a rojo (correlación)
palette="Set1"         # colores vivos
palette="Set2"         # colores suaves
palette="muted"        # colores apagados
palette="viridis"      # verde a amarillo
palette="rocket"       # morado a naranja

# Colores individuales en Matplotlib
color="steelblue"
color="#FF5733"        # hex
color=(0.2, 0.4, 0.6)  # RGB
```

### Estilo global de Seaborn
```python
sns.set_theme(style="whitegrid")   # fondo blanco con cuadrícula
sns.set_theme(style="darkgrid")    # fondo oscuro con cuadrícula
sns.set_theme(style="white")       # fondo blanco limpio
sns.set_theme(style="ticks")       # minimalista

# Tamaño de fuente global
sns.set_theme(font_scale=1.3)
```

### Guardar en alta calidad
```python
plt.savefig("grafico.png", dpi=300, bbox_inches="tight")
plt.savefig("grafico.pdf", bbox_inches="tight")   # vectorial
```

---

## 6. Flujo completo típico en hackathon

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Configuración global
sns.set_theme(style="whitegrid", font_scale=1.2)

# Cargar datos
df = pd.read_csv("datos.csv")

# Figura con múltiples gráficos
fig, axes = plt.subplots(2, 2, figsize=(14, 10))
fig.suptitle("Dashboard de Ventas", fontsize=16, fontweight="bold")

# Gráfico 1 — Ventas por ciudad
sns.barplot(data=df, x="ciudad", y="ventas", ax=axes[0, 0], palette="Blues_d")
axes[0, 0].set_title("Ventas por Ciudad")
axes[0, 0].tick_params(axis="x", rotation=45)

# Gráfico 2 — Distribución de precios
sns.histplot(data=df, x="precio", bins=20, kde=True, ax=axes[0, 1], color="steelblue")
axes[0, 1].set_title("Distribución de Precios")

# Gráfico 3 — Correlación
sns.heatmap(df.corr(), annot=True, fmt=".2f", cmap="coolwarm", ax=axes[1, 0])
axes[1, 0].set_title("Correlación")

# Gráfico 4 — Ventas por categoría
sns.boxplot(data=df, x="categoria", y="ventas", ax=axes[1, 1], palette="Set2")
axes[1, 1].set_title("Ventas por Categoría")

plt.tight_layout()
plt.savefig("dashboard.png", dpi=300, bbox_inches="tight")
plt.show()
```

---

## 7. Referencia rápida

```
¿Tendencia en el tiempo?          → plt.plot() / sns.lineplot()
¿Comparar categorías?             → sns.barplot() / sns.countplot()
¿Distribución de valores?         → sns.histplot() / sns.boxplot()
¿Relación entre dos variables?    → sns.scatterplot() / sns.regplot()
¿Correlación entre columnas?      → sns.heatmap(df.corr())
¿Exploración rápida general?      → sns.pairplot()
¿Proporciones?                    → plt.pie()
¿Detectar outliers?               → sns.boxplot() / sns.violinplot()
¿Varios gráficos juntos?          → plt.subplots(filas, columnas)
```

---

## 8. Errores comunes ⚠️

```python
# ❌ Olvidar plt.show() — el gráfico no aparece
sns.barplot(data=df, x="ciudad", y="ventas")
# sin plt.show()

# ✅ Siempre cerrar con plt.show()
sns.barplot(data=df, x="ciudad", y="ventas")
plt.show()

# ❌ Gráficos se solapan entre sí
plt.figure()
plt.plot(...)
plt.figure()   # nueva figura pero la anterior queda abierta

# ✅ Cerrar figura anterior
plt.close("all")   # cierra todas las figuras abiertas

# ❌ Etiquetas del eje X cortadas
sns.barplot(data=df, x="ciudad_nombre_largo", y="ventas")
plt.show()   # texto cortado

# ✅ Rotar etiquetas y ajustar márgenes
plt.xticks(rotation=45, ha="right")
plt.tight_layout()
plt.show()

# ❌ Heatmap con columnas no numéricas
sns.heatmap(df.corr())   # falla si hay columnas de texto

# ✅ Seleccionar solo numéricas
sns.heatmap(df.select_dtypes(include="number").corr(), annot=True)
```

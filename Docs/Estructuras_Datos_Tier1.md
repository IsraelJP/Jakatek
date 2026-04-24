# Estructuras de Datos Tier 1 en Python 🐍

---

## 1. Array / Lista

### ¿Qué es?
Colección ordenada de elementos accesibles por índice. En Python las listas son dinámicas — no necesitas definir el tamaño.

### Operaciones esenciales
```python
lista = [10, 20, 30, 40]

# Acceso
lista[0]          # → 10  | O(1)
lista[-1]         # → 40  | O(1) — último elemento

# Insertar
lista.append(50)          # O(1) — al final
lista.insert(2, 99)       # O(n) — en posición específica

# Eliminar
lista.pop()               # O(1) — elimina el último
lista.pop(2)              # O(n) — elimina en posición
lista.remove(20)          # O(n) — elimina por valor

# Buscar
20 in lista               # O(n)
lista.index(20)           # O(n) — retorna posición

# Ordenar
lista.sort()              # O(n log n) — modifica la lista
sorted(lista)             # O(n log n) — retorna nueva lista

# Slicing
lista[1:3]        # → [20, 30] — sublista
lista[::-1]       # → lista al revés
```

### Usos ideales ✅
- Colecciones ordenadas donde importa la posición
- Iterar sobre todos los elementos
- Acceso frecuente por índice
- Guardar historial o secuencias

### Limitaciones ❌
- Buscar un elemento es lento: O(n)
- Insertar/eliminar en medio es lento: O(n) — desplaza todo
- No garantiza unicidad — puede tener duplicados

### Patrones comunes en hackathons
```python
# Contar frecuencias con lista de contadores
conteo = [0] * 26   # para letras del abecedario
conteo[ord('a') - ord('a')] += 1

# Two pointers — muy común en problemas de arrays
izq, der = 0, len(lista) - 1
while izq < der:
    # lógica aquí
    izq += 1
    der -= 1

# Sliding window
ventana = sum(lista[:3])
for i in range(3, len(lista)):
    ventana += lista[i] - lista[i - 3]
```

### Complejidades
| Operación | Complejidad |
|---|---|
| Acceso por índice | O(1) |
| Append al final | O(1) amortizado |
| Insert en medio | O(n) |
| Remove por valor | O(n) |
| Búsqueda | O(n) |
| Sort | O(n log n) |

---

## 2. HashMap / Diccionario

### ¿Qué es?
Colección de pares clave-valor. La clave es única y permite acceso en O(1). Es la estructura más versátil para hackathons.

### Operaciones esenciales
```python
mapa = {"nombre": "Ana", "edad": 25}

# Acceso
mapa["nombre"]            # → "Ana"     | O(1) — lanza KeyError si no existe
mapa.get("nombre")        # → "Ana"     | O(1) — retorna None si no existe
mapa.get("altura", 170)   # → 170       | O(1) — default si no existe

# Insertar / Actualizar
mapa["ciudad"] = "CDMX"   # O(1)

# Eliminar
del mapa["edad"]           # O(1) — lanza KeyError si no existe
mapa.pop("edad", None)     # O(1) — seguro, no lanza error

# Verificar existencia
"nombre" in mapa           # O(1)

# Iterar
mapa.keys()                # todas las claves
mapa.values()              # todos los valores
mapa.items()               # pares (clave, valor)

for clave, valor in mapa.items():
    print(clave, valor)
```

### Usos ideales ✅
- Contar frecuencias de elementos
- Cachear resultados (memoización)
- Agrupar elementos por categoría
- Cualquier búsqueda que deba ser O(1)
- Representar grafos (lista de adyacencia)

### Limitaciones ❌
- No mantiene orden de inserción... bueno, desde Python 3.7 sí lo mantiene, pero no ordena por valor
- Las claves deben ser inmutables (str, int, tuple — no listas)
- Más memoria que una lista simple

### Patrones comunes en hackathons
```python
from collections import defaultdict, Counter

# Contar frecuencias — el uso más común
texto = "hackathon"
frecuencia = {}
for letra in texto:
    frecuencia[letra] = frecuencia.get(letra, 0) + 1

# Más limpio con Counter
frecuencia = Counter("hackathon")
frecuencia.most_common(3)   # → las 3 letras más frecuentes

# defaultdict — evita KeyError al inicializar
grupos = defaultdict(list)
for nombre, equipo in datos:
    grupos[equipo].append(nombre)

# Memoización con dict
cache = {}
def fibonacci(n):
    if n in cache:
        return cache[n]
    if n <= 1:
        return n
    cache[n] = fibonacci(n-1) + fibonacci(n-2)
    return cache[n]

# Invertir un diccionario
invertido = {v: k for k, v in mapa.items()}
```

### Complejidades
| Operación | Complejidad |
|---|---|
| Acceso por clave | O(1) |
| Insertar | O(1) |
| Eliminar | O(1) |
| Búsqueda de clave | O(1) |
| Iterar | O(n) |

---

## 3. Stack (Pila)

### ¿Qué es?
Estructura **LIFO** (Last In, First Out) — el último en entrar es el primero en salir. En Python se implementa con una lista normal.

```
 push →  [ 1 | 2 | 3 ]  ← pop
```

### Operaciones esenciales
```python
stack = []

# Push — agregar al tope
stack.append(10)    # O(1)
stack.append(20)
stack.append(30)    # stack = [10, 20, 30]

# Pop — sacar del tope
stack.pop()         # → 30  | O(1)

# Peek — ver el tope sin sacar
stack[-1]           # → 20  | O(1)

# Verificar si está vacío
len(stack) == 0
not stack           # más pythónico
```

### Usos ideales ✅
- Validar paréntesis balanceados `()[]{}`
- Deshacer/rehacer acciones (Ctrl+Z)
- Recorrido DFS (Depth First Search)
- Evaluar expresiones matemáticas
- Navegación hacia atrás (historial del browser)
- Llamadas recursivas (call stack)

### Limitaciones ❌
- Solo acceso eficiente al tope
- Buscar un elemento en medio es O(n)
- No sirve para procesar en orden de llegada

### Patrones comunes en hackathons
```python
# Validar paréntesis — problema clásico
def parentesis_validos(s: str) -> bool:
    stack = []
    pares = {')': '(', ']': '[', '}': '{'}

    for char in s:
        if char in "([{":
            stack.append(char)
        elif char in ")]}":
            if not stack or stack[-1] != pares[char]:
                return False
            stack.pop()

    return len(stack) == 0

# DFS iterativo con stack
def dfs(grafo, inicio):
    stack = [inicio]
    visitados = set()

    while stack:
        nodo = stack.pop()
        if nodo not in visitados:
            visitados.add(nodo)
            for vecino in grafo[nodo]:
                stack.append(vecino)
```

### Complejidades
| Operación | Complejidad |
|---|---|
| Push (append) | O(1) |
| Pop | O(1) |
| Peek (tope) | O(1) |
| Búsqueda | O(n) |

---

## 4. Queue (Cola)

### ¿Qué es?
Estructura **FIFO** (First In, First Out) — el primero en entrar es el primero en salir. Usa `deque` de `collections`, no una lista (pop(0) en lista es O(n)).

```
enqueue →  [ 3 | 2 | 1 ]  → dequeue
```

### Operaciones esenciales
```python
from collections import deque

queue = deque()

# Enqueue — agregar al final
queue.append(10)     # O(1)
queue.append(20)
queue.append(30)     # queue = deque([10, 20, 30])

# Dequeue — sacar del frente
queue.popleft()      # → 10  | O(1)

# Peek — ver el frente sin sacar
queue[0]             # → 20  | O(1)

# Verificar si está vacía
not queue

# deque también funciona como stack doble
queue.appendleft(5)  # agregar al frente
queue.pop()          # sacar del final
```

### Usos ideales ✅
- Recorrido BFS (Breadth First Search)
- Procesar tareas en orden de llegada
- Simular filas (bancos, impresoras)
- Notificaciones o eventos en orden
- Caché LRU (con `maxlen`)

### Limitaciones ❌
- Solo acceso eficiente al frente y al final
- Buscar en medio es O(n)
- No sirve para priorizar elementos (usa Heap para eso)

### Patrones comunes en hackathons
```python
from collections import deque

# BFS — encontrar camino más corto
def bfs(grafo, inicio, fin):
    queue = deque([[inicio]])
    visitados = {inicio}

    while queue:
        camino = queue.popleft()
        nodo = camino[-1]

        if nodo == fin:
            return camino

        for vecino in grafo[nodo]:
            if vecino not in visitados:
                visitados.add(vecino)
                queue.append(camino + [vecino])

    return None

# Cola con tamaño máximo — útil para ventana deslizante
ultimos_5 = deque(maxlen=5)
for numero in range(10):
    ultimos_5.append(numero)   # automáticamente descarta los viejos
# → deque([5, 6, 7, 8, 9])
```

### Complejidades
| Operación | Complejidad |
|---|---|
| Append (enqueue) | O(1) |
| Popleft (dequeue) | O(1) — ⚠️ usar deque, no lista |
| Peek (frente) | O(1) |
| Búsqueda | O(n) |

---

## Resumen comparativo

| | Lista | Dict | Stack | Queue |
|---|---|---|---|---|
| **Orden** | ✅ por índice | ✅ inserción | ✅ LIFO | ✅ FIFO |
| **Acceso** | O(1) por índice | O(1) por clave | O(1) tope | O(1) frente |
| **Búsqueda** | O(n) | O(1) | O(n) | O(n) |
| **Insertar** | O(1) final | O(1) | O(1) | O(1) |
| **Eliminar** | O(n) medio | O(1) | O(1) | O(1) |
| **Uso principal** | Secuencias | Clave-valor | LIFO / DFS | FIFO / BFS |

---

## ¿Cuál usar según el problema?

```
¿Necesito buscar rápido?           → Dict
¿Necesito orden de llegada?        → Queue
¿Necesito el último procesado?     → Stack
¿Necesito acceso por posición?     → Lista
¿Necesito contar frecuencias?      → Dict / Counter
¿Necesito el camino más corto?     → Queue + BFS
¿Necesito explorar en profundidad? → Stack + DFS
```

---

> 💡 **Tips para hackathon:**
> - `Counter` y `defaultdict` de `collections` te ahorran muchísimo código
> - Nunca uses `lista.pop(0)` — usa `deque.popleft()` para queues
> - Un dict puede resolver el 60% de los problemas de optimización
> - Stack + Queue son la base de DFS y BFS respectivamente

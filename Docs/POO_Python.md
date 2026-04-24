# POO en Python con @dataclass 🐍

## 1. Clases y Objetos básicos

```python
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class Persona:
    nombre: str                                   # obligatorio
    edad: int = 18                                # opcional, default 18
    ciudad: Optional[str] = None                  # opcional, default None
    hobbies: list = field(default_factory=list)   # listas/dicts siempre así
```

---

## 2. Los 4 Pilares

### Encapsulamiento
Ocultar datos internos usando `_` (convención) o `__` (privado real).

```python
@dataclass
class CuentaBancaria:
    _saldo: float = field(default=0, repr=False)  # oculto en print()

    def depositar(self, monto: float):
        self._saldo += monto

    def get_saldo(self):
        return self._saldo
```

### Herencia
```python
@dataclass
class Animal:
    nombre: str
    edad: int

    def hablar(self):
        return "..."

@dataclass
class Perro(Animal):
    raza: str = "Desconocida"          # ⚠️ siempre con default en hijos

    def hablar(self):                  # sobreescribe el método
        return f"¡Guau! Soy {self.nombre}"

@dataclass
class PerroPolicia(Perro):             # herencia multinivel
    badge: int = 0

    def hablar(self):
        return f"{super().hablar()} 🚔 Badge: {self.badge}"
```

> ⚠️ **Regla:** Si el padre tiene campos con default, los hijos también deben tenerlos.

### Polimorfismo
```python
animales = [Perro("Rex", 3), PerroPolicia("Thor", 5, badge=42)]

for a in animales:
    print(a.hablar())   # cada uno responde diferente
```

### Abstracción (Interfaces)
```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def hablar(self) -> str: pass     # obligatorio implementar

    def respirar(self):               # opcional sobreescribir
        return "Inhala... exhala..."

@dataclass
class Gato(Animal):
    nombre: str

    def hablar(self) -> str:
        return f"Miau, soy {self.nombre}"
```

---

## 3. Características de @dataclass

### `__post_init__` — lógica post-constructor
```python
@dataclass
class Persona:
    nombre: str
    edad: int
    es_adulto: bool = field(init=False)  # no se pasa al crear

    def __post_init__(self):
        self.es_adulto = self.edad >= 18  # se calcula automáticamente
```

### `frozen=True` — objeto inmutable
```python
@dataclass(frozen=True)
class Punto:
    x: float
    y: float

p = Punto(1.0, 2.0)
p.x = 5  # ❌ Error — no se puede modificar
```

### `order=True` — comparar y ordenar
```python
@dataclass(order=True)
class Producto:
    precio: float
    nombre: str

productos = [Producto(30, "Camisa"), Producto(10, "Calcetines")]
sorted(productos)  # ordena por precio automáticamente
```

### `field()` opciones
```python
@dataclass
class Usuario:
    nombre: str
    password: str = field(repr=False)           # oculto en print()
    hobbies: list = field(default_factory=list) # lista nueva por instancia
    score: int = field(init=False, default=0)   # no se pasa al crear
```

---

## 4. Constructores alternativos con `@classmethod`
```python
@dataclass
class Persona:
    nombre: str
    edad: int

    @classmethod
    def desde_string(cls, texto: str):   # "Ana,25"
        nombre, edad = texto.split(",")
        return cls(nombre, int(edad))

    @classmethod
    def anonima(cls):
        return cls("Anónimo", 0)

p1 = Persona("Ana", 25)
p2 = Persona.desde_string("Ana,25")
p3 = Persona.anonima()
```

---

## 5. Múltiples Interfaces
```python
from abc import ABC, abstractmethod

class Nadador(ABC):
    @abstractmethod
    def nadar(self): pass

class Volador(ABC):
    @abstractmethod
    def volar(self): pass

@dataclass
class Pato(Nadador, Volador):      # implementa "ambas interfaces"
    nombre: str

    def nadar(self): return f"{self.nombre} nada 🏊"
    def volar(self): return f"{self.nombre} vuela 🦆"
```

---

## 6. Tabla resumen: Java vs Python

| Java | Python |
|---|---|
| `class` | `@dataclass class` |
| `interface` | `ABC` con solo `@abstractmethod` |
| `abstract class` | `ABC` con métodos concretos y abstractos |
| `implements` | heredar de la clase ABC |
| `extends` | heredar normal |
| `@Getter @Setter` | `@property` |
| `final class` | `@dataclass(frozen=True)` |
| Constructor sobrecargado | `@classmethod` alternativo |

---

## 7. Complejidades clave

| Estructura | Acceso | Búsqueda | Insertar | Eliminar |
|---|---|---|---|---|
| Lista/Array | O(1) | O(n) | O(1)* | O(n) |
| Dict/HashMap | O(1) | O(1) | O(1) | O(1) |
| Stack | O(n) | O(n) | O(1) | O(1) |
| Queue | O(n) | O(n) | O(1) | O(1) |

---

> 💡 **Tips para hackathon:**
> - Usa `dict` para búsquedas rápidas
> - Usa `@dataclass(frozen=True)` para objetos que no deben cambiar
> - Usa `__post_init__` para calcular campos derivados
> - Usa `@classmethod` en lugar de sobrecarga de constructores

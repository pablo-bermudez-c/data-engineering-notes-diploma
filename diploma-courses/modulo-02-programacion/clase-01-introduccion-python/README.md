# Módulo 2 — Introducción a la Programación y al Preprocesamiento de Datos

---

## Tabla de contenidos

1. [¿Qué es Python y por qué usarlo en Data Engineering?](#bloque-1)
2. [Fundamentos del lenguaje: print, comentarios y variables](#bloque-2)
3. [Tipos de datos y operadores](#bloque-3)
4. [Strings: operaciones y métodos](#bloque-4)
5. [Conversión de tipos e input()](#bloque-5)
6. [Estructuras de datos](#bloque-6)

---

## Bloque 1 — ¿Qué es Python y por qué usarlo en Data Engineering? {#bloque-1}

### ¿Qué es Python?

Python es un **lenguaje de programación interpretado, dinámico y de alto nivel**, creado en 1991 por Guido van Rossum. Está diseñado para ser simple y legible, lo que permite enfocarse en la lógica del problema en lugar de en la complejidad del lenguaje.

```
Lenguaje Interpretado vs Compilado
────────────────────────────────────────────────────────
  COMPILADO (ej. C++)            INTERPRETADO (Python)
  ┌──────────────┐               ┌──────────────┐
  │ Código fuente│               │ Código fuente│
  └──────┬───────┘               └──────┬───────┘
         │ compilar                     │ ejecutar
         ▼                              ▼ línea a línea
  ┌──────────────┐               ┌──────────────┐
  │  Ejecutable  │               │  Intérprete  │
  │  (binario)   │               │   Python     │
  └──────────────┘               └──────────────┘
  Más rápido en ejecución        Más fácil de depurar
```

### Características principales

| Característica | Descripción |
|---|---|
| **Interpretado** | No necesita compilación previa; se ejecuta línea a línea |
| **Tipado dinámico** | El tipo de variable se infiere en tiempo de ejecución, no al declarar |
| **Alto nivel** | Sintaxis cercana al lenguaje natural; abstrae complejidades del hardware |
| **Multiparadigma** | Soporta programación orientada a objetos (POO) y funcional |
| **Open source** | Gratuito, con comunidad global de más de 450.000 paquetes en PyPI |
| **Multiplataforma** | Funciona en Windows, Mac, Linux y servicios cloud |

### ¿Por qué Python en Data Engineering?

Python es el estándar de facto en el ecosistema de datos. Según el índice TIOBE (Mar 2026), Python lidera con un 21.25% de popularidad entre todos los lenguajes de programación.

```
Stack Python para Data Engineering
────────────────────────────────────────────────────────────
┌─────────────────────────────────────────────────────────┐
│                  CAPAS DEL STACK DE DATOS               │
├──────────────────┬──────────────────────────────────────┤
│  Orquestación    │  Apache Airflow, Luigi               │
├──────────────────┼──────────────────────────────────────┤
│  Big Data        │  PySpark, Hadoop                     │
├──────────────────┼──────────────────────────────────────┤
│  Bases de datos  │  SQLAlchemy, PyODBC                  │
├──────────────────┼──────────────────────────────────────┤
│  Procesamiento   │  Pandas, NumPy                       │
├──────────────────┼──────────────────────────────────────┤
│  Archivos        │  CSV, JSON, Parquet, Avro            │
├──────────────────┼──────────────────────────────────────┤
│  PYTHON CORE     │  Base de todo lo anterior            │
└──────────────────┴──────────────────────────────────────┘
```

### Rol del Data Engineer y Python

El Data Engineer usa Python para:
- **Diseñar y operar pipelines de datos** (ETL/ELT)
- **Garantizar calidad y disponibilidad** de los datos
- **Construir plataformas escalables** que integren múltiples fuentes
- **Habilitar a otros equipos** (analistas, científicos de datos)

> 💡 **Contexto DE:** En la práctica, un pipeline de datos en producción combina Python + SQL + servicios cloud. Python es el pegamento que une todas las piezas.

### Entorno recomendado

| Herramienta | Uso |
|---|---|
| `python.org` / Anaconda | Instalación base de Python |
| **Jupyter Notebook / JupyterLab** | Entorno interactivo (usado en este curso) |
| VS Code | Editor para scripts `.py` de producción |
| PyCharm | IDE completo para proyectos grandes |
| `venv` / `conda` | Gestión de entornos virtuales aislados |

> ⚠️ **Buena práctica:** Siempre usar **entornos virtuales** para aislar dependencias entre proyectos. Un pipeline que funciona en tu máquina debe funcionar igual en el servidor de producción.

```bash
# Verificar versión de Python instalada
python --version

# Desde Python (útil en Jupyter)
import sys
print(sys.version)
# Output: 3.12.9 | packaged by Anaconda, Inc. | ...
```

---

## Bloque 2 — Fundamentos: print(), comentarios y variables {#bloque-2}

### La función `print()`

`print()` muestra información en la consola. Es la función más básica y uno de los primeros pasos al aprender cualquier lenguaje.

**Sintaxis:**
```python
print(objeto1, objeto2, ..., sep=' ', end='\n')
```

| Parámetro | Descripción | Valor por defecto |
|---|---|---|
| `sep` | Separador entre objetos | `' '` (espacio) |
| `end` | Carácter al finalizar la línea | `'\n'` (salto de línea) |

```python
# Uso básico
print("Hola mundo")                              # Hola mundo

# Múltiples elementos
print("Python", "es", "genial")                  # Python es genial

# Separador personalizado
print("Python", "es", "genial", sep="-")         # Python-es-genial

# Evitar salto de línea al final
print("Primera línea", end=" | ")
print("Segunda línea")                           # Primera línea | Segunda línea
```

#### Formas de imprimir variables

```python
nombre = "Diego"
edad = 14

# Método 1: concatenación directa (básico)
print("Mi nombre es", nombre, "y tengo", edad, "años")

# Método 2: concatenación con + (requiere convertir a str)
print("Hola, me llamo " + nombre + " y tengo " + str(edad) + " años")

# Método 3: .format() (Python 2/3 compatible)
print("Hola, me llamo {} y tengo {} años".format(nombre, edad))

# Método 4: f-string (recomendado desde Python 3.6+)
print(f"Hola, me llamo {nombre} y tengo {edad} años")
```

> 💡 **Buena práctica DE:** En código de producción (scripts de pipeline, logs de Airflow, etc.) siempre usa **f-strings** — son más legibles y menos propensas a errores que la concatenación con `+`.

```python
# Dentro de {} se pueden incluir expresiones completas
x = 5
y = 3
print(f"{x} + {y} = {x + y}")     # 5 + 3 = 8
print(f"Precio con IVA: {100 * 1.19:.2f}")  # Precio con IVA: 119.00
```

---

### Comentarios en Python

Los comentarios documentan el código y son ignorados por el intérprete. Son esenciales en proyectos de Data Engineering donde el código es mantenido por equipos.

```python
# Comentario de una línea: todo lo que sigue al # es ignorado

print('Hola, Python!')  # También funciona al final de una línea de código

"""
Comentario multilínea (docstring):
Útil para documentar funciones, clases o bloques complejos.
Python lo trata como un string sin asignar → lo ignora.
"""
print("hola")
```

> ⚠️ **Error común:** Olvidar comentar código complejo. En un pipeline ETL con 500 líneas, los comentarios son la diferencia entre código mantenible e inmantenible.

---

### Palabras reservadas

Son palabras con significado predefinido en el lenguaje. **No se pueden usar como nombres de variables.**

```python
import keyword
print(keyword.kwlist)

# Las 35 palabras reservadas de Python 3.12:
# False, None, True, and, as, assert, async, await,
# break, class, continue, def, del, elif, else,
# except, finally, for, from, global, if, import,
# in, is, lambda, nonlocal, not, or, pass, raise,
# return, try, while, with, yield
```

> ⚠️ **Error común:** Usar `list`, `dict`, `type` o `input` como nombres de variables. Aunque no son palabras reservadas, **sobrescriben funciones built-in** de Python — un error difícil de depurar en pipelines.

---

### Variables

Una variable es un nombre que apunta a un valor en memoria. En Python, **el tipo se infiere automáticamente** al asignar el valor (tipado dinámico).

```python
# Declaración y asignación
soy_numero   = 22              # int
soy_texto    = "Data Engineer" # str
soy_decimal  = 3.14            # float
soy_logico   = True            # bool

# Reasignación: la variable puede cambiar de tipo
x = 10        # x es int
x = "ahora soy texto"  # x ahora es str (Python lo permite)
```

**Reglas para nombres de variables:**
- Pueden contener letras, números y guiones bajos `_`
- No pueden comenzar con un número
- No pueden contener espacios
- Son sensibles a mayúsculas/minúsculas (`edad` ≠ `Edad`)

```python
# ✅ Válido
precio_unitario = 1500
total_registros = 10000
es_activo = True

# ❌ Inválido
# una variable = 10    → SyntaxError (espacio en el nombre)
# 2precio = 1500       → SyntaxError (comienza con número)
```

> 💡 **Convención PEP 8:** En Python se usa `snake_case` (palabras separadas por `_`) para nombres de variables y funciones. Esto es obligatorio en código profesional de Data Engineering.

---

### Errores en Python

Python es interpretado: detiene la ejecución en la primera línea con error.

```python
# Tipos comunes de error:

# SyntaxError: error de sintaxis (se detecta antes de ejecutar)
# print("Hola, Python!)    ← falta comilla de cierre

# NameError: uso de variable no definida
# print(variable_inexistente)

# TypeError: operación con tipos incompatibles
# print("texto" + 5)       ← no se puede sumar str con int

# Comportamiento del intérprete:
print("Esto será impreso")
# frint("Esto causará un error")    ← detiene la ejecución aquí
# print("Esto NO se imprimirá")     ← nunca se alcanza
```

---

## Bloque 3 — Tipos de datos y operadores {#bloque-3}

### Tipos de datos básicos

| Tipo | Declaración | Ejemplo | Uso en DE |
|---|---|---|---|
| Entero | `int` | `x = 124` | IDs, conteos de registros, años |
| Decimal | `float` | `x = 124.56` | Precios, coordenadas, métricas |
| Texto | `str` | `x = "Hola"` | Nombres, códigos, rutas de archivos |
| Booleano | `bool` | `x = True` | Flags de validación, filtros |

```python
# Verificar el tipo de una variable con type()
entero   = 5
flotante = 1.6
texto    = "Python"
logico   = True

print(type(entero))    # <class 'int'>
print(type(flotante))  # <class 'float'>
print(type(texto))     # <class 'str'>
print(type(logico))    # <class 'bool'>
```

---

### Operadores matemáticos

| Operador | Descripción | Ejemplo | Resultado |
|---|---|---|---|
| `+` | Suma | `10 + 3` | `13` |
| `-` | Resta | `10 - 3` | `7` |
| `*` | Multiplicación | `10 * 3` | `30` |
| `/` | División (siempre float) | `10 / 3` | `3.333...` |
| `//` | División entera (floor) | `10 // 3` | `3` |
| `%` | Módulo (residuo) | `10 % 3` | `1` |
| `**` | Exponenciación | `10 ** 3` | `1000` |

> ⚠️ **Regla de oro:** Cualquier operación entre `int` y `float` produce `float`. La división `/` **siempre** retorna `float`, incluso si el resultado es exacto (`10 / 2 = 5.0`).

```python
a = 10
b = 3

print(f"Suma:            {a} + {b}  = {a + b}")    # 13
print(f"Resta:           {a} - {b}  = {a - b}")    # 7
print(f"Multiplicación:  {a} * {b}  = {a * b}")    # 30
print(f"División:        {a} / {b}  = {a / b}")    # 3.3333...
print(f"División entera: {a} // {b} = {a // b}")   # 3
print(f"Módulo:          {a} % {b}  = {a % b}")    # 1
print(f"Exponenciación:  {a} ** {b} = {a ** b}")   # 1000
```

**Precedencia de operaciones** (de mayor a menor):
```
1. Paréntesis ()
2. Exponenciación **
3. Multiplicación *, División /, División entera //, Módulo %
4. Suma +, Resta -
```

```python
# Ejemplo de precedencia
resultado = 2 + 3 * 4     # 14  (no 20)
resultado = (2 + 3) * 4   # 20  (paréntesis primero)
resultado = 2 ** 3 + 1    # 9   (exponente primero: 8 + 1)
```

> 💡 **Contexto DE:** El operador `%` (módulo) es muy útil en pipelines. Por ejemplo, para saber si un registro tiene ID par o impar: `id % 2 == 0`. El operador `//` se usa para calcular páginas en APIs con paginación: `total_registros // tamaño_pagina`.

---

## Bloque 4 — Strings: operaciones y métodos {#bloque-4}

### Creación de strings

Un string es una **cadena de caracteres**. Se puede crear con comillas simples `'` o dobles `"`.

```python
nombre   = 'Juan'
apellido = "Pérez"

# Para incluir comillas dentro del string, mezclar tipos:
frase1 = 'El sistema dijo "Error 404"'       # comillas dobles dentro
frase2 = "La ruta es C:\\Users\\data"        # barra invertida escapada

# String de múltiples líneas
descripcion = """Este es un texto
que ocupa múltiples
líneas."""
```

### Operaciones con strings

```python
nombre   = "Juan"
apellido = "Pérez"

# Concatenación (unir strings)
nombre_completo = "Mi nombre es " + nombre + " " + apellido
print(nombre_completo)    # Mi nombre es Juan Pérez

# Repetición
print(3 * nombre)         # JuanJuanJuan

# Longitud
print(len(nombre))        # 4
```

### Indexación y slicing

```python
texto = "Python"
#         0 1 2 3 4 5    (índices positivos)
#        -6-5-4-3-2-1    (índices negativos)

print(texto[0])     # P  (primer carácter)
print(texto[-1])    # n  (último carácter)
print(texto[1:4])   # yth (desde índice 1 hasta 3, sin incluir 4)
print(texto[:3])    # Pyt (desde el inicio hasta índice 2)
print(texto[2:])    # thon (desde índice 2 hasta el final)
print(texto[::2])   # Pto (cada 2 caracteres)
```

### Métodos útiles de strings

| Método | Descripción | Ejemplo |
|---|---|---|
| `len(s)` | Longitud del string | `len("Python")` → `6` |
| `s.lower()` | Todo en minúsculas | `"PYTHON".lower()` → `"python"` |
| `s.upper()` | Todo en mayúsculas | `"python".upper()` → `"PYTHON"` |
| `s.capitalize()` | Primera letra mayúscula | `"python".capitalize()` → `"Python"` |
| `s.title()` | Cada palabra con mayúscula inicial | `"hola mundo".title()` → `"Hola Mundo"` |
| `s.strip()` | Elimina espacios al inicio y fin | `"  hola  ".strip()` → `"hola"` |
| `s.lstrip()` | Elimina espacios a la izquierda | `"  hola".lstrip()` → `"hola"` |
| `s.rstrip()` | Elimina espacios a la derecha | `"hola  ".rstrip()` → `"hola"` |
| `s.replace(old, new)` | Reemplaza subcadenas | `"hola mundo".replace("mundo", "Python")` → `"hola Python"` |
| `s.split(sep)` | Divide en lista por separador | `"a,b,c".split(",")` → `['a', 'b', 'c']` |
| `sep.join(lista)` | Une lista con separador | `",".join(["a","b","c"])` → `"a,b,c"` |
| `s.find(sub)` | Índice de primera aparición | `"Python".find("th")` → `2` |
| `s.count(sub)` | Cuenta apariciones de subcadena | `"Hola mundo".count("o")` → `2` |
| `s.isalpha()` | True si solo tiene letras | `"Python".isalpha()` → `True` |
| `s.isdigit()` | True si solo tiene dígitos | `"1234".isdigit()` → `True` |
| `s.swapcase()` | Invierte mayúsculas/minúsculas | `"Hola".swapcase()` → `"hOLA"` |

```python
# Ejemplos prácticos
nombre_raw = "  ana gómez  "

# Limpieza típica al ingestar datos
nombre_limpio = nombre_raw.strip().title()
print(nombre_limpio)    # Ana Gómez

# Separar un CSV manual
fila = "Santiago,Chile,33.45"
campos = fila.split(",")
print(campos)           # ['Santiago', 'Chile', '33.45']
print(campos[1])        # Chile
```

> 💡 **Contexto DE:** `.strip()`, `.lower()`, `.replace()` y `.split()` son las operaciones más usadas al **limpiar datos de texto** antes de cargarlos en una base de datos. Los datos crudos suelen llegar con espacios extra, mayúsculas inconsistentes y separadores variables.

> ⚠️ **Error común:** Olvidar que los strings son **inmutables**. Los métodos no modifican el string original; retornan uno nuevo. Siempre asignar el resultado: `nombre = nombre.strip()`.

---

## Bloque 5 — Conversión de tipos e input() {#bloque-5}

### Conversión de tipos (type casting)

Python permite convertir entre tipos usando funciones built-in:

| Función | Convierte a | Ejemplo |
|---|---|---|
| `int(x)` | Entero | `int("42")` → `42`; `int(3.9)` → `3` |
| `float(x)` | Decimal | `float("3.14")` → `3.14`; `float(5)` → `5.0` |
| `str(x)` | String | `str(42)` → `"42"` |
| `bool(x)` | Booleano | `bool(0)` → `False`; `bool("hola")` → `True` |

```python
# int(): trunca (no redondea) al convertir float
x = int(3.9)    # x = 3  (no 4)
y = int("42")   # y = 42

# float()
a = float("3.14")   # a = 3.14
b = float(5)        # b = 5.0

# str(): convierte cualquier valor a texto
c = str(42)         # c = "42"

# bool(): valores "falsos" en Python
# 0, 0.0, "" (string vacío), [] (lista vacía), {}, None → False
# Todo lo demás → True
print(bool(0))      # False
print(bool("Hola")) # True
```

> ⚠️ **Error común:** Intentar convertir texto no numérico a `int` o `float` genera `ValueError`. Siempre validar antes o usar `try/except`.

```python
# ❌ Esto falla
# int("Hola")    → ValueError: invalid literal for int()

# ✅ Conversión segura
valor = "123"
resultado = int(valor)   # 123  → funciona porque "123" es numérico

# 💡 En pipelines de datos, es común recibir números como strings
# desde archivos CSV o APIs → hay que convertirlos antes de operar
```

### La función `input()`

`input()` permite capturar datos ingresados por el usuario desde la consola. **Siempre retorna un `str`**, independiente de lo que se ingrese.

```python
# Sintaxis
variable = input("Mensaje para el usuario: ")

# Ejemplo básico
nombre = input("Ingresa tu nombre: ")
print(f"Hola, {nombre}!")

# Como siempre retorna str, hay que convertir si necesitamos números
edad    = int(input("Ingresa tu edad: "))
altura  = float(input("Ingresa tu altura en metros: "))

print(f"Tienes {edad} años y mides {altura} m")
```

> ⚠️ **Importante:** En pipelines de Data Engineering, `input()` raramente se usa (los datos vienen de archivos, APIs o bases de datos). Sin embargo, es fundamental para aprender la interacción básica y para construir scripts de utilidad o configuración.

---

<details>
<summary><strong>🟢 Ejercicio 1 — Variables y operadores básicos (click para ver)</strong></summary>

**Enunciado:**
1. Crear dos variables `a = 5` y `b = 3`. Mostrar la suma, resta, multiplicación, división y módulo.
2. Crear `x = 10` y `y = 3.5`. Calcular y mostrar `x + y`, `x * y` y `x / y`.
3. Crear `texto = "Python"` y mostrar en mayúsculas, minúsculas y capitalizado.
4. Contar cuántas veces aparece la letra `"o"` en `"Hola mundo"`.

**Solución:**
```python
# 1. Operadores con enteros
a = 5
b = 3
print(a + b)   # 8
print(a - b)   # 2
print(a * b)   # 15
print(a / b)   # 1.666...
print(a % b)   # 2

# 2. Mezcla int y float
x = 10
y = 3.5
print(x + y)   # 13.5
print(x * y)   # 35.0
print(x / y)   # 2.857...

# 3. Métodos de string
texto = "Python"
print(texto.lower())       # python
print(texto.upper())       # PYTHON
print(texto.capitalize())  # Python

# 4. Contar apariciones
print("Hola mundo".count("o"))  # 2
```

</details>

<details>
<summary><strong>🟢 Ejercicio 2 — input() y cálculos (click para ver)</strong></summary>

**Enunciado:**
5. Pedir al usuario su nombre y edad, y mostrar un mensaje personalizado.
6. Calcular el área de un rectángulo: pedir base y altura. (`Área = base × altura`)
7. Calcular el área de un círculo: pedir el radio. (`Área = π × radio²`)

**Solución:**
```python
import math  # para usar math.pi

# 5. Nombre y edad
nombre = input("Ingresa tu nombre: ")
edad   = int(input("Ingresa tu edad: "))
print(f"Tu eres {nombre} y tienes {edad} años")

# 6. Área del rectángulo
base   = float(input("Ingresa la base: "))
altura = float(input("Ingresa la altura: "))
area_rect = base * altura
print(f"Área del rectángulo: {area_rect}")

# 7. Área del círculo
radio    = float(input("Ingresa el radio: "))
area_circ = math.pi * radio ** 2
print(f"Área del círculo: {area_circ:.2f}")
```

</details>

---

## Bloque 6 — Estructuras de datos {#bloque-6}

Python tiene cuatro estructuras de datos built-in que son el corazón del manejo de información:

```
Resumen de estructuras de datos
─────────────────────────────────────────────────────────────────
  Tipo        Sintaxis         ¿Ordenada?  ¿Mutable?  Duplicados?
  ──────────  ───────────────  ──────────  ─────────  ──────────
  list        [1, 2, 3]           Sí          Sí         Sí
  tuple       (1, 2, 3)           Sí          No         Sí
  dict        {"k": "v"}          Sí*         Sí     Claves: No
  set         {1, 2, 3}           No          Sí         No
─────────────────────────────────────────────────────────────────
  * dict mantiene orden de inserción desde Python 3.7
```

---

### 6.1 — Listas (`list`)

Una lista es una colección **ordenada y modificable** de elementos. Se define con corchetes `[]`.

```python
# Crear listas
frutas   = ["manzana", "banana", "cereza"]
numeros  = [10, 20, 30, 40]
mixta    = [1, "hola", True, 3.14]   # puede mezclar tipos
vacia    = []                         # lista vacía
```

#### Acceder a elementos (indexación)

```python
frutas = ["manzana", "banana", "cereza"]
#               0         1         2     (positivos)
#              -3        -2        -1     (negativos)

print(frutas[0])    # manzana (primer elemento)
print(frutas[1])    # banana
print(frutas[-1])   # cereza (último elemento)
print(frutas[-2])   # banana (penúltimo)
```

#### Slicing (rebanado)

```python
numeros = [10, 20, 30, 40, 50]

print(numeros[1:4])   # [20, 30, 40]   (desde 1 hasta 3, sin incluir 4)
print(numeros[:3])    # [10, 20, 30]   (desde inicio hasta índice 2)
print(numeros[2:])    # [30, 40, 50]   (desde índice 2 hasta el final)
print(numeros[::2])   # [10, 30, 50]   (cada 2 elementos)
```

#### Modificar elementos

```python
frutas = ["manzana", "banana", "cereza"]
frutas[1] = "pera"
print(frutas)   # ["manzana", "pera", "cereza"]
```

#### Métodos de listas

| Método | Acción |
|---|---|
| `lista.append(x)` | Agrega `x` al final |
| `lista.insert(i, x)` | Inserta `x` en posición `i` |
| `lista.remove(x)` | Elimina la primera aparición de `x` |
| `lista.pop()` | Elimina y retorna el último elemento |
| `lista.pop(i)` | Elimina y retorna el elemento en posición `i` |
| `lista.clear()` | Elimina todos los elementos |
| `lista.index(x)` | Retorna el índice de la primera aparición de `x` |
| `lista.sort()` | Ordena la lista en su lugar (modifica la original) |
| `sorted(lista)` | Retorna una nueva lista ordenada (no modifica la original) |
| `lista.reverse()` | Invierte el orden de la lista |
| `lista.copy()` | Retorna una copia de la lista |
| `lista.extend(otra)` | Agrega todos los elementos de `otra` al final |

```python
frutas = ["manzana", "banana", "cereza"]

frutas.append("pera")        # ["manzana", "banana", "cereza", "pera"]
frutas.insert(1, "uva")      # ["manzana", "uva", "banana", "cereza", "pera"]
frutas.remove("banana")      # ["manzana", "uva", "cereza", "pera"]
frutas.pop()                 # elimina "pera" → ["manzana", "uva", "cereza"]
frutas.sort()                # ["cereza", "manzana", "uva"]

# Funciones sobre listas
numeros = [1, 2, 3, 4, 5]
print(len(numeros))   # 5
print(sum(numeros))   # 15
print(max(numeros))   # 5
print(min(numeros))   # 1
```

#### Listas anidadas (matrices)

```python
# Representar una tabla de datos como lista de listas
matriz = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

print(matriz[0])        # [1, 2, 3]   (primera fila)
print(matriz[1][2])     # 6           (fila 2, columna 3)
```

> 💡 **Contexto DE:** Las listas son la base de muchas operaciones en pipelines. Por ejemplo, una lista de rutas de archivos a procesar, una lista de nombres de columnas, o una lista de errores encontrados durante la validación de datos.

---

### 6.2 — Tuplas (`tuple`)

Una tupla es una colección **ordenada e inmutable**. Se define con paréntesis `()`. Al no poder modificarse, son más eficientes en memoria que las listas.

```python
# Crear tuplas
frutas    = ("manzana", "banana", "cereza")
coordenas = (33.45, -70.65)   # latitud, longitud
rgb       = (255, 128, 0)     # color en formato RGB
```

```python
frutas = ("manzana", "banana", "cereza")

# Acceder (igual que listas)
print(frutas[0])    # manzana
print(frutas[-1])   # cereza

# Recorrer
for fruta in frutas:
    print(fruta)

# Operaciones disponibles
numeros = (1, 2, 3, 2, 4, 2)
print(len(numeros))        # 6
print(numeros.count(2))    # 3  (cuántas veces aparece el 2)
print(numeros.index(4))    # 4  (índice donde aparece el 4)
```

```python
# ❌ Las tuplas NO se pueden modificar
# frutas[0] = "pera"   → TypeError: 'tuple' object does not support item assignment
```

> 💡 **Contexto DE:** Las tuplas se usan para almacenar datos que **no deben cambiar**: coordenadas geográficas, credenciales de conexión (host, puerto, db), o los nombres de columnas de un esquema fijo. Son más seguras que las listas cuando la inmutabilidad es una garantía importante.

---

### 6.3 — Sets (`set`)

Un set es una colección **desordenada, sin índices y sin elementos duplicados**. Se define con llaves `{}` o con `set()`.

```python
# Crear sets
frutas  = {"manzana", "banana", "cereza"}
numeros = {1, 2, 3, 4, 5}
vacio   = set()   # ⚠️ {} crea un diccionario vacío, NO un set vacío

# El orden NO está garantizado
print({3, 1, 4, 1, 5, 9, 2, 6})  # {1, 2, 3, 4, 5, 6, 9} (sin duplicados)
```

```python
frutas = {"manzana", "banana", "cereza"}

# Verificar pertenencia (muy eficiente en sets)
print("banana" in frutas)   # True
print("pera" in frutas)     # False

# Agregar y eliminar
frutas.add("pera")          # agrega un elemento
frutas.remove("banana")     # elimina (error si no existe)
frutas.discard("kiwi")      # elimina sin error si no existe
```

#### Operaciones de conjuntos

```python
A = {1, 2, 3, 4}
B = {3, 4, 5, 6}

print(A | B)   # Unión:        {1, 2, 3, 4, 5, 6}
print(A & B)   # Intersección: {3, 4}
print(A - B)   # Diferencia:   {1, 2}   (en A pero no en B)
```

```
Operaciones de conjuntos — Diagrama de Venn
─────────────────────────────────────────────────
  A = {1, 2, 3, 4}     B = {3, 4, 5, 6}

  Unión (A | B):
  ┌──────────────────┐
  │ 1  2  ┌──────┐   │
  │       │ 3  4 │5 6│
  └───────┤      ├───┘
          └──────┘
  Resultado: {1, 2, 3, 4, 5, 6}

  Intersección (A & B):
             ┌──────┐
             │ 3  4 │
             └──────┘
  Resultado: {3, 4}
─────────────────────────────────────────────────
```

> 💡 **Contexto DE:** El uso más común de sets en Data Engineering es **eliminar duplicados rápidamente**:

```python
# Caso real: lista de IDs de clientes con duplicados
ids_clientes = [101, 102, 103, 101, 104, 102, 105]

# Eliminar duplicados con set
ids_unicos = list(set(ids_clientes))
print(ids_unicos)    # [101, 102, 103, 104, 105] (sin duplicados)
print(f"Duplicados eliminados: {len(ids_clientes) - len(ids_unicos)}")
```

---

### 6.4 — Diccionarios (`dict`)

Un diccionario almacena pares **clave:valor**. Las claves son únicas; los valores pueden ser de cualquier tipo. Se define con llaves `{}`.

```python
# Crear un diccionario
persona = {
    "nombre":   "Ana",
    "edad":     25,
    "ciudad":   "Santiago",
    "activo":   True
}
```

#### Acceder a valores

```python
persona = {"nombre": "Ana", "edad": 25, "ciudad": "Santiago"}

# Acceso directo (error si la clave no existe)
print(persona["nombre"])    # Ana
print(persona["edad"])      # 25

# Acceso con .get() → retorna None (o valor por defecto) si la clave no existe
print(persona.get("pais"))           # None
print(persona.get("pais", "NE"))     # NE (valor por defecto)
```

#### Modificar, agregar y eliminar

```python
persona = {"nombre": "Ana", "edad": 25, "ciudad": "Santiago"}

# Modificar
persona["edad"] = 26

# Agregar nueva clave
persona["profesion"] = "Data Engineer"

# Eliminar
persona.pop("ciudad")     # elimina por clave, retorna el valor
del persona["edad"]       # elimina con del
persona.clear()           # elimina todo
```

#### Recorrer un diccionario

```python
estudiante = {"nombre": "Luis", "edad": 20, "carrera": "Informática"}

# Solo claves
for clave in estudiante:
    print(clave)            # nombre, edad, carrera

# Solo valores
for valor in estudiante.values():
    print(valor)            # Luis, 20, Informática

# Claves y valores juntos
for clave, valor in estudiante.items():
    print(f"{clave}: {valor}")  # nombre: Luis, edad: 20, ...
```

#### Métodos útiles

```python
estudiante = {"nombre": "Luis", "edad": 20, "carrera": "Informática"}

print(estudiante.keys())     # dict_keys(['nombre', 'edad', 'carrera'])
print(estudiante.values())   # dict_values(['Luis', 20, 'Informática'])
print(estudiante.items())    # dict_items([('nombre', 'Luis'), ...])
```

#### Diccionarios anidados

```python
# Caso real: registro de alumnos con datos estructurados
alumnos = {
    "A001": {"nombre": "Ana",   "nota": 90},
    "A002": {"nombre": "Pedro", "nota": 75},
    "A003": {"nombre": "Lucía", "nota": 60}
}

print(alumnos["A001"]["nombre"])   # Ana
print(alumnos["A002"]["nota"])     # 75

# Recorrer todos los alumnos
for codigo, datos in alumnos.items():
    print(f"{codigo}: {datos['nombre']} - {datos['nota']}")
```

> 💡 **Contexto DE:** Los diccionarios son la estructura de datos más usada en Data Engineering para representar **registros de datos**, **schemas**, **configuraciones** y **respuestas de APIs JSON**. Una fila de una tabla de base de datos suele representarse como un diccionario: `{"id": 1, "nombre": "Ana", "monto": 5000.0}`.

> ⚠️ **Error común:** Acceder a una clave inexistente con `[]` lanza `KeyError`. En pipelines de producción, siempre preferir `.get()` para evitar que el proceso falle por datos inesperados.

---

### Resumen comparativo de estructuras de datos

| Estructura | Sintaxis | Ordenada | Mutable | Duplicados | Cuándo usarla |
|---|---|---|---|---|---|
| **Lista** | `[1, 2, 3]` | ✅ | ✅ | ✅ | Colección que puede cambiar: registros a procesar, log de errores |
| **Tupla** | `(1, 2, 3)` | ✅ | ❌ | ✅ | Datos fijos: coordenadas, credenciales, esquemas |
| **Set** | `{1, 2, 3}` | ❌ | ✅ | ❌ | Eliminar duplicados, operaciones de conjuntos |
| **Diccionario** | `{"k": "v"}` | ✅* | ✅ | Claves: ❌ | Registros clave:valor, config, respuestas JSON/API |

---

<details>
<summary><strong>🟢 Ejercicio 3 — Listas y operaciones (click para ver)</strong></summary>

**Enunciado:**
Dado el siguiente pipeline de nombres de archivos CSV a procesar:

```python
archivos = ["ventas_enero.csv", "ventas_febrero.csv", "ventas_enero.csv", "ventas_marzo.csv"]
```

1. Mostrar cuántos archivos hay en la lista.
2. Agregar `"ventas_abril.csv"` al final.
3. Eliminar el archivo duplicado.
4. Mostrar la lista ordenada.

**Solución:**
```python
archivos = ["ventas_enero.csv", "ventas_febrero.csv", "ventas_enero.csv", "ventas_marzo.csv"]

# 1. Cantidad de archivos
print(f"Total: {len(archivos)}")                   # Total: 4

# 2. Agregar nuevo archivo
archivos.append("ventas_abril.csv")
print(archivos)

# 3. Eliminar duplicado
archivos.remove("ventas_enero.csv")  # elimina la primera aparición
print(archivos)

# 4. Lista ordenada
print(sorted(archivos))
```

</details>

<details>
<summary><strong>🟢 Ejercicio 4 — Diccionarios y estructuras compuestas (click para ver)</strong></summary>

**Enunciado:**
Crear un diccionario que represente un producto de una tienda con las claves: `id`, `nombre`, `precio`, `stock`. Luego:
1. Mostrar el nombre y precio del producto.
2. Actualizar el precio con un 10% de descuento.
3. Verificar si la clave `"categoria"` existe en el diccionario.

**Solución:**
```python
producto = {
    "id":     "P001",
    "nombre": "Laptop",
    "precio": 850000,
    "stock":  15
}

# 1. Mostrar nombre y precio
print(f"Producto: {producto['nombre']}, Precio: ${producto['precio']:,}")

# 2. Aplicar descuento del 10%
producto["precio"] = producto["precio"] * 0.90
print(f"Precio con descuento: ${producto['precio']:,.0f}")

# 3. Verificar si existe la clave
if "categoria" in producto:
    print(f"Categoría: {producto['categoria']}")
else:
    print("El producto no tiene categoría asignada")

# Alternativa con .get()
categoria = producto.get("categoria", "Sin categoría")
print(categoria)
```

</details>

---

## Referencia rápida — Python Módulo 2

```
TIPOS DE DATOS
─────────────────────────────────────────────────────
  int     → 42          Enteros
  float   → 3.14        Decimales
  str     → "hola"      Texto
  bool    → True/False  Lógico

OPERADORES
─────────────────────────────────────────────────────
  +  -  *  /            Básicos
  //  %  **             División entera, módulo, potencia

ESTRUCTURAS DE DATOS
─────────────────────────────────────────────────────
  [a, b, c]             Lista: ordenada, mutable
  (a, b, c)             Tupla: ordenada, inmutable
  {a, b, c}             Set: desordenado, sin duplicados
  {"k": "v"}            Dict: clave:valor

CONVERSIÓN DE TIPOS
─────────────────────────────────────────────────────
  int(x)   float(x)   str(x)   bool(x)

MÉTODOS CLAVE PARA DATA ENGINEERING
─────────────────────────────────────────────────────
  str      → .strip()  .lower()  .split()  .replace()
  list     → .append() .remove() .sort()   .extend()
  dict     → .get()    .keys()   .values() .items()
  set      → .add()    .discard() (uso: deduplicar)
```

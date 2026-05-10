# Clase 4 — Archivos, NumPy y Consumo de APIs

---

## Tabla de contenidos

1. [Manejo de archivos de texto](#bloque-1)
2. [Archivos CSV con el módulo `csv`](#bloque-2)
3. [Lectura eficiente de archivos grandes](#bloque-3)
4. [NumPy — Computación numérica eficiente](#bloque-4)
5. [Creación y propiedades de arrays](#bloque-5)
6. [Operaciones, indexing y slicing](#bloque-6)
7. [Funciones universales y `np.where`](#bloque-7)
8. [Consumo de APIs](#bloque-8)
9. [JSON — el formato de las APIs](#bloque-9)
10. [Ejemplos prácticos de APIs](#bloque-10)

---

## Bloque 1 — Manejo de archivos de texto {#bloque-1}

Un **archivo** es un conjunto de datos almacenado en disco que persiste más allá de la ejecución del programa. En Python, la función `open()` es la puerta de entrada para trabajar con cualquier archivo.

### Modos de apertura

```python
archivo = open("nombre.txt", "modo", encoding="utf-8")
```

| Modo | Nombre | Comportamiento |
|---|---|---|
| `"r"` | Read | Lee el archivo. Error si no existe |
| `"w"` | Write | Escribe. **Crea o sobrescribe** si ya existe |
| `"a"` | Append | Agrega al final. Crea si no existe |
| `"r+"` | Read+Write | Lee y escribe sin borrar el contenido |

> ⚠️ **Siempre especifica `encoding="utf-8"`** cuando trabajes con texto en español. Sin este parámetro, los caracteres con tilde o ñ pueden causar errores en algunos sistemas (especialmente en Windows con `cp1252` por defecto).

---

### Leer un archivo completo

```python
archivo = open("notas.txt", "r", encoding="utf-8")
contenido = archivo.read()    # lee TODO el archivo como un string
print(contenido)
archivo.close()               # importante: cerrar siempre
```

### Leer línea por línea

```python
archivo = open("notas.txt", "r", encoding="utf-8")
for linea in archivo:
    print(linea.strip())      # .strip() elimina el \n al final
archivo.close()
```

### Leer todas las líneas como lista

```python
archivo = open("notas.txt", "r", encoding="utf-8")
lineas = archivo.readlines()  # retorna lista de strings
print(lineas)                 # ["línea 1\n", "línea 2\n", ...]
archivo.close()
```

---

### El contexto `with` — la mejor práctica

Python ofrece el gestor de contexto `with open(...)`, que **cierra el archivo automáticamente** al salir del bloque, incluso si ocurre un error. Es la forma recomendada siempre.

```python
# Antes: manual y propenso a olvidar .close()
archivo = open("datos.txt", "r", encoding="utf-8")
contenido = archivo.read()
archivo.close()

# Ahora: con with (recomendado)
with open("datos.txt", "r", encoding="utf-8") as archivo:
    contenido = archivo.read()
# El archivo se cierra automáticamente aquí
```

### Escribir en un archivo

```python
# Modo "w": crea o sobrescribe
with open("resultado.txt", "w", encoding="utf-8") as archivo:
    archivo.write("Primera línea\n")
    archivo.write("Segunda línea\n")

# Modo "a": agrega sin borrar lo existente
with open("resultado.txt", "a", encoding="utf-8") as archivo:
    archivo.write("Línea agregada después\n")
```

### Ejemplo completo: guardar y leer una lista

```python
# Guardar nombres en un archivo
nombres = ["Ana", "Juan", "Pedro", "Lucía"]

with open("nombres.txt", "w", encoding="utf-8") as archivo:
    for nombre in nombres:
        archivo.write(nombre + "\n")

# Leer los nombres y procesarlos
with open("nombres.txt", "r", encoding="utf-8") as archivo:
    for linea in archivo:
        print("Nombre:", linea.strip())
# Output:
# Nombre: Ana
# Nombre: Juan
# Nombre: Pedro
# Nombre: Lucía
```

> 💡 **Contexto DE:** Los archivos de texto y log son ubicuos en Data Engineering. Los pipelines escriben logs de errores, registros de ejecución y archivos de auditoría. Saber leerlos y escribirlos correctamente es una habilidad básica del Data Engineer.

---

## Bloque 2 — Archivos CSV con el módulo `csv` {#bloque-2}

Los **CSV** (*Comma-Separated Values*) son el formato más común para intercambiar datos tabulares. Python incluye el módulo `csv` en su librería estándar (sin necesidad de instalarlo).

> ℹ️ **¿Cuándo usar `csv` y cuándo `pandas`?** Pandas (`read_csv`) es la opción habitual para analizar datos. El módulo `csv` es útil cuando necesitas **más control** sobre la lectura, cuando el archivo es muy grande para cargarlo en memoria, o cuando no quieres depender de Pandas en un script ligero (ej: una pequeña Lambda en AWS).

### Leer un CSV

```python
import csv

with open("personas.csv", "r", encoding="utf-8") as archivo:
    lector = csv.reader(archivo)
    for fila in lector:
        print(fila)          # cada fila es una lista
# Output: ['Ana', '28', 'Santiago']
#         ['Luis', '35', 'Valparaíso']
```

### Leer como diccionario (`DictReader`)

`DictReader` convierte cada fila en un diccionario usando la primera fila como claves:

```python
import csv

with open("personas.csv", "r", encoding="utf-8") as archivo:
    lector = csv.DictReader(archivo)
    for fila in lector:
        print(fila["Nombre"], "-", fila["Ciudad"])
# Output: Ana - Santiago
#         Luis - Valparaíso
```

### Escribir un CSV

```python
import csv

datos = [
    ["Nombre", "Edad", "Ciudad"],       # encabezado
    ["Pedro",  28,     "La Serena"],
    ["María",  35,     "Antofagasta"]
]

with open("nuevas_personas.csv", "w", encoding="utf-8", newline="") as archivo:
    escritor = csv.writer(archivo)
    escritor.writerows(datos)           # escribe todas las filas a la vez
```

> ⚠️ El parámetro `newline=""` en modo escritura es necesario en Windows para evitar líneas en blanco dobles entre filas.

### Agregar filas a un CSV existente

```python
import csv

nuevos = [
    ["Carlos", 40, "Puerto Montt"],
    ["Sofía",  27, "Temuco"]
]

with open("nuevas_personas.csv", "a", encoding="utf-8", newline="") as archivo:
    escritor = csv.writer(archivo)
    escritor.writerows(nuevos)
```

---

## Bloque 3 — Lectura eficiente de archivos grandes {#bloque-3}

Cuando un archivo tiene cientos de MB o varios GB, **no es posible cargarlo entero en memoria** con `.read()` o `.readlines()`. La solución es procesarlo de forma incremental.

### Streaming: leer línea a línea

El iterador `for linea in archivo` es la forma más eficiente: solo mantiene una línea en memoria a la vez.

```python
# Eficiente: procesa línea a línea sin cargar todo en RAM
with open("archivo_grande.txt", "r", encoding="utf-8") as archivo:
    for linea in archivo:
        procesar(linea.strip())    # solo una línea en memoria a la vez
```

### Control fino con `readline()`

```python
with open("archivo.txt", "r", encoding="utf-8") as archivo:
    linea = archivo.readline()     # leer la primera línea
    while linea:
        print(linea.strip())
        linea = archivo.readline() # leer la siguiente
```

### Leer solo las primeras N líneas

```python
# Leer solo el encabezado + primeras 3 filas de datos
with open("archivo.txt", "r", encoding="utf-8") as archivo:
    for _ in range(3):
        print(archivo.readline().strip())
```

> 💡 **Contexto DE:** En producción, los archivos de log de un sistema pueden pesar varios GB. Un pipeline que los procesa no puede cargarlos enteros en memoria. La lectura por streaming, combinada con un buffer o procesamiento por chunks, es la técnica estándar para estos casos. Pandas también lo soporta con el parámetro `chunksize` en `read_csv()` — útil cuando el archivo no cabe en RAM.

---

## Bloque 4 — NumPy: Computación numérica eficiente {#bloque-4}

**NumPy** (*Numerical Python*) es la librería base para el cómputo numérico en Python. Provee el `ndarray`: un array N-dimensional extremadamente rápido y eficiente en memoria.

### ¿Por qué NumPy y no listas de Python?

La diferencia de velocidad es drástica. Aquí el benchmark real con 10 millones de números:

```python
import numpy as np

x = np.random.randn(10_000_000)

# Python nativo
%timeit sum(x)       # ~720 ms

# NumPy
%timeit np.sum(x)    # ~14.6 ms
```

**NumPy es ~50× más rápido** para esta operación. La diferencia crece con operaciones más complejas.

```
¿Por qué es tan rápido NumPy?
──────────────────────────────────────────────────────────────────
  Lista Python               ndarray NumPy
  ─────────────────────      ─────────────────────────────────────
  Objetos Python en RAM      Bloque contiguo de memoria
  Un tipo por elemento       Un tipo para todos (dtype uniforme)
  Loop en Python             Operaciones en C/Fortran
  Interpretado               Vectorizado (sin loops visibles)
──────────────────────────────────────────────────────────────────
```

> 💡 **Conexión:** Pandas está construido sobre NumPy. Cuando ejecutas `df["col"].mean()`, por debajo se llama a una operación NumPy. Por eso entender NumPy te hace mejor con Pandas.

---

## Bloque 5 — Creación y propiedades de arrays {#bloque-5}

```python
import numpy as np
```

### Crear arrays desde listas

```python
# Array 1D (vector)
arr1 = np.array([1.2, 5.2, 5.0, 7.8, 0.3])
print(arr1)        # array([1.2, 5.2, 5. , 7.8, 0.3])

# Array 2D (matriz) desde lista de listas
arr2 = np.array([
    [1.2, 5.2, 5.0, 7.8, 0.3],
    [4.1, 7.2, 4.8, 0.1, 7.7]
])
print(arr2.shape)  # (2, 5) → 2 filas, 5 columnas
print(arr2.ndim)   # 2 → dos dimensiones
print(arr2.dtype)  # float64
```

### Propiedades del ndarray

| Atributo | Descripción | Ejemplo |
|---|---|---|
| `.shape` | Tupla con el tamaño de cada dimensión | `(3, 4)` |
| `.ndim` | Número de dimensiones | `2` |
| `.dtype` | Tipo de datos almacenado | `float64` |
| `.size` | Total de elementos | `12` |

### Funciones para crear arrays

| Función | Descripción | Ejemplo |
|---|---|---|
| `np.array(lista)` | Desde lista Python | `np.array([1, 2, 3])` |
| `np.zeros(n)` | Array de ceros | `np.zeros(5)` |
| `np.zeros((f, c))` | Matriz de ceros | `np.zeros((4, 6))` |
| `np.ones((f, c))` | Matriz de unos | `np.ones((3, 3))` |
| `np.arange(inicio, fin)` | Secuencia entera (como `range`) | `np.arange(0, 10)` |
| `np.linspace(inicio, fin, n)` | n valores equiespaciados | `np.linspace(0, 1, 5)` |
| `np.eye(n)` | Matriz identidad N×N | `np.eye(3)` |
| `np.random.randn(n)` | n valores aleatorios (distribución normal) | `np.random.randn(100)` |

```python
np.zeros(10)          # array([0., 0., 0., ..., 0.])
np.zeros((4, 6))      # matriz 4×6 de ceros
np.arange(0, 10)      # array([0, 1, 2, ..., 9])
np.linspace(0, 1, 5)  # array([0.  , 0.25, 0.5 , 0.75, 1.  ])
np.eye(3)             # matriz identidad 3×3
```

### Tipos de datos (dtype)

NumPy solo puede almacenar **un tipo de dato por array**. Esto es lo que lo hace tan eficiente.

```python
# Especificar dtype al crear
arr = np.array([1, 2, 3], dtype=np.float64)
print(arr.dtype)    # float64

# Convertir dtype con astype
arr_float = np.array([1.7, 2.5, 3.9])
arr_int = arr_float.astype(np.int64)    # array([1, 2, 3]) → trunca, no redondea
```

Tipos más comunes: `np.int32`, `np.int64`, `np.float32`, `np.float64`, `np.bool_`

---

## Bloque 6 — Operaciones, indexing y slicing {#bloque-6}

### Vectorización: operaciones sin bucles

La gran ventaja de NumPy es la **vectorización**: aplicar operaciones a todos los elementos sin escribir un `for`.

```python
A = np.array([[1, 2, 3],
              [4, 5, 6]])

# Operaciones elemento a elemento
A + A       # array([[ 2,  4,  6], [ 8, 10, 12]])
A * A       # array([[ 1,  4,  9], [16, 25, 36]])
A - A       # array([[0, 0, 0], [0, 0, 0]])

# Operaciones con escalar (se aplica a todos los elementos)
A * 5       # array([[ 5, 10, 15], [20, 25, 30]])
A ** 0.5    # raíz cuadrada de cada elemento
A + 10      # suma 10 a cada elemento

# Comparaciones → retornan array de booleanos
B = np.array([[1, 7, 4], [4, 12, 2]])
A > B       # array([[False, False, False], [False, False,  True]])
```

### Broadcasting

*Broadcasting* es la capacidad de NumPy de aplicar operaciones entre arrays de **distintas formas** cuando son compatibles. Es una de sus características más potentes.

```python
A = np.ones((3, 3))    # matriz 3×3 de unos
B = np.arange(3)       # array([0, 1, 2])

A + B
# array([[1., 2., 3.],    ← B se "estiró" a cada fila
#        [1., 2., 3.],
#        [1., 2., 3.]])

# Otro ejemplo clásico: sumar un escalar
a = np.array([1, 2, 3])
a + 2    # array([3, 4, 5])
```

```
Broadcasting visual
──────────────────────────────────────────────────────────
  A (3×3)     B (1×3)          A + B (3×3)
  ─────────   ─────────        ─────────────
  1  1  1     0  1  2    →     1  2  3
  1  1  1  +  0  1  2    →     1  2  3
  1  1  1     0  1  2    →     1  2  3
              ↑ se replica en cada fila
──────────────────────────────────────────────────────────
```

### Indexing y slicing

La sintaxis es similar a las listas de Python, con la extensión a múltiples dimensiones usando comas.

```python
A = np.arange(10)
# array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

A[5]      # 5
A[5:8]    # array([5, 6, 7])
A[:3]     # array([0, 1, 2])
A[::2]    # array([0, 2, 4, 6, 8])  (cada 2 elementos)

# Asignación sobre un slice
A[5:8] = 0
# array([0, 1, 2, 3, 4, 0, 0, 0, 8, 9])
```

#### Matrices 2D

```python
C = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

C[2]        # array([7, 8, 9])       → fila 2 completa
C[2, 1]     # 8                      → fila 2, columna 1
C[:, 1]     # array([2, 5, 8])       → toda la columna 1
C[:2]       # primeras 2 filas
C[:2, 1:]   # primeras 2 filas, desde columna 1

# Sintaxis [filas, columnas]
C[1:2, :2]  # array([[4, 5]])        → fila 1, columnas 0 y 1
```

```
Indexing visual en matriz 3×3
──────────────────────────────────────────
         col 0  col 1  col 2
  fila 0 [  1,    2,    3 ]
  fila 1 [  4,    5,    6 ]
  fila 2 [  7,    8,    9 ]

  C[2, 1]   →  8
  C[:, 1]   →  [2, 5, 8]   (columna entera)
  C[:2, 1:] →  [[2,3],[5,6]] (submatriz)
──────────────────────────────────────────
```

### Reshape y transposición

```python
A = np.arange(15)
# array([0, 1, 2, ..., 14])

B = A.reshape((3, 5))     # convierte a matriz 3×5
# array([[ 0,  1,  2,  3,  4],
#        [ 5,  6,  7,  8,  9],
#        [10, 11, 12, 13, 14]])

B.T                        # transpuesta: 5×3
# array([[ 0,  5, 10],
#        [ 1,  6, 11],
#        [ 2,  7, 12],
#        [ 3,  8, 13],
#        [ 4,  9, 14]])
```

> ⚠️ El total de elementos debe ser el mismo al hacer `reshape`. `np.arange(15).reshape((3, 5))` funciona porque 3×5=15. `reshape((3, 6))` daría error porque 3×6=18 ≠ 15.

---

## Bloque 7 — Funciones universales y `np.where` {#bloque-7}

### Funciones universales (ufuncs)

Las **ufuncs** son funciones que operan **elemento a elemento** sobre arrays enteros, sin necesidad de bucles.

```python
A = np.arange(10)

np.sqrt(A)   # raíz cuadrada de cada elemento
# array([0., 1., 1.414, 1.732, 2., ...])

np.exp(A)    # e^x de cada elemento
# array([1., 2.718, 7.389, ...])

np.abs(A)    # valor absoluto
np.log(A)    # logaritmo natural (nota: log(0) = -inf)
np.sin(A)    # seno de cada elemento
```

#### Ufuncs binarias (dos arrays → un array)

```python
x = np.random.randn(5)   # array aleatorio
y = np.random.randn(5)   # otro array aleatorio

np.maximum(x, y)   # máximo elemento a elemento entre x e y
np.minimum(x, y)   # mínimo elemento a elemento
np.add(x, y)       # equivalente a x + y
```

### Estadísticas sobre arrays

```python
A = np.array([[1, 2, 3], [4, 5, 6]])

np.sum(A)           # 21 (suma de todos los elementos)
np.sum(A, axis=0)   # array([5, 7, 9])  → suma por columna
np.sum(A, axis=1)   # array([ 6, 15])  → suma por fila

np.mean(A)          # 3.5
np.std(A)           # desviación estándar
np.min(A)           # 1
np.max(A)           # 6
```

---

### `np.where` — condicionales vectorizados

`np.where` es el `if/else` vectorizado de NumPy: aplica una condición a **todo el array** de una sola vez, sin bucles.

```python
# Sintaxis
np.where(condición, valor_si_True, valor_si_False)
```

```python
a = np.array([1, 2, 3, 4, 5])

# Clasificar cada elemento
resultado = np.where(a > 3, "Mayor", "Menor o igual")
print(resultado)
# array(['Menor o igual', 'Menor o igual', 'Menor o igual', 'Mayor', 'Mayor'])
```

### `np.where` en DataFrames de Pandas

Esta es la aplicación más habitual en Data Engineering: crear columnas condicionales de forma eficiente.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "A": [1, 2, 3, 4, 5],
    "B": [10, 20, 30, 40, 50]
})

# Crear columna con condición simple
df["C"] = np.where(df["A"] > 3, "Mayor", "Menor o igual")

# Crear columna con condición múltiple
df["D"] = np.where(
    (df["A"] > 3) & (df["B"] < 50),   # condición: A>3 Y B<50
    "Cumple",
    "No cumple"
)

print(df)
#    A   B              C          D
# 0  1  10  Menor o igual  No cumple
# 1  2  20  Menor o igual  No cumple
# 2  3  30  Menor o igual  No cumple
# 3  4  40          Mayor     Cumple   ← A=4>3 y B=40<50 ✓
# 4  5  50          Mayor  No cumple   ← A=5>3 PERO B=50 no es <50
```

> 💡 **¿Por qué `np.where` en lugar de `.apply()` o un `for`?**
> Porque es **vectorizado**: opera sobre todo el array en una sola pasada en C, sin Python overhead. En datasets de millones de filas, la diferencia es de segundos vs minutos.

---

## Bloque 8 — Consumo de APIs {#bloque-8}

### ¿Qué es una API?

Una **API** (*Application Programming Interface*) es un contrato que define cómo dos sistemas se comunican. En el contexto de datos, las APIs web son la forma más común de acceder a datos externos en tiempo real: precios de divisas, clima, datos del gobierno, redes sociales, etc.

```
Flujo de una llamada a una API
──────────────────────────────────────────────────────────────────
  Tu programa          Internet              Servidor API
  ──────────           ────────              ────────────
  requests.get()  →→→  HTTP Request  →→→    Recibe y procesa
                                            la solicitud
                    ←←←  HTTP Response  ←←←  Envía respuesta
  data = r.json()  ←      (JSON)             en formato JSON
──────────────────────────────────────────────────────────────────
```

### Métodos HTTP

Las APIs usan el protocolo HTTP. Los métodos más comunes son:

| Método | Acción | Analogía |
|---|---|---|
| `GET` | Leer / consultar datos | Abrir una página web |
| `POST` | Crear / enviar datos | Enviar un formulario |
| `PUT` | Actualizar datos existentes | Editar un registro |
| `DELETE` | Eliminar datos | Borrar un registro |

En este curso nos enfocaremos en **GET**, que es el más común para consumir datos.

### Códigos de estado HTTP

La respuesta siempre incluye un código de estado que indica si la solicitud fue exitosa:

| Código | Significado |
|---|---|
| `200` | OK — éxito |
| `201` | Created — recurso creado |
| `400` | Bad Request — error en la solicitud |
| `401` | Unauthorized — necesitas autenticarte |
| `404` | Not Found — recurso no encontrado |
| `429` | Too Many Requests — superaste el rate limit |
| `500` | Internal Server Error — error del servidor |

### La librería `requests`

```python
# Instalar (solo una vez)
pip install requests

# Importar
import requests
```

### Primer ejemplo: explorar una respuesta

```python
import requests

url = "https://example.com"
response = requests.get(url)

# Atributos de la respuesta
print(response.status_code)    # 200
print(response.url)             # URL final (después de redirects)
print(response.headers)         # metadatos de la respuesta (dict)
print(response.text)            # contenido como string
print(type(response.text))      # <class 'str'>
```

---

## Bloque 9 — JSON: el formato de las APIs {#bloque-9}

**JSON** (*JavaScript Object Notation*) es el formato estándar para intercambiar datos entre APIs y aplicaciones. Es legible por humanos y fácil de procesar en Python.

```json
{
  "nombre": "Ana",
  "edad": 28,
  "activo": true,
  "cursos": ["Python", "SQL", "Spark"],
  "direccion": {
    "ciudad": "Santiago",
    "pais": "Chile"
  }
}
```

### JSON y Python: correspondencia de tipos

| JSON | Python |
|---|---|
| `object {}` | `dict` |
| `array []` | `list` |
| `string ""` | `str` |
| `number` | `int` / `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

### Trabajar con JSON en Python

```python
import json

# 1. Parsear JSON desde un string
texto_json = '{"nombre": "Ana", "edad": 28}'
datos = json.loads(texto_json)    # string → dict Python
print(datos["nombre"])            # Ana

# 2. Convertir dict Python a JSON
dic = {"nombre": "Ana", "edad": 28}
texto = json.dumps(dic, indent=4) # dict → string JSON formateado

# 3. Desde una respuesta HTTP (el método más común)
response = requests.get(url)
datos = response.json()           # retorna directamente como dict/list
# equivalente a: json.loads(response.text)
```

### Imprimir JSON de forma legible

```python
import json, requests

response = requests.get("https://api.agify.io?name=ana")
datos = response.json()

# Ver la estructura completa de forma indentada
print(json.dumps(datos, indent=4, ensure_ascii=False))
# {
#     "name": "ana",
#     "age": 32,
#     "count": 6543
# }
```

> 💡 **Conexión con la Clase 4 del Módulo 01:** allí viste JSON como formato de ingesta a nivel de pipeline (con `pd.json_normalize` para aplanar). Aquí lo abordamos como un artefacto básico del lenguaje (con `json.loads` / `json.dumps`). Ambas perspectivas son útiles según el contexto.

---

## Bloque 10 — Ejemplos prácticos de APIs {#bloque-10}

### Patrón general para consumir una API

```python
import requests
import json
import pandas as pd

# 1. Definir URL
url = "https://api.ejemplo.com/datos"

# 2. Hacer la solicitud
response = requests.get(url)

# 3. Verificar que fue exitosa
if response.status_code == 200:
    # 4. Parsear el JSON
    data = response.json()

    # 5. Convertir a DataFrame para analizar
    df = pd.DataFrame(data)
else:
    print(f"Error: {response.status_code}")
```

---

### Ejemplo 1: Predecir edad por nombre (agify.io)

```python
import requests

nombre = "ana"
url = f"https://api.agify.io?name={nombre}"
response = requests.get(url)

if response.status_code == 200:
    data = response.json()
    print(f"Nombre:    {data['name']}")
    print(f"Edad pred: {data['age']} años")
    print(f"Muestras:  {data['count']:,}")
# Output:
# Nombre:    ana
# Edad pred: 32 años
# Muestras:  6,543
```

### Ejemplo 2: Feriados de Chile (api.boostr.cl)

```python
import requests
import pandas as pd

url = "https://api.boostr.cl/holidays.json"
response = requests.get(url)
data = response.json()

df = pd.DataFrame(data["data"])
print(df.head())
#          date                     title
# 0  2026-01-01          Año Nuevo
# 1  2026-04-03          Viernes Santo
# ...
```

### Ejemplo 3: Sismos recientes en Chile

```python
import requests
import pandas as pd

url = "https://api.boostr.cl/earthquakes/recent.json"
headers = {"accept": "application/json"}

response = requests.get(url, headers=headers)
data = response.json()

df = pd.DataFrame(data["data"])
print(df[["date", "magnitude", "depth", "place"]].head())
```

### Ejemplo 4: Indicadores económicos (mindicador.cl)

Esta API es especialmente útil para automatizar reportes financieros en Chile.

```python
import requests
import pandas as pd

# Obtener UF del año 2026
indicador = "uf"
year = 2026
url = f"https://mindicador.cl/api/{indicador}/{year}"

response = requests.get(url)
data = response.json()
df_uf = pd.DataFrame(data["serie"])
print(df_uf.head())
#                      fecha    valor
# 0  2026-04-10T00:00:00.000  38245.67
# 1  2026-04-09T00:00:00.000  38240.12
# ...

# Obtener tipo de cambio del dólar
url = f"https://mindicador.cl/api/dolar/{year}"
df_dolar = pd.DataFrame(requests.get(url).json()["serie"])
print(df_dolar.head(5))
```

### Ejemplo 5: Paginación con PokéAPI

Muchas APIs devuelven grandes cantidades de datos en **páginas**. Este ejemplo muestra cómo manejar paginación con el parámetro `limit`:

```python
import requests
import json
import pandas as pd

url = "https://pokeapi.co/api/v2/pokemon?limit=10"
response = requests.get(url)
datos = response.json()

# Ver la estructura del JSON
print(json.dumps(datos, indent=4))
# {
#     "count": 1302,
#     "next": "https://pokeapi.co/api/v2/pokemon?offset=10&limit=10",
#     "results": [
#         {"name": "bulbasaur", "url": "..."},
#         {"name": "ivysaur", "url": "..."},
#         ...
#     ]
# }

# Extraer solo los resultados y convertir a DataFrame
df = pd.DataFrame(datos["results"])
print(df)
#          name                             url
# 0   bulbasaur  https://pokeapi.co/api/v2/...
# 1     ivysaur  https://pokeapi.co/api/v2/...
# ...
```

> 💡 **Patrón de paginación:** observa los campos `next` y `previous` en la respuesta. Puedes construir un bucle que siga consumiendo páginas hasta que `next` sea `null`. Esta es una de las técnicas más usadas para extraer grandes catálogos completos.

### Manejo de errores y buenas prácticas

```python
import requests
import time

def consultar_api(url, reintentos=3, espera=2):
    """
    Consulta una URL con reintentos automáticos ante fallos.

    Parámetros:
    url (str): URL del endpoint
    reintentos (int): Número máximo de intentos
    espera (int): Segundos entre intentos

    Retorna:
    dict: Datos JSON o None si falla
    """
    for intento in range(1, reintentos + 1):
        try:
            response = requests.get(url, timeout=10)

            if response.status_code == 200:
                return response.json()
            elif response.status_code == 429:    # Too Many Requests
                print(f"Rate limit alcanzado. Esperando {espera}s...")
                time.sleep(espera)
            else:
                print(f"Error {response.status_code} en intento {intento}")
        except requests.exceptions.ConnectionError:
            print(f"Sin conexión. Intento {intento}/{reintentos}")
            time.sleep(espera)

    return None

# Usar la función
datos = consultar_api("https://mindicador.cl/api/dolar/2026")
if datos:
    df = pd.DataFrame(datos["serie"])
    print(df.head())
```

> 💡 **Contexto DE:** En pipelines de producción, el manejo de errores en llamadas a APIs es crítico. Las APIs pueden fallar por problemas de red, límites de uso (*rate limits*) o mantenimiento del servidor. Un pipeline bien construido incorpora reintentos, *logging* de errores y alertas cuando una fuente de datos no responde.

---

<details>
<summary><strong>🟢 Ejercicio 1 — Archivos (click para ver)</strong></summary>

**Enunciado:**
1. Crea una lista de 5 ciudades chilenas y guárdalas en un archivo `ciudades.txt`, una por línea.
2. Lee el archivo e imprime cada ciudad con su número de línea (ej: `1. Santiago`).
3. Agrega 2 ciudades más al archivo sin borrar las anteriores.

**Solución:**
```python
# 1. Escribir ciudades
ciudades = ["Santiago", "Valparaíso", "Concepción", "Temuco", "Antofagasta"]

with open("ciudades.txt", "w", encoding="utf-8") as f:
    for ciudad in ciudades:
        f.write(ciudad + "\n")

# 2. Leer e imprimir con número de línea
with open("ciudades.txt", "r", encoding="utf-8") as f:
    for i, linea in enumerate(f, start=1):
        print(f"{i}. {linea.strip()}")

# 3. Agregar más ciudades
nuevas = ["Puerto Montt", "La Serena"]
with open("ciudades.txt", "a", encoding="utf-8") as f:
    for ciudad in nuevas:
        f.write(ciudad + "\n")
```

</details>

<details>
<summary><strong>🟢 Ejercicio 2 — NumPy (click para ver)</strong></summary>

**Enunciado:**
1. Crea un array con los números del 1 al 20.
2. Extrae los elementos en posiciones pares (0, 2, 4...).
3. Calcula la media, desviación estándar y suma total.
4. Crea una matriz 4×5 con números del 0 al 19 usando `arange` + `reshape`.
5. Usando `np.where`, clasifica cada elemento como "par" o "impar".

**Solución:**
```python
import numpy as np

# 1. Array 1 al 20
arr = np.arange(1, 21)

# 2. Elementos en posiciones pares (slicing con paso 2)
print(arr[::2])    # array([1, 3, 5, 7, ..., 19])

# 3. Estadísticas
print(f"Media: {arr.mean():.2f}")  # 10.5
print(f"Std:   {arr.std():.2f}")   # 5.77
print(f"Suma:  {arr.sum()}")       # 210

# 4. Reshape a 4×5
matriz = np.arange(20).reshape((4, 5))
print(matriz)

# 5. Clasificar par/impar
etiquetas = np.where(arr % 2 == 0, "par", "impar")
print(etiquetas)
```

</details>

<details>
<summary><strong>🟢 Ejercicio 3 — APIs (click para ver)</strong></summary>

**Enunciado:**
1. Consulta la API `https://api.agify.io?name=TU_NOMBRE` e imprime la edad predicha.
2. Consulta los feriados de Chile desde `https://api.boostr.cl/holidays.json` y crea un DataFrame.
3. Filtra solo los feriados del mes de septiembre.

**Solución:**
```python
import requests
import pandas as pd

# 1. Edad predicha
nombre = "ana"   # cambia por tu nombre
response = requests.get(f"https://api.agify.io?name={nombre}")
if response.status_code == 200:
    data = response.json()
    print(f"Edad predicha para '{nombre}': {data['age']} años")

# 2. Feriados de Chile
response = requests.get("https://api.boostr.cl/holidays.json")
data = response.json()
df = pd.DataFrame(data["data"])
print(df.head())

# 3. Filtrar septiembre
# La columna date tiene formato "YYYY-MM-DD"
df["mes"] = df["date"].str[5:7]    # extraer el mes como string
septiembre = df[df["mes"] == "09"]
print(septiembre[["date", "title"]])
```

</details>

---

## Referencia rápida — Archivos, NumPy y APIs

```
ARCHIVOS
─────────────────────────────────────────────────────────────────
  open("archivo.txt", "r/w/a", encoding="utf-8")
  archivo.read()          → todo como string
  archivo.readlines()     → lista de líneas
  archivo.readline()      → una línea a la vez
  archivo.write("texto")  → escribir
  with open(...) as f:    → cierra automáticamente (recomendado)

CSV
─────────────────────────────────────────────────────────────────
  import csv
  csv.reader(archivo)               → iterable de listas
  csv.DictReader(archivo)           → iterable de dicts
  csv.writer(archivo).writerows([]) → escribir filas

NUMPY
─────────────────────────────────────────────────────────────────
  np.array([...])                   → crear array
  np.zeros((f,c))  np.ones((f,c))  np.eye(n)
  np.arange(inicio, fin)            → secuencia entera
  np.linspace(a, b, n)              → n valores equiespaciados
  arr.shape  arr.ndim  arr.dtype  arr.size
  arr.reshape((f,c))  arr.T         → reformar / transponer
  arr.astype(np.float64)            → cambiar dtype
  arr[i]  arr[i, j]  arr[:, j]      → indexing
  np.sum/mean/std/min/max(arr)      → estadísticas
  np.sqrt/exp/log/abs(arr)          → ufuncs
  np.where(cond, val_T, val_F)      → condicional vectorizado
  np.maximum(x, y)                  → máximo elemento a elemento

APIS
─────────────────────────────────────────────────────────────────
  import requests
  response = requests.get(url)      → llamada GET
  response.status_code              → 200 = éxito
  response.text                     → contenido como string
  response.json()                   → contenido como dict/list

JSON
─────────────────────────────────────────────────────────────────
  import json
  json.loads(texto)                 → string → dict/list Python
  json.dumps(datos, indent=4)       → dict/list → string JSON
  response.json()                   → equivalente a json.loads
```

---

*→ Volver al [Módulo 02: Programación en Python](../README.md)*

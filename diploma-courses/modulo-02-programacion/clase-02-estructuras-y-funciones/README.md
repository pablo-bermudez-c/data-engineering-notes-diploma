# Módulo 2 — Estructuras de Control y Funciones

---

## Tabla de contenidos

1. [Condicionales: `if`, `elif`, `else`](#bloque-1)
2. [Operadores de comparación y lógicos](#bloque-2)
3. [Bucle `for`](#bloque-3)
4. [Bucle `while`](#bloque-4)
5. [Control de flujo: `break`, `continue`](#bloque-5)
6. [Funciones](#bloque-6)
7. [Recursión](#bloque-7)
8. [Ámbito de variables (Scope)](#bloque-8)

---

## Bloque 1 — Condicionales: `if`, `elif`, `else` {#bloque-1}

Las estructuras condicionales permiten que el programa **tome decisiones** y ejecute bloques de código distintos según si una condición es verdadera o falsa.

```
Flujo de un condicional if/elif/else
──────────────────────────────────────────────────
              ┌──────────────────────┐
              │       condición      │
              └───┬───────────────┬──┘
              True│          False│
                  ▼               ▼
          ┌──────────────┐  ┌──────────────┐
          │  bloque if   │  │ otro elif/   │
          └──────────────┘  │    else      │
                            └──────────────┘
──────────────────────────────────────────────────
```

### Sintaxis

```python
if condición:
    # código si la condición es True
elif otra_condición:
    # código si la condición anterior es False, pero esta es True
else:
    # código si ninguna condición anterior se cumple
```

> ⚠️ **Indentación obligatoria:** Python usa la indentación (4 espacios) para definir bloques de código. A diferencia de otros lenguajes, **no usa llaves `{}`**. Una indentación incorrecta produce `IndentationError`.

---

### `if` simple

```python
edad = 20

if edad >= 18:
    print("Eres mayor de edad")
# Si edad < 18, no se imprime nada
```

### `if` + `else`

```python
edad = 15

if edad >= 18:
    print("Eres mayor de edad")
else:
    print("Eres menor de edad")
```

### `if` + `elif` + `else`

```python
nota = 4

if nota >= 6:
    print("Excelente")
elif nota >= 4:
    print("Aprobado")
else:
    print("Reprobado")
```

### Múltiples `elif`

```python
dia = 3

if dia == 1:
    print("Lunes")
elif dia == 2:
    print("Martes")
elif dia == 3:
    print("Miércoles")
else:
    print("Otro día")
# Output: Miércoles
```

### Condicionales anidados

```python
temperatura = 35
humedad = 80

if temperatura > 30:
    if humedad > 70:
        print("Día muy caluroso y húmedo")
    else:
        print("Día caluroso")
elif 20 <= temperatura <= 30:   # rango: operador encadenado
    print("Día agradable")
else:
    print("Día frío")
# Output: Día muy caluroso y húmedo
```

> 💡 **Operador encadenado:** `20 <= temperatura <= 30` es equivalente a `temperatura >= 20 and temperatura <= 30`. Python permite esta sintaxis más legible.

---

### Diferencia clave: `if` encadenado vs `elif`

Esta distinción es importante y suele generar confusión:

```python
a = 15

# Con if encadenado: TODAS las condiciones se evalúan siempre
if a % 5 == 0:
    print(f"{a} es divisible entre 5")   # se imprime
if a % 3 == 0:
    print(f"{a} es divisible entre 3")   # también se imprime
# Output: "15 es divisible entre 5" Y "15 es divisible entre 3"

# Con elif: en cuanto una condición es True, el resto NO se evalúa
if a % 5 == 0:
    print(f"{a} es divisible entre 5")   # se imprime
elif a % 3 == 0:
    print(f"{a} es divisible entre 3")   # no se evalúa siquiera
# Output: solo "15 es divisible entre 5"
```

> 💡 **Contexto DE:** En pipelines de validación de datos, la diferencia entre `if` y `elif` determina cuántas validaciones se ejecutan sobre cada registro. Usar `elif` cuando las condiciones son mutuamente excluyentes hace el código más eficiente y predecible.

---

## Bloque 2 — Operadores de comparación y lógicos {#bloque-2}

### Operadores de comparación

Retornan siempre `True` o `False`:

| Operador | Descripción | Ejemplo | Resultado |
|---|---|---|---|
| `==` | Igual a | `5 == 5` | `True` |
| `!=` | Distinto de | `5 != 3` | `True` |
| `>` | Mayor que | `5 > 3` | `True` |
| `<` | Menor que | `5 < 3` | `False` |
| `>=` | Mayor o igual que | `5 >= 5` | `True` |
| `<=` | Menor o igual que | `3 <= 5` | `True` |

### Operadores lógicos

Combinan condiciones booleanas:

| Operador | Nombre | Descripción | Ejemplo | Resultado |
|---|---|---|---|---|
| `and` | Y lógico | True solo si **ambas** condiciones son True | `True and False` | `False` |
| `or` | O lógico | True si **al menos una** condición es True | `True or False` | `True` |
| `not` | Negación | Invierte el valor booleano | `not True` | `False` |

```python
nota = 4
asistencia = 60

# and: ambas condiciones deben cumplirse
if nota >= 4 and asistencia >= 75:
    print("Aprobado")
else:
    print("Reprobado")
# Output: Reprobado (nota >= 4 es True, pero asistencia >= 75 es False)

# or: basta que una condición se cumpla
if nota >= 6 or asistencia == 100:
    print("Distinción")
else:
    print("Sin distinción")

# not: invierte la condición
activo = False
if not activo:
    print("Usuario inactivo")
```

> ⚠️ **Error común:** Confundir `=` (asignación) con `==` (comparación) dentro de un `if`. `if x = 5:` produce `SyntaxError`. La comparación correcta es `if x == 5:`.

---

## Bloque 3 — Bucle `for` {#bloque-3}

El bucle `for` recorre los elementos de una **secuencia** (lista, string, rango, diccionario, etc.) y ejecuta un bloque de código por cada elemento.

```
Flujo del bucle for
────────────────────────────────────────────
  secuencia = [a, b, c, d]

  ┌──────────────────────────┐
  │ for elemento in secuencia│
  └────────────┬─────────────┘
               │ ¿quedan elementos?
         Sí ◄──┴──► No → Sale del bucle
         │
         ▼
  ┌──────────────┐
  │  bloque for  │
  └──────────────┘
         │
         └──► siguiente elemento
────────────────────────────────────────────
```

### Sintaxis

```python
for variable in secuencia:
    # código a ejecutar en cada iteración
```

### Recorrer listas y strings

```python
# Lista
ropas = ["pantalon", "polera", "zapato"]
for ropa in ropas:
    print(ropa)
# Output: pantalon / polera / zapato

# String: itera carácter por carácter
for letra in "Python":
    print(letra)
# Output: P / y / t / h / o / n
```

### `range()` — generar secuencias numéricas

```python
# range(fin)           → 0, 1, ..., fin-1
# range(inicio, fin)   → inicio, ..., fin-1
# range(inicio, fin, paso)

for i in range(3):
    print("Iteración:", i)
# Output: 0, 1, 2

for i in range(2, 11):
    print(i)
# Output: 2, 3, 4, ..., 10

for i in range(0, 11, 2):
    print(i)
# Output: 0, 2, 4, 6, 8, 10 (de 2 en 2)

for i in range(10, 0, -1):
    print(i)
# Output: 10, 9, 8, ..., 1 (cuenta regresiva)
```

### `enumerate()` — índice + valor

Cuando necesitamos el índice y el valor al mismo tiempo:

```python
nombres = ["Ana", "Luis", "Pedro"]

for indice, nombre in enumerate(nombres):
    print(indice, nombre)
# Output:
# 0 Ana
# 1 Luis
# 2 Pedro
```

> 💡 **Contexto DE:** `enumerate()` es muy útil cuando se procesan columnas de un archivo y se necesita conocer la posición de cada campo, o cuando se registra el número de fila junto con su contenido en un log.

### `for` con diccionarios

```python
precios = {"manzana": 100, "banana": 200, "cereza": 300}

# Solo claves (comportamiento por defecto)
for fruta in precios:
    print(fruta)

# Claves y valores juntos con .items()
for fruta, precio in precios.items():
    print(f"{fruta} cuesta {precio}")
# Output: manzana cuesta 100 / banana cuesta 200 / ...
```

### Patrones comunes con `for`

```python
# Acumular suma
numeros = [10, 20, 30, 40]
total = 0
for n in numeros:
    total += n
print("Total:", total)   # 100

# Filtrar elementos
notas = [45, 67, 34, 55, 10]
for nota in notas:
    if nota >= 40:
        print("Aprobado:", nota)
    else:
        print("Reprobado:", nota)

# Construir una nueva lista
cuadrados = []
for i in range(1, 6):
    cuadrados.append(i ** 2)
print(cuadrados)   # [1, 4, 9, 16, 25]
```

### Dibujar figuras con `for` anidados

```python
# Triángulo de asteriscos
for i in range(1, 6):
    print("*" * i)
# Output:
# *
# **
# ***
# ****
# *****
```

> ⚠️ **Error común:** Modificar una lista mientras se la recorre con `for` puede producir resultados inesperados. Si necesitas filtrar o transformar, construye una nueva lista en lugar de modificar la original.

---

## Bloque 4 — Bucle `while` {#bloque-4}

El bucle `while` ejecuta un bloque de código **mientras una condición sea verdadera**. Se usa cuando no se sabe de antemano cuántas iteraciones se necesitan.

```
Flujo del bucle while
────────────────────────────────────────────
  ┌──────────────────┐
  │ while condición: │◄──────┐
  └────────┬─────────┘       │
           │                 │
      True │     False       │
           ▼        └─► Sale del bucle
  ┌──────────────┐
  │ bloque while │
  └──────────────┘
           │
           └────────────────►(vuelve a evaluar condición)
────────────────────────────────────────────
```

### Sintaxis

```python
while condición:
    # código a ejecutar mientras la condición sea True
```

### Ejemplos básicos

```python
# Contador simple
contador = 3
while contador <= 5:
    print("Número:", contador)
    contador += 1   # ⚠️ sin esto → bucle infinito
# Output: Número: 3 / Número: 4 / Número: 5

# Condición booleana
sigue = True
n = 0
while sigue:
    n += 1
    print("Vuelta", n)
    if n == 3:
        sigue = False
# Output: Vuelta 1 / Vuelta 2 / Vuelta 3
```

> ⚠️ **Bucle infinito:** Si la condición del `while` nunca se vuelve `False`, el programa se ejecutará indefinidamente. Siempre asegurarse de que algo dentro del bucle **modifique la condición** para que eventualmente sea `False`.

---

## Bloque 5 — Control de flujo: `break` y `continue` {#bloque-5}

Estas palabras clave permiten controlar el flujo de ejecución **dentro** de un bucle.

| Palabra clave | Efecto |
|---|---|
| `break` | **Interrumpe** el bucle completamente y sale de él |
| `continue` | **Salta** la iteración actual y avanza a la siguiente |

### `break` — salir del bucle

```python
# Bucle infinito controlado con break
while True:
    respuesta = input("Escribe 'salir' para terminar: ")
    if respuesta == "salir":
        print("Programa finalizado")
        break
    print("Escribiste:", respuesta)
```

```python
# break en un for: detener la búsqueda al encontrar el elemento
archivos = ["datos_enero.csv", "datos_febrero.csv", "ERROR.log", "datos_marzo.csv"]

for archivo in archivos:
    if "ERROR" in archivo:
        print(f"Archivo problemático encontrado: {archivo}")
        break   # detenemos el procesamiento
    print(f"Procesando: {archivo}")
```

### `continue` — saltar iteración

```python
# Saltar el número 3
numero = 0
while numero < 5:
    numero += 1
    if numero == 3:
        continue    # vuelve al inicio del while sin ejecutar el print
    print("Número:", numero)
# Output: 1, 2, 4, 5  (el 3 no aparece)
```

```python
# continue en un for: ignorar registros vacíos
registros = ["Ana", "", "Luis", None, "Pedro", ""]

for registro in registros:
    if not registro:    # si es vacío o None
        continue        # saltar este registro
    print("Procesando:", registro)
# Output: Ana / Luis / Pedro
```

> 💡 **Contexto DE:** `break` y `continue` son muy usados en pipelines de procesamiento. `continue` sirve para **saltar registros inválidos o vacíos** sin detener el proceso. `break` es útil para **detener la carga** al encontrar un error crítico o al alcanzar un límite de registros.

```
Diferencia visual entre break y continue
──────────────────────────────────────────────────────────────
  Secuencia: [1, 2, 3, 4, 5]  → condición en elemento 3

  break:                    continue:
  1 ✔                       1 ✔
  2 ✔                       2 ✔
  3 → DETIENE TODO          3 → SALTA ESTE, sigue
  (4 y 5 nunca se ven)      4 ✔
                            5 ✔
──────────────────────────────────────────────────────────────
```

---

## Bloque 6 — Funciones {#bloque-6}

Una función es un **bloque de código reutilizable** al que se le asigna un nombre. Permite dividir un programa en partes independientes, cada una con una responsabilidad clara.

```
Anatomía de una función
────────────────────────────────────────────────────────
  def nombre_funcion(param1, param2):
  ──┬─────────┬───────────┬─────────────────────────────
    │         │           └── Parámetros (entrada)
    │         └──────────────  Nombre (snake_case)
    └────────────────────────  Palabra reservada

      """Docstring: descripción de la función"""
      ──────────────────────────────────────────── Documentación

      # cuerpo de la función
      resultado = param1 + param2
      return resultado
      ────────┬────────
              └── Valor de retorno (salida)
────────────────────────────────────────────────────────
```

### Sintaxis básica

```python
def nombre_funcion(parámetros):
    """Docstring: descripción de la función"""
    # bloque de código
    return valor
```

### Formas de llamar a una función

```python
# 1. Desde el espacio de trabajo principal
print("hola")          # función built-in
len([1, 2, 3])         # función built-in

# 2. Desde un objeto (método)
"hola".upper()

# 3. Desde un módulo
import math
math.sqrt(16)

# 4. Funciones definidas por el usuario
def saludar():
    print("¡Hola, bienvenido!")

saludar()   # llamada a la función
```

---

### 6.1 Función sin parámetros ni retorno

```python
def saludar():
    print("¡Hola, bienvenido a la clase de funciones en Python!")

# Llamada
saludar()
# Output: ¡Hola, bienvenido a la clase de funciones en Python!
```

### 6.2 Función con parámetros

Los **parámetros** son variables que reciben valores al llamar la función. Los valores entregados en la llamada se llaman **argumentos**.

```python
def saludar_persona(nombre):
    print(f"Hola {nombre}, ¡bienvenido a la clase de Python!")

# Llamadas con distintos argumentos
saludar_persona("Ana")     # Hola Ana, ¡bienvenido!
saludar_persona("Carlos")  # Hola Carlos, ¡bienvenido!
saludar_persona("Pedro")   # Hola Pedro, ¡bienvenido!
```

### 6.3 Función con valor de retorno (`return`)

`return` devuelve un valor al lugar donde se llamó la función. Una función sin `return` devuelve `None` implícitamente.

```python
def sumar(a, b):
    return a + b

resultado = sumar(5, 7)
print("La suma es:", resultado)   # La suma es: 12

# El valor retornado puede usarse directamente en una expresión
print(sumar(10, 20) * 2)   # 60
```

### 6.4 Parámetros con valores por defecto

Se asigna un valor que se usa cuando el argumento **no se pasa** en la llamada.

```python
def saludar(nombre="invitado"):
    print(f"Hola {nombre}!")

saludar()           # Hola invitado!  (usa el valor por defecto)
saludar("María")    # Hola María!     (usa el valor pasado)
```

> ⚠️ **Regla:** Los parámetros con valor por defecto deben ir **después** de los parámetros sin valor por defecto.

```python
# ✅ Correcto
def registrar(nombre, ciudad="Santiago"):
    print(f"{nombre} - {ciudad}")

# ❌ Incorrecto → SyntaxError
# def registrar(ciudad="Santiago", nombre):
#     ...
```

### 6.5 Múltiples valores de retorno

Python permite retornar más de un valor en forma de **tupla**:

```python
def operaciones(a, b):
    suma  = a + b
    resta = a - b
    return suma, resta          # retorna una tupla implícita

# Desempaquetar los valores al llamar
x, y = operaciones(10, 6)
print("Suma:", x)    # 16
print("Resta:", y)   # 4

# También se puede guardar como tupla completa
resultado = operaciones(10, 6)
print(resultado)    # (16, 4)
```

### 6.6 `*args` — número variable de argumentos posicionales

Permite pasar **cualquier cantidad** de argumentos; Python los agrupa en una **tupla**.

```python
def sumar_todos(*args):
    return sum(args)

print(sumar_todos(1, 2, 3))           # 6
print(sumar_todos(1, 2, 3, 4, 5))     # 15
print(sumar_todos(1, 2, 3, 4, 5, 6, 7, 8, 9, 10))  # 55
```

### 6.7 `**kwargs` — número variable de argumentos con nombre

Permite pasar argumentos con nombre (`clave=valor`); Python los agrupa en un **diccionario**.

```python
def mostrar_info(**kwargs):
    for clave, valor in kwargs.items():
        print(f"{clave}: {valor}")

mostrar_info(nombre="Juan", edad=25, ciudad="Santiago")
# Output:
# nombre: Juan
# edad: 25
# ciudad: Santiago
```

> 💡 **Contexto DE:** `**kwargs` es muy común en funciones que construyen queries SQL dinámicos o configuran conexiones a bases de datos, donde los parámetros pueden variar según el motor (PostgreSQL, MySQL, Redshift, etc.).

```python
# Ejemplo aplicado: función de conexión flexible
def conectar_db(**kwargs):
    print("Conectando con los siguientes parámetros:")
    for k, v in kwargs.items():
        print(f"  {k} = {v}")

conectar_db(host="localhost", port=5432, database="ventas", user="admin")
```

### 6.8 Funciones lambda (anónimas)

Las funciones lambda son funciones pequeñas de **una sola línea** sin nombre asignado explícitamente. Son útiles cuando se necesita una función simple de forma puntual.

**Sintaxis:**
```python
lambda parámetros: expresión
```

```python
# Función normal
def cuadrado(x):
    return x ** 2

# Equivalente como lambda
cuadrado = lambda x: x ** 2
print(cuadrado(5))    # 25

# Lambda con múltiples parámetros
suma = lambda a, b: a + b
print(suma(3, 7))     # 10
```

> 💡 **Contexto DE:** Las lambdas son especialmente útiles junto a funciones como `sorted()`, `map()` y `filter()`, que son muy comunes al transformar datos antes de cargarlos en una base de datos.

```python
# Ordenar una lista de diccionarios por un campo específico
registros = [
    {"nombre": "Carlos", "ventas": 150},
    {"nombre": "Ana",    "ventas": 320},
    {"nombre": "Luis",   "ventas": 90}
]

# Ordenar por ventas usando lambda
ordenados = sorted(registros, key=lambda r: r["ventas"], reverse=True)
for r in ordenados:
    print(r["nombre"], "→", r["ventas"])
# Output: Ana → 320 / Carlos → 150 / Luis → 90
```

### 6.9 Documentación de funciones (Docstrings)

Un docstring es un string en la primera línea del cuerpo de la función que documenta su propósito, parámetros y retorno. Es una **buena práctica obligatoria** en código de producción.

```python
def multiplicar(a, b):
    """
    Multiplica dos números y devuelve el resultado.

    Parámetros:
    a (int, float): Primer número
    b (int, float): Segundo número

    Retorna:
    int, float: El producto de a y b
    """
    return a * b

# Consultar la documentación de la función
help(multiplicar)
```

---

### Buenas prácticas en funciones

| Práctica | Descripción |
|---|---|
| **Nombres descriptivos** | `calcular_promedio()` en lugar de `cp()` |
| **Una función = una tarea** | Cada función debe hacer una sola cosa y hacerla bien |
| **Docstrings** | Documentar propósito, parámetros y retorno |
| **Evitar variables globales** | Pasar y retornar valores en lugar de modificar globales |
| **Tamaño razonable** | Si una función supera ~20 líneas, considerar dividirla |

> 💡 **Contexto DE:** En pipelines de datos, cada etapa suele encapsularse en una función: `extraer_datos()`, `limpiar_registros()`, `transformar_fechas()`, `cargar_en_db()`. Esto facilita el testing, la reutilización y el mantenimiento del código.

---

<details>
<summary><strong>🟢 Ejercicio 1 — Condicionales (click para ver)</strong></summary>

**Enunciado:**
1. Crear un programa que reciba una temperatura e imprima "Hace frío" (< 15), "Está templado" (15-25) o "Hace calor" (> 25).
2. Crear un programa que reciba una edad e imprima la categoría: "Niño" (< 12), "Adolescente" (12-17) o "Adulto" (≥ 18).

**Solución:**
```python
# 1. Temperatura
temperatura = float(input("Ingresa la temperatura: "))

if temperatura < 15:
    print("Hace frío")
elif temperatura <= 25:
    print("Está templado")
else:
    print("Hace calor")

# 2. Categoría por edad
edad = int(input("Ingresa la edad: "))

if edad < 12:
    print("Niño")
elif edad <= 17:
    print("Adolescente")
else:
    print("Adulto")
```

</details>

<details>
<summary><strong>🟢 Ejercicio 2 — Bucles (click para ver)</strong></summary>

**Enunciado:**
3. Dada la lista `palabras = ["python", "data", "ai", "cloud"]`, imprimir cada palabra y su cantidad de letras.
4. Dada la lista `notas = [45, 67, 34, 55, 10]`, imprimir "Aprobado" si la nota ≥ 40 y "Reprobado" si es menor.
5. Mostrar la tabla de multiplicar de un número ingresado por el usuario (del 1 al 10) usando `while`.

**Solución:**
```python
# 3. Palabras y su longitud
palabras = ["python", "data", "ai", "cloud"]
for palabra in palabras:
    print(f"{palabra}: {len(palabra)} letras")

# 4. Notas aprobadas/reprobadas
notas = [45, 67, 34, 55, 10]
for nota in notas:
    if nota >= 40:
        print(f"{nota} → Aprobado")
    else:
        print(f"{nota} → Reprobado")

# 5. Tabla de multiplicar con while
numero = int(input("Ingresa un número: "))
i = 1
while i <= 10:
    print(f"{numero} x {i} = {numero * i}")
    i += 1
```

</details>

<details>
<summary><strong>🟢 Ejercicio 3 — Funciones (click para ver)</strong></summary>

**Enunciado:**
6. Función que recibe un número y devuelve `True` si es par, `False` si es impar.
7. Función que recibe una lista de notas y devuelve el promedio, la nota más alta, la más baja, y si el alumno aprueba (promedio ≥ 40).

**Solución:**
```python
# 6. Paridad
def es_par(numero):
    """Devuelve True si el número es par, False si es impar."""
    return numero % 2 == 0

print(es_par(4))    # True
print(es_par(7))    # False

# 7. Análisis de notas
def analizar_notas(notas):
    """
    Recibe una lista de notas y devuelve estadísticas básicas.

    Parámetros:
    notas (list): Lista de notas numéricas

    Retorna:
    tuple: (promedio, máxima, mínima, aprueba)
    """
    promedio = sum(notas) / len(notas)
    maxima   = max(notas)
    minima   = min(notas)
    aprueba  = promedio >= 40

    return promedio, maxima, minima, aprueba

prom, alta, baja, aprueba = analizar_notas([45, 67, 34, 55, 10])
print(f"Promedio: {prom:.1f}")
print(f"Nota más alta: {alta}")
print(f"Nota más baja: {baja}")
print(f"¿Aprueba? {'Sí' if aprueba else 'No'}")
```

</details>

<details>
<summary><strong>🟢 Ejercicio 4 — Calculadora con funciones (click para ver)</strong></summary>

**Enunciado:**
Función que recibe dos números y una operación (`+`, `-`, `*`, `/`) y devuelve el resultado.

**Solución:**
```python
def calculadora(num1, num2, operacion):
    """
    Realiza una operación aritmética entre dos números.

    Parámetros:
    num1 (float): Primer operando
    num2 (float): Segundo operando
    operacion (str): Operador: '+', '-', '*', '/'

    Retorna:
    float: Resultado de la operación, o None si la operación es inválida
    """
    if operacion == "+":
        return num1 + num2
    elif operacion == "-":
        return num1 - num2
    elif operacion == "*":
        return num1 * num2
    elif operacion == "/":
        if num2 == 0:
            print("Error: no se puede dividir por cero")
            return None
        return num1 / num2
    else:
        print("Operación no válida")
        return None

print(calculadora(10, 3, "+"))    # 13
print(calculadora(10, 3, "/"))    # 3.333...
print(calculadora(10, 0, "/"))    # Error: no se puede dividir por cero
```

</details>

---

## Bloque 7 — Recursión {#bloque-7}

La recursión es una técnica en la que una **función se llama a sí misma** para resolver un problema. Se usa cuando un problema puede dividirse en **subproblemas más pequeños del mismo tipo**.

```
Estructura de una función recursiva
────────────────────────────────────────────────────────
  def funcion_recursiva(n):
      if caso_base:         ← condición que DETIENE la recursión
          return valor_base
      else:
          return funcion_recursiva(n - 1)  ← llamada más simple
────────────────────────────────────────────────────────
  ⚠️ Sin caso base → bucle infinito → RecursionError
```

Toda función recursiva debe tener:
- **Caso base:** condición que detiene la recursión (sin él, loop infinito)
- **Caso recursivo:** la función se llama a sí misma con una versión simplificada del problema

### Ejemplo básico: cuenta regresiva

```python
def cuenta_regresiva(n):
    if n == 0:           # Caso base: detener aquí
        print("¡Despegue!")
    else:                # Caso recursivo
        print(n)
        cuenta_regresiva(n - 1)   # llamada con n más pequeño

cuenta_regresiva(5)
# Output: 5 / 4 / 3 / 2 / 1 / ¡Despegue!
```

```
Pila de llamadas para cuenta_regresiva(3)
──────────────────────────────────────────────
  cuenta_regresiva(3) → imprime 3
    cuenta_regresiva(2) → imprime 2
      cuenta_regresiva(1) → imprime 1
        cuenta_regresiva(0) → imprime ¡Despegue!
        ← retorna
      ← retorna
    ← retorna
  ← retorna
──────────────────────────────────────────────
```

### Ejemplo clásico: factorial

El factorial de `n` se define como `n! = n × (n-1) × (n-2) × ... × 1`:

```python
def factorial(n):
    if n == 0 or n == 1:   # Caso base
        return 1
    else:                   # Caso recursivo
        return n * factorial(n - 1)

print(factorial(5))   # 120
# Expansión: 5 × 4 × 3 × 2 × 1 = 120
```

### Ventajas, desventajas y límites

| Aspecto | Detalle |
|---|---|
| ✅ Código más simple | Ideal para problemas con estructura jerárquica o divisible |
| ✅ Útil en estructuras anidadas | Directorios, árboles, grafos |
| ❌ Mayor consumo de memoria | Cada llamada ocupa espacio en la pila del sistema |
| ❌ Puede ser más lento | Que una solución iterativa equivalente |
| ⚠️ Límite por defecto | Python permite ~1000 llamadas recursivas (`RecursionError` si se supera) |

> 💡 **Contexto DE:** Aunque la recursión raramente se usa en pipelines de producción (donde se prefiere la iteración por eficiencia), es fundamental para **recorrer estructuras JSON anidadas**, **explorar directorios** de archivos recursivamente, o trabajar con **árboles de dependencias** entre tareas.

> ⚠️ **Regla práctica:** Si el problema puede resolverse fácilmente con un `for` o `while`, preferir la solución iterativa. La recursión se justifica cuando la estructura del problema es naturalmente recursiva (ej. árbol de carpetas, datos JSON multinivel).

---

## Bloque 8 — Ámbito de variables (Scope) {#bloque-8}

El **scope** determina en qué partes del código una variable es accesible. Python sigue la regla **LEGB**: Local → Enclosing → Global → Built-in.

```
Niveles de scope en Python
────────────────────────────────────────────────────────
  ┌─────────────────────────────────────────┐
  │  Built-in: print, len, range, ...       │  más externo
  │  ┌───────────────────────────────────┐  │
  │  │  Global: variables del script     │  │
  │  │  ┌─────────────────────────────┐  │  │
  │  │  │  Local: variables de la fn  │  │  │  más interno
  │  │  └─────────────────────────────┘  │  │
  │  └───────────────────────────────────┘  │
  └─────────────────────────────────────────┘
────────────────────────────────────────────────────────
```

### Ámbito Local

Las variables definidas **dentro** de una función solo existen en esa función. No son accesibles desde fuera.

```python
def mi_funcion():
    variable_local = "Soy una variable local."
    print(variable_local)   # ✅ accesible aquí

mi_funcion()
# print(variable_local)   # ❌ NameError: no existe fuera de la función
```

### Ámbito Global

Las variables definidas **fuera** de cualquier función son globales y accesibles desde cualquier parte del script.

```python
variable_global = "Soy una variable global."

def leer_global():
    print(variable_global)   # ✅ se puede leer sin problema

leer_global()

# Para MODIFICAR una variable global dentro de una función,
# se debe declarar con la palabra clave 'global'
def modificar_global():
    global variable_global
    variable_global = "¡He sido modificada!"

print("Antes:", variable_global)
modificar_global()
print("Después:", variable_global)
```

> ⚠️ **Buena práctica:** Evitar modificar variables globales dentro de funciones. Puede generar **efectos secundarios difíciles de depurar**. Lo correcto es pasar el valor como parámetro y retornar el resultado.

```python
# ❌ Mal: modifica estado global
total = 0
def agregar(valor):
    global total
    total += valor

# ✅ Bien: recibe y retorna sin efectos secundarios
def agregar(total, valor):
    return total + valor

total = 0
total = agregar(total, 10)
total = agregar(total, 20)
print(total)   # 30
```

---

## Referencia rápida — Estructuras de Control y Funciones

```
CONDICIONALES
─────────────────────────────────────────────────────
  if condición:          # evalúa siempre
  elif otra_condición:   # evalúa solo si el if falló
  else:                  # ejecuta si todo lo anterior falló

OPERADORES LÓGICOS
─────────────────────────────────────────────────────
  and   → True si AMBAS condiciones son True
  or    → True si AL MENOS UNA condición es True
  not   → invierte el valor booleano

BUCLES
─────────────────────────────────────────────────────
  for x in secuencia:    # itera sobre una secuencia
  for i in range(n):     # itera n veces (0 a n-1)
  while condición:       # itera mientras sea True

CONTROL DE FLUJO
─────────────────────────────────────────────────────
  break      → sale del bucle completamente
  continue   → salta la iteración actual

FUNCIONES
─────────────────────────────────────────────────────
  def nombre(params):          # definición
      """Docstring"""
      return valor              # retorno

  def fn(a, b="default"):      # parámetro con valor por defecto
  def fn(*args):               # argumentos posicionales variables (tupla)
  def fn(**kwargs):            # argumentos nombrados variables (dict)
  lambda x: expresión          # función anónima de una línea

SCOPE
─────────────────────────────────────────────────────
  Local    → variable dentro de una función
  Global   → variable fuera de funciones
  global x → declara que x es la variable global
```

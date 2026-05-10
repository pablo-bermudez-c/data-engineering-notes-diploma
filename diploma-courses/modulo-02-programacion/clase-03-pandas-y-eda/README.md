# Clase 3 — Pandas y Análisis Exploratorio de Datos (EDA)

---

## Tabla de contenidos

1. [¿Qué es Pandas?](#bloque-1)
2. [Series y DataFrames](#bloque-2)
3. [Cargar y explorar datos](#bloque-3)
4. [Selección de columnas y filas](#bloque-4)
5. [Filtrado de datos](#bloque-5)
6. [Nuevas columnas y transformaciones](#bloque-6)
7. [Estadísticas y agregaciones](#bloque-7)
8. [Agrupaciones con `groupby`](#bloque-8)
9. [Valores faltantes y duplicados](#bloque-9)
10. [EDA — Análisis Exploratorio de Datos](#bloque-10)
11. [Visualización con Matplotlib](#bloque-11)

---

## Bloque 1 — ¿Qué es Pandas? {#bloque-1}

**Pandas** es la librería de Python más usada para el **análisis y manipulación de datos tabulares**. Su nombre viene de *Panel Data*, un término de econometría. Es el estándar de la industria en Data Engineering, Data Science y Analytics.

```
¿Por qué Pandas?
──────────────────────────────────────────────────────────────
  Antes de Pandas           Con Pandas
  ────────────────────      ──────────────────────────────────
  Listas y diccionarios     DataFrame (tabla con etiquetas)
  Bucles para filtrar       Filtros en una línea
  Cálculos manuales         Estadísticas integradas
  Archivos CSV a mano       read_csv() con una línea
──────────────────────────────────────────────────────────────
```

### Instalación e importación

```python
# Instalar (solo una vez, desde terminal)
pip install pandas

# Importar en tu script o notebook
import pandas as pd        # pd es el alias estándar
import numpy as np         # numpy es un complemento frecuente
```

> 💡 **Contexto DE:** Pandas es la herramienta central para transformar datos en pipelines ETL: leer archivos, limpiar columnas, aplicar reglas de negocio y exportar el resultado. Casi todo pipeline Python que manipula datos tabulares usa Pandas.

---

## Bloque 2 — Series y DataFrames {#bloque-2}

Pandas tiene dos estructuras de datos fundamentales:

```
Estructuras principales de Pandas
───────────────────────────────────────────────────────
  Series                        DataFrame
  ──────────────────────        ──────────────────────────────
  Una columna con índice        Tabla 2D (filas × columnas)
                                Cada columna es una Series

  0    Santiago                       Ciudad    Región    Temp
  1    Valparaíso         →       0   Santiago  Metro     22
  2    Concepción                 1   Valpo...  Valpo     18
  dtype: object                  2   Concep... Biobío    15
───────────────────────────────────────────────────────
```

### Series

Una **Series** es un array unidimensional con etiquetas (índices).

```python
# Crear una Serie manualmente
s = pd.Series([10, 20, 30, 40], index=["a", "b", "c", "d"])
print(s)
# a    10
# b    20
# c    30
# d    40
# dtype: int64

# Acceder a elementos
print(s["b"])     # 20
print(s.iloc[1])  # 20  (también por posición, usando .iloc)
```

### DataFrame

Un **DataFrame** es una tabla bidimensional: filas y columnas etiquetadas.

```python
# Crear un DataFrame desde un diccionario
data = {
    "nombre":  ["Ana", "Luis", "Pedro", "Sofía"],
    "ciudad":  ["Santiago", "Valparaíso", "Santiago", "Temuco"],
    "ventas":  [1500, 980, 2100, 760],
    "activo":  [True, True, False, True]
}

df = pd.DataFrame(data)
print(df)
#   nombre       ciudad  ventas  activo
# 0    Ana     Santiago    1500    True
# 1   Luis   Valparaíso     980    True
# 2  Pedro     Santiago    2100   False
# 3  Sofía       Temuco     760    True
```

> 💡 **Analogía:** un DataFrame es esencialmente una tabla SQL en memoria. Si vienes de SQL, casi todas las operaciones (`SELECT`, `WHERE`, `GROUP BY`, `JOIN`) tienen un equivalente directo en Pandas. Esta es la conexión que usaremos a lo largo de toda la clase.

---

## Bloque 3 — Cargar y explorar datos {#bloque-3}

En la práctica, los datos no se crean manualmente: se leen desde archivos.

### Leer un archivo CSV

```python
# Lectura básica
df = pd.read_csv("ventas.csv")

# Con separador personalizado (punto y coma es común en datasets latinoamericanos)
df = pd.read_csv("ventas.csv", sep=";")

# Con encoding específico (para tildes y ñ)
df = pd.read_csv("ventas.csv", sep=";", encoding="utf-8")

# Otros formatos
df = pd.read_excel("ventas.xlsx")
df = pd.read_json("datos.json")
```

> ⚠️ **Error común:** Si el CSV usa `;` como separador (formato europeo/latinoamericano) y no se especifica `sep=";"`, Pandas interpretará toda la fila como una sola columna.

---

### Funciones de exploración inicial

Una vez cargado el dataset, lo primero es entender su estructura. Estas funciones son el **checklist mínimo** de exploración:

| Función | Descripción | Retorna |
|---|---|---|
| `df.head(n)` | Primeras `n` filas (default: 5) | DataFrame |
| `df.tail(n)` | Últimas `n` filas (default: 5) | DataFrame |
| `df.shape` | Dimensiones: (filas, columnas) | Tupla |
| `df.info()` | Tipos de datos y valores no nulos | Imprime resumen |
| `df.describe()` | Estadísticas de columnas numéricas | DataFrame |
| `df.columns` | Nombres de las columnas | Index |
| `df.dtypes` | Tipo de dato de cada columna | Series |
| `df.nunique()` | Cantidad de valores únicos por columna | Series |
| `df['col'].unique()` | Valores únicos de una columna | Array |
| `df['col'].value_counts()` | Frecuencia de cada valor | Series |

```python
# Dataset de ejemplo: pasajeros del Titanic
df = pd.read_csv("titanic.csv")

# ¿Cuántas filas y columnas tiene?
print(df.shape)          # (891, 5)

# Ver las primeras filas
print(df.head())
#    Survived  Pclass     Sex   Age     Fare
# 0         0       3    male  22.0   7.2500
# 1         1       1  female  38.0  71.2833
# 2         1       3  female  26.0   7.9250

# Información general: tipos y valores no nulos
print(df.info())
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 891 entries, 0 to 890
# Data columns (total 5 columns):
#  #   Column    Non-Null Count  Dtype
# ---  ------    --------------  -----
#  0   Survived  891 non-null    int64
#  1   Pclass    891 non-null    int64
#  2   Sex       891 non-null    object
#  3   Age       714 non-null    float64   ← 177 valores faltantes!
#  4   Fare      891 non-null    float64

# Estadísticas descriptivas de columnas numéricas
print(df.describe())
#          Survived      Pclass         Age        Fare
# count  891.000000  891.000000  714.000000  891.000000
# mean     0.383838    2.308642   29.699118   32.204208
# std      0.486592    0.836071   14.526497   49.693429
# min      0.000000    1.000000    0.420000    0.000000
# 25%      0.000000    2.000000   20.125000    7.910400
# 50%      0.000000    3.000000   28.000000   14.454200
# 75%      1.000000    3.000000   38.000000   31.000000
# max      1.000000    3.000000   80.000000  512.329200

# Valores únicos en una columna categórica
print(df["Sex"].unique())          # ['male' 'female']
print(df["Sex"].value_counts())
# male      577
# female    314

# Cuántos valores únicos tiene cada columna
print(df.nunique())
# Survived      2
# Pclass        3
# Sex           2
# Age          88
# Fare        248
```

> 💡 **Contexto DE:** `df.info()` es la primera función que debes ejecutar al recibir un dataset nuevo. Te dice de inmediato si hay valores nulos, qué tipos tienen las columnas (un `object` donde esperabas `float64` indica un problema de calidad) y cuánta memoria usa el dataset.

---

## Bloque 4 — Selección de columnas y filas {#bloque-4}

### Seleccionar columnas

```python
# Una columna → retorna Series
serie = df["Survived"]

# Múltiples columnas → retorna DataFrame (lista dentro de lista)
df_sub = df[["Nombre", "Ciudad", "Ventas"]]
```

### Seleccionar filas con `loc` e `iloc`

| Método | Tipo de acceso | Sintaxis |
|---|---|---|
| `df.loc[etiqueta]` | Por **etiqueta** (nombre de índice) | `df.loc[0, "Age"]` |
| `df.iloc[posición]` | Por **posición** numérica | `df.iloc[0, 2]` |

```python
# loc: por etiqueta de índice y nombre de columna
df.loc[0]                    # fila con índice 0 (toda la fila)
df.loc[0, "Age"]             # fila 0, columna Age → 22.0
df.loc[0:4, ["Age","Fare"]]  # filas 0 a 4, columnas Age y Fare

# iloc: por posición (0-based)
df.iloc[0]                   # primera fila
df.iloc[0, 2]                # primera fila, tercera columna
df.iloc[0:5, 2:4]            # primeras 5 filas, columnas 2 y 3

# Seleccionar todas las filas de una columna por posición
df.iloc[:, 3]                # columna en posición 3 (Age)
```

> ⚠️ **Error común:** `loc` incluye el extremo final del rango (`loc[0:4]` incluye el 4), mientras que `iloc` no lo incluye (`iloc[0:4]` va del 0 al 3). `iloc` se comporta como el slicing estándar de listas de Python.

---

## Bloque 5 — Filtrado de datos {#bloque-5}

Filtrar en Pandas significa aplicar **condiciones booleanas** sobre las filas.

### Filtro simple

```python
# Pasajeros que sobrevivieron
sobrevivientes = df[df["Survived"] == 1]

# Pasajeros mayores de 40 años
mayores40 = df[df["Age"] > 40]

# Solo mujeres
mujeres = df[df["Sex"] == "female"]

# Una región específica
magallanes = df[df["Region"] == "Magallanes"]
```

### Filtros múltiples

```python
# Operador AND: ambas condiciones deben cumplirse (usar &)
mujeres_1clase = df[(df["Sex"] == "female") & (df["Pclass"] == 1)]

# Operador OR: al menos una condición debe cumplirse (usar |)
clases_altas = df[(df["Pclass"] == 1) | (df["Pclass"] == 2)]

# Negación: condición contraria (usar ~)
no_sobrevivieron = df[~(df["Survived"] == 1)]

# Equivalente a !=
no_sobrevivieron = df[df["Survived"] != 1]
```

> ⚠️ **Error común:** En Pandas se usan `&` y `|` en lugar de `and` y `or`. Además, cada condición **debe ir entre paréntesis** cuando se combinan: `df[(cond1) & (cond2)]`.

### Filtrar con `.isin()`

Útil para verificar pertenencia a una lista de valores:

```python
# Pasajeros en primera o segunda clase
df[df["Pclass"].isin([1, 2])]

# Registros de ciertas ciudades
ciudades = ["Santiago", "Valparaíso", "Concepción"]
df[df["Ciudad"].isin(ciudades)]
```

### Filtrar con `.str`

Para condiciones sobre texto:

```python
# Filas donde el nombre contiene "Juan" (sin importar mayúsculas)
df[df["Nombre"].str.contains("Juan", case=False)]

# Nombres que empiezan con "M"
df[df["Nombre"].str.startswith("M")]
```

### Alternativa más legible: `.query()`

Cuando una expresión combina varias condiciones, `df[df["..."] == ...]` se vuelve difícil de leer. El método `.query()` permite expresar el filtro como una cadena tipo SQL:

```python
# Equivalente a: df[(df["Sex"] == "female") & (df["Pclass"] == 1) & (df["Age"] >= 25)]
df.query("Sex == 'female' and Pclass == 1 and Age >= 25")

# Inyectar variables Python con @
edad_min = 25
df.query("Sex == 'female' and Age >= @edad_min")
```

> 💡 **Cuándo usar `query()`:** para filtros largos o cuando el código se está volviendo ilegible por la repetición de `df["..."]`. Para filtros simples, la sintaxis con `&`/`|` es suficiente y más rápida.

---

## Bloque 6 — Nuevas columnas y transformaciones {#bloque-6}

### Crear una nueva columna

```python
# A partir de operaciones entre columnas existentes
df["HorasTotales"] = df["HorasDia"] + df["HorasNoche"]

# A partir de una constante
df["Pais"] = "Chile"

# Aplicar una función a cada valor (con apply)
df["Edad_Categoria"] = df["Age"].apply(
    lambda edad: "Joven" if edad < 30 else "Adulto"
)
```

### Renombrar columnas

```python
# Renombrar columnas específicas
df = df.rename(columns={
    "HorasDia":    "Horas_Dia",
    "HorasNoche":  "Horas_Noche"
})

# Renombrar todas las columnas (debe coincidir en cantidad)
df.columns = ["Nombre", "Ciudad", "Region", "Horas_Dia", "Horas_Noche"]
```

### Eliminar columnas

```python
# Eliminar una columna
df = df.drop(columns=["columna_innecesaria"])

# Eliminar varias columnas
df = df.drop(columns=["col1", "col2"])
```

### Ordenar filas

```python
# Ordenar por una columna (ascendente por defecto)
df_ordenado = df.sort_values("HorasTotales")

# Descendente
df_ordenado = df.sort_values("HorasTotales", ascending=False)

# Por múltiples columnas
df_ordenado = df.sort_values(["Ciudad", "HorasTotales"], ascending=[True, False])

# Resetear el índice tras ordenar
df_ordenado = df.sort_values("HorasTotales").reset_index(drop=True)
```

> 💡 **Contexto DE:** `sort_values` + `head()` es la forma más rápida de obtener un ranking: los 10 registros con mayor valor de una métrica, que es una consulta habitual al entregar reportes a negocio.

---

## Bloque 7 — Estadísticas y agregaciones {#bloque-7}

### Funciones estadísticas sobre columnas

```python
# Funciones individuales
df["Age"].mean()     # promedio → 29.69
df["Age"].median()   # mediana  → 28.0
df["Age"].std()      # desviación estándar → 14.52
df["Age"].min()      # mínimo   → 0.42
df["Age"].max()      # máximo   → 80.0
df["Age"].sum()      # suma total
df["Age"].count()    # cantidad de valores no nulos

# Resumen completo
df.describe()

# Correlación entre columnas numéricas
df.corr(numeric_only=True)
#           Survived    Pclass       Age      Fare
# Survived  1.000000 -0.338481 -0.077221  0.257307
# Pclass   -0.338481  1.000000 -0.369226 -0.549500
# Age      -0.077221 -0.369226  1.000000  0.096067
# Fare      0.257307 -0.549500  0.096067  1.000000
```

### `value_counts()` para columnas categóricas

```python
# Frecuencia de cada categoría
df["Sex"].value_counts()
# male      577
# female    314

# En porcentajes
df["Sex"].value_counts(normalize=True)
# male      0.647587
# female    0.352413
```

---

## Bloque 8 — Agrupaciones con `groupby` {#bloque-8}

`groupby` es la operación más poderosa de Pandas: **divide el dataset en grupos** según una o más columnas, y aplica una función a cada grupo.

> 💡 **Conexión con SQL:** `groupby` es el equivalente Python al `GROUP BY` de SQL. Si ya viste la clase de SQL del Módulo 01, esta sección te resultará familiar — la lógica es idéntica, solo cambia la sintaxis.

```
Lógica de groupby (Split → Apply → Combine)
──────────────────────────────────────────────────────────────────
  Dataset original            Agrupado por Clase
  ─────────────────           ─────────────────────────────────
  Pclass  Survived            Pclass 1 → [1, 0, 1, 1, ...] → mean
  1       1            →      Pclass 2 → [1, 0, 1, ...] → mean
  3       0                   Pclass 3 → [0, 0, 1, ...] → mean
  2       1
  1       0                   Resultado:
  ...                         Pclass  Survived
                              1       0.630
                              2       0.473
                              3       0.242
──────────────────────────────────────────────────────────────────
```

### Sintaxis básica

```python
# Promedio de Age por clase
df.groupby("Pclass")["Age"].mean()
# Pclass
# 1    38.233
# 2    29.878
# 3    25.141

# Suma de horas por región
df.groupby("Region")["HorasTotales"].sum()

# Múltiples funciones a la vez con .agg()
df.groupby("Pclass")["Age"].agg(["mean", "min", "max", "count"])
#         mean   min   max  count
# Pclass
# 1      38.23  0.92  80.0    186
# 2      29.88  0.67  70.0    173
# 3      25.14  0.42  74.0    355

# Agrupar por múltiples columnas
df.groupby(["Sex", "Pclass"])["Survived"].mean()
# Sex     Pclass
# female  1         0.968
#         2         0.921
#         3         0.500
# male    1         0.369
#         2         0.157
#         3         0.135
```

### Obtener el grupo con mayor/menor valor

```python
# Ciudad con mayor promedio de horas nocturnas
promedio_por_ciudad = df.groupby("Ciudad")["HorasNoche"].mean()
ciudad_max = promedio_por_ciudad.idxmax()
print(f"Ciudad con más horas nocturnas: {ciudad_max}")

# Ciudad con menos horas totales
totales_por_ciudad = df.groupby("Ciudad")["HorasTotales"].sum()
ciudad_min = totales_por_ciudad.idxmin()
print(f"Ciudad con menos horas totales: {ciudad_min}")
```

> 💡 **Contexto DE:** Dominar `groupby` es fundamental para construir reportes de negocio: ventas por región, registros por categoría, promedios por período. Si te encuentras escribiendo un `for` para acumular suma por categoría, casi seguro que `groupby` lo resuelve en una línea.

---

## Bloque 9 — Valores faltantes y duplicados {#bloque-9}

Los datos del mundo real siempre tienen imperfecciones. Pandas tiene herramientas específicas para detectarlas y tratarlas.

### Detectar valores faltantes

```python
# Cantidad de valores nulos por columna
df.isna().sum()
# Survived      0
# Pclass        0
# Sex           0
# Age         177    ← 177 registros sin edad
# Fare          0

# Porcentaje de nulos
df.isna().mean() * 100
# Age    19.87%

# Ver las filas con nulos en Age
df[df["Age"].isna()]
```

### Tratar valores faltantes

```python
# Opción 1: Eliminar filas con nulos (puede perder información)
df_sin_nulos = df.dropna()

# Eliminar solo si hay nulos en columnas específicas
df_sin_nulos = df.dropna(subset=["Age"])

# Opción 2: Rellenar con un valor fijo
df["Age"] = df["Age"].fillna(0)

# Opción 3: Rellenar con la media (más habitual)
media_edad = df["Age"].mean()
df["Age"] = df["Age"].fillna(media_edad)

# Opción 4: Rellenar con el valor anterior (útil en series temporales)
df["valor"] = df["valor"].ffill()
```

### Detectar y eliminar duplicados

```python
# ¿Hay filas duplicadas?
print(df.duplicated().sum())    # número de filas duplicadas

# Ver las filas duplicadas
df[df.duplicated()]

# Eliminar duplicados (conserva la primera ocurrencia)
df = df.drop_duplicates()

# Eliminar duplicados basándose en columnas específicas
df = df.drop_duplicates(subset=["Nombre", "Ciudad"])
```

> ⚠️ **Buena práctica DE:** Nunca elimines datos sin documentar por qué. En un pipeline de producción, es mejor registrar los registros eliminados en un log o tabla de errores antes de descartarlos. La trazabilidad es fundamental.

---

## Bloque 10 — EDA: Análisis Exploratorio de Datos {#bloque-10}

**EDA** (*Exploratory Data Analysis*) es el proceso de investigar, resumir y visualizar un dataset antes de tomar decisiones o construir modelos. No es solo ejecutar funciones: es **hacerse preguntas** sobre los datos y buscar respuestas.

```
Flujo de un EDA típico
──────────────────────────────────────────────────────────────────
  1. Cargar datos          → read_csv(), shape, columns
  2. Exploración inicial   → head(), info(), describe()
  3. Calidad de datos      → isna(), duplicated(), dtypes
  4. Limpieza              → fillna(), dropna(), drop_duplicates()
  5. Análisis univariado   → value_counts(), describe() por columna
  6. Análisis bivariado    → groupby(), corr(), crosstab()
  7. Visualización         → barras, histogramas, scatter, heatmap
  8. Conclusiones          → ¿Qué aprendiste? ¿Qué anomalías hay?
──────────────────────────────────────────────────────────────────
```

### Ejemplo integrado: Dataset Titanic

```python
import pandas as pd
import numpy as np

# 1. Cargar y seleccionar columnas relevantes
df = pd.read_csv("titanic.csv")
df = df[["Survived", "Pclass", "Sex", "Age", "Fare"]]

# 2. Exploración inicial
print(df.shape)        # (891, 5)
print(df.dtypes)
print(df.describe())

# 3. Calidad de datos
print("\nValores nulos:")
print(df.isna().sum())
# Age: 177 nulos

# 4. Limpieza: rellenar Age con la media
df["Age"] = df["Age"].fillna(df["Age"].mean())
print(df.isna().sum())    # ahora 0 nulos

# 5. Análisis: tasa de supervivencia por clase
print("\nSupervivencia por clase:")
print(df.groupby("Pclass")["Survived"].mean().round(2))
# Pclass
# 1    0.63  → 63% sobrevivió
# 2    0.47
# 3    0.24  → solo 24% sobrevivió

# 6. Análisis: supervivencia por sexo
print("\nSupervivencia por sexo:")
print(df.groupby("Sex")["Survived"].mean().round(2))
# female    0.74   → 74% de las mujeres sobrevivió
# male      0.19   → solo 19% de los hombres

# 7. Correlación entre variables numéricas
print("\nCorrelación con Survived:")
print(df.corr(numeric_only=True)["Survived"].sort_values())
# Pclass     -0.338  → clase más baja, menos supervivencia
# Age        -0.077
# Survived    1.000
# Fare        0.257  → tarifa más alta, más supervivencia
```

> 💡 **El paso 8 es el más importante.** Las funciones de Pandas no hacen EDA por sí solas — solo entregan números. El EDA real consiste en **interpretar** esos números: ¿qué historia cuentan? ¿qué decisión informan? Si tu notebook termina en un `print(df.describe())` sin conclusión escrita, no terminaste el EDA.

---

## Bloque 11 — Visualización con Matplotlib {#bloque-11}

Los datos son más fáciles de interpretar cuando se visualizan. **Matplotlib** es la librería base de visualización en Python.

### Configuración básica

```python
import matplotlib.pyplot as plt

# Modo inline en Jupyter (las gráficas aparecen en el notebook)
%matplotlib inline
```

### Gráfico de barras

Ideal para **variables categóricas**: frecuencias, conteos, comparaciones.

```python
# Supervivencia (0 vs 1)
plt.figure(figsize=(4, 3))
df["Survived"].value_counts().plot(kind="bar", color=["red", "green"])
plt.title("Supervivencia en el Titanic")
plt.xlabel("0 = No, 1 = Sí")
plt.ylabel("Cantidad de pasajeros")
plt.xticks(rotation=0)       # etiquetas horizontales
plt.tight_layout()
plt.show()

# Distribución por clase de pasajero
plt.figure(figsize=(4, 3))
df["Pclass"].value_counts().sort_index().plot(kind="bar", color="steelblue")
plt.title("Pasajeros por clase")
plt.xlabel("Clase")
plt.ylabel("Cantidad")
plt.tight_layout()
plt.show()
```

### Histograma

Ideal para **variables numéricas**: ver la distribución de los valores.

```python
# Distribución de edades
plt.figure(figsize=(5, 3))
df["Age"].plot(kind="hist", bins=20, edgecolor="white", color="steelblue")
plt.title("Distribución de edades")
plt.xlabel("Edad")
plt.ylabel("Frecuencia")
plt.tight_layout()
plt.show()
```

### Gráfico de dispersión (Scatter)

Ideal para **relaciones entre dos variables numéricas**.

```python
# Relación entre Edad y Tarifa
plt.figure(figsize=(5, 4))
plt.scatter(df["Age"], df["Fare"], alpha=0.4, color="steelblue")
plt.title("Edad vs Tarifa")
plt.xlabel("Edad")
plt.ylabel("Tarifa")
plt.tight_layout()
plt.show()

# Scatter coloreado por una tercera variable
colors = df["Survived"].map({0: "red", 1: "green"})
plt.figure(figsize=(5, 4))
plt.scatter(df["Age"], df["Fare"], c=colors, alpha=0.5)
plt.title("Edad vs Tarifa (color = Survived)")
plt.xlabel("Edad")
plt.ylabel("Tarifa")
plt.tight_layout()
plt.show()
```

### Estructura general de una figura Matplotlib

```python
# Anatomía de una gráfica bien construida
plt.figure(figsize=(ancho, alto))    # tamaño en pulgadas
# ... código del gráfico ...
plt.title("Título descriptivo")
plt.xlabel("Etiqueta eje X")
plt.ylabel("Etiqueta eje Y")
plt.legend()                         # leyenda (si aplica)
plt.tight_layout()                   # evitar que se corten etiquetas
plt.show()                           # mostrar la figura
```

> 💡 **Consejo:** Siempre incluye `plt.tight_layout()` antes de `plt.show()`. Evita que los títulos y etiquetas se superpongan o queden cortados.

---

<details>
<summary><strong>🟢 Ejercicio 1 — Exploración básica (click para ver)</strong></summary>

Dado el siguiente dataset ficticio de ventas:

```python
import pandas as pd

data = {
    "vendedor":  ["Ana", "Luis", "Pedro", "Ana", "Luis", "Pedro", "Sofía"],
    "region":    ["Norte", "Sur", "Norte", "Sur", "Norte", "Sur", "Norte"],
    "mes":       ["Ene", "Ene", "Ene", "Feb", "Feb", "Feb", "Ene"],
    "ventas":    [1500, 980, 2100, 760, 1300, 1750, None],
    "clientes":  [10, 7, 14, 5, 9, 12, 6]
}

df = pd.DataFrame(data)
```

1. ¿Cuántas filas y columnas tiene?
2. ¿Qué columnas tienen valores nulos?
3. Rellena el nulo con el promedio de ventas.
4. Filtra los registros donde ventas > 1200 y región == "Norte".

**Solución:**

```python
# 1. Dimensiones
print(df.shape)       # (7, 5)

# 2. Nulos por columna
print(df.isna().sum())
# ventas    1  ← único nulo

# 3. Rellenar con el promedio
promedio = df["ventas"].mean()
df["ventas"] = df["ventas"].fillna(promedio)

# 4. Filtrar
filtrado = df[(df["ventas"] > 1200) & (df["region"] == "Norte")]
print(filtrado)
```

</details>

<details>
<summary><strong>🟢 Ejercicio 2 — Agrupaciones y rankings (click para ver)</strong></summary>

Con el mismo DataFrame del ejercicio anterior:

1. Calcula el total de ventas por vendedor.
2. Calcula el promedio de ventas por región y mes.
3. Crea una columna `ventas_por_cliente = ventas / clientes`.
4. Muestra los 3 registros con mayor `ventas_por_cliente`.

**Solución:**

```python
# 1. Total por vendedor
print(df.groupby("vendedor")["ventas"].sum())

# 2. Promedio por región y mes
print(df.groupby(["region", "mes"])["ventas"].mean())

# 3. Nueva columna
df["ventas_por_cliente"] = df["ventas"] / df["clientes"]

# 4. Top 3
top3 = df.sort_values("ventas_por_cliente", ascending=False).head(3)
print(top3[["vendedor", "ventas", "clientes", "ventas_por_cliente"]])
```

</details>

<details>
<summary><strong>🟢 Ejercicio 3 — EDA completo (click para ver)</strong></summary>

Descarga el dataset `titanic.csv` y realiza un EDA básico respondiendo estas preguntas:

1. ¿Cuántos pasajeros hay en cada clase?
2. ¿Cuál es la edad promedio de los sobrevivientes vs los que no sobrevivieron?
3. ¿Qué porcentaje de mujeres sobrevivió? ¿Y de hombres?
4. Genera un gráfico de barras de supervivencia por sexo.

**Solución:**

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("titanic.csv")
df = df[["Survived", "Pclass", "Sex", "Age", "Fare"]]
df["Age"] = df["Age"].fillna(df["Age"].mean())

# 1. Pasajeros por clase
print(df["Pclass"].value_counts().sort_index())

# 2. Edad promedio por supervivencia
print(df.groupby("Survived")["Age"].mean())
# 0    30.63  (no sobrevivieron, más edad promedio)
# 1    28.41  (sobrevivieron)

# 3. Tasa de supervivencia por sexo
print(df.groupby("Sex")["Survived"].mean().round(2))
# female    0.74  → 74%
# male      0.19  → 19%

# 4. Gráfico de barras
tasas = df.groupby("Sex")["Survived"].mean()
plt.figure(figsize=(5, 4))
tasas.plot(kind="bar", color=["pink", "steelblue"])
plt.title("Tasa de supervivencia por sexo")
plt.ylabel("Proporción de sobrevivientes")
plt.xlabel("Sexo")
plt.xticks(rotation=0)
plt.tight_layout()
plt.show()
```

</details>

---

## Referencia rápida — Pandas

```
CARGA DE DATOS
─────────────────────────────────────────────────────────────────
  pd.read_csv("archivo.csv", sep=";")
  pd.read_excel("archivo.xlsx")

EXPLORACIÓN
─────────────────────────────────────────────────────────────────
  df.head(n)     df.tail(n)     df.shape       df.info()
  df.describe()  df.columns     df.dtypes      df.nunique()
  df["col"].value_counts()      df["col"].unique()

SELECCIÓN
─────────────────────────────────────────────────────────────────
  df["col"]                     → Serie
  df[["col1", "col2"]]          → DataFrame
  df.loc[fila, col]             → por etiqueta
  df.iloc[i, j]                 → por posición

FILTRADO
─────────────────────────────────────────────────────────────────
  df[df["col"] > valor]
  df[(cond1) & (cond2)]         → AND
  df[(cond1) | (cond2)]         → OR
  df[df["col"].isin([a, b])]
  df.query("col > 5 and otra == 'x'")

TRANSFORMACIONES
─────────────────────────────────────────────────────────────────
  df["nueva"] = df["a"] + df["b"]
  df.sort_values("col", ascending=False)
  df.rename(columns={"old": "new"})
  df.drop(columns=["col"])

ESTADÍSTICAS
─────────────────────────────────────────────────────────────────
  .mean()  .median()  .std()  .min()  .max()  .sum()  .count()
  df.corr(numeric_only=True)

AGRUPACIONES
─────────────────────────────────────────────────────────────────
  df.groupby("col")["otra"].mean()
  df.groupby("col")["otra"].agg(["mean", "sum", "count"])
  .idxmax()  .idxmin()   → índice del mayor/menor valor

VALORES FALTANTES
─────────────────────────────────────────────────────────────────
  df.isna().sum()
  df.fillna(valor)
  df.dropna(subset=["col"])
  df.drop_duplicates()
```

---

*→ Próxima clase: [Archivos, NumPy y APIs](../clase-04-archivos-numpy-y-apis/README.md)*

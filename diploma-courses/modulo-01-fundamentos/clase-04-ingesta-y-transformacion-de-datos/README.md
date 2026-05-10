# Clase 4: Ingesta y Transformación de Datos

**Manejo práctico de CSV, JSON y XML con Pandas**

---

# Hoja de Ruta

| Sección | Formato | Herramienta principal | Desafío típico |
|---|---|---|---|
| **01** | CSV | `pandas.read_csv()` | Encoding, tipos incorrectos, nulos |
| **02** | JSON | `requests` + `pd.json_normalize()` | Aplanamiento de estructuras anidadas |
| **03** | XML | `xml.etree.ElementTree` | Parseo de DOM jerárquico |
| **04** | Arquitectura | Pipeline completo | Del dato crudo al valor |

> **Entorno de trabajo:** Usaremos **Google Colab** con Python y Pandas. Los datos se descargan directamente a la nube usando la **API de Kaggle**, replicando el flujo real de un Data Engineer: los datos nunca pasan por tu disco local.

---

# Configurando el Entorno

## Google Colab + Kaggle API

En el mundo real, los datos viven en servidores remotos, buckets de S3, APIs externas o plataformas de datos como Kaggle. Un Data Engineer raramente trabaja con archivos que ya están en su computador.

Google Colab ofrece un entorno de ejecución en la nube con Python preinstalado, acceso a GPU/TPU gratuito y, lo más importante para esta clase, conectividad directa con fuentes de datos externas.

### Configuración de la API de Kaggle

```python
# Paso 1: Subir el archivo kaggle.json desde tu cuenta Kaggle
# (Settings → Account → API → Create New Token)
from google.colab import files
files.upload()   # Selecciona tu kaggle.json

# Paso 2: Crear el directorio oculto de configuración
!mkdir -p ~/.kaggle

# Paso 3: Copiar las credenciales al directorio correcto
!cp kaggle.json ~/.kaggle/

# Paso 4: Restringir permisos (¡recuerda lo aprendido en la Clase 2!)
!chmod 600 ~/.kaggle/kaggle.json

# Paso 5: Verificar que la configuración funciona
!kaggle datasets list --search "netflix" | head -5
```

> **Conexión con Clase 2:** `chmod 600` es exactamente lo que aprendimos para proteger credenciales. El archivo `kaggle.json` contiene tu API key y no debe ser legible por otros usuarios del sistema.

---

# Sección 1: CSV — Archivos Planos Universales

## ¿Qué es un CSV y por qué es el formato más común?

El formato CSV (*Comma-Separated Values*) es el denominador común del intercambio de datos. Cualquier sistema, de cualquier época, puede producir o consumir un CSV: desde SAP hasta una planilla de Excel de 1995.

```
# Un CSV bien formado
show_id,type,title,director,country,date_added,release_year,rating,duration
s1,Movie,Dick Johnson Is Dead,Kirsten Johnson,United States,"September 25, 2021",2020,PG-13,90 min
s2,TV Show,Blood & Water,,South Africa,"September 24, 2021",2021,TV-MA,2 Seasons
s3,TV Show,Gilmore Girls,,United States,"January 1, 2016",2000,TV-G,7 Seasons
```

### Características y problemas frecuentes

| Característica | Detalle | Problema para el Data Engineer |
|---|---|---|
| **Delimitadores** | `,` `;` `\|` `\t` | El campo puede contener el mismo carácter del delimitador |
| **Text qualifiers** | Comillas `"` envuelven campos con comas internas | Comillas mal cerradas rompen el parseo completo |
| **Sin esquema** | No hay tipos definidos, todo es texto | Pandas infiere los tipos: puede equivocarse |
| **Encoding** | `utf-8`, `latin-1`, `cp1252` | Un carácter inválido puede interrumpir la lectura |
| **Nulos implícitos** | Celda vacía, `"N/A"`, `"Not Given"`, `"NULL"` | Cada sistema los representa diferente |

---

## Ingesta de CSV con Pandas

### `pd.read_csv()` — Parámetros esenciales

```python
import pandas as pd

# Lectura básica
df = pd.read_csv('datos.csv')

# Lectura controlada (producción)
df = pd.read_csv(
    'datos.csv',
    sep=',',                          # Delimitador (default: coma)
    encoding='utf-8',                 # Encoding explícito
    na_values=['', 'NaN', 'Not Given', 'N/A', 'NULL'],  # Qué considerar nulo
    dtype={'show_id': str},           # Forzar tipos en columnas críticas
    parse_dates=['date_added'],       # Parsear fechas automáticamente
    nrows=1000,                       # Leer solo las primeras N filas (exploración)
    usecols=['title', 'type', 'country', 'release_year']  # Solo columnas necesarias
)
```

### Descarga directa con Kaggle API

```python
# Descargar dataset de Netflix desde Kaggle
!kaggle datasets download -d shivamb/netflix-shows

# Descomprimir
!unzip -o netflix-shows.zip

# Leer con parámetros de producción
df_netflix = pd.read_csv(
    'netflix_titles.csv',
    na_values=['', 'NaN', 'Not Given']
)

# Verificar que la carga fue exitosa
print(f"Filas: {df_netflix.shape[0]:,}")
print(f"Columnas: {df_netflix.shape[1]}")
df_netflix.head(3)
```

---

## Diagnóstico inicial: las tres preguntas del Data Engineer

Ante cualquier CSV nuevo, siempre ejecuta estas tres líneas antes de tocar los datos:

```python
# 1. ¿Qué estructura tiene? (nombres de columnas y tipos inferidos)
df_netflix.info()
```

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 8807 entries, 0 to 8806
Data columns (total 12 columns):
 #   Column        Non-Null Count  Dtype
---  ------        --------------  -----
 0   show_id       8807 non-null   object
 1   type          8807 non-null   object
 2   title         8806 non-null   object
 3   director      6173 non-null   object    ← 2634 nulos
 4   cast          7982 non-null   object
 5   country       7976 non-null   object
 6   date_added    8797 non-null   object    ← debería ser datetime, no object
 7   release_year  8807 non-null   int64
 8   rating        8803 non-null   object
 9   duration      8804 non-null   object    ← "90 min" o "2 Seasons" mezclados
 10  listed_in     8807 non-null   object
 11  description   8807 non-null   object
dtypes: int64(1), object(11)
```

```python
# 2. ¿Cuántos nulos hay por columna?
print(df_netflix.isnull().sum().sort_values(ascending=False))
```

```
director        2634   ← 30% de los registros sin director
cast             825
country          831
date_added        10
rating             4
duration           3
title              1
```

```python
# 3. ¿Cómo lucen las estadísticas básicas?
df_netflix.describe(include='all')
```

---

## Desafío Práctico 1 — CSV: Identificar y corregir tipos de datos

### Problema 1: `date_added` es `object` en lugar de `datetime`

```python
# ❌ Estado actual: fecha como texto
print(df_netflix['date_added'].dtype)     # object
print(df_netflix['date_added'].head(3))
# September 25, 2021
# September 24, 2021
# January 1, 2016

# ✅ Corrección: convertir a datetime
df_netflix['date_added'] = pd.to_datetime(
    df_netflix['date_added'],
    format='mixed',      # Maneja múltiples formatos de fecha
    errors='coerce'      # Los que no parseen se convierten en NaT (no en error)
)

print(df_netflix['date_added'].dtype)     # datetime64[ns]
print(df_netflix['date_added'].head(3))
# 2021-09-25
# 2021-09-24
# 2016-01-01
```

### Problema 2: Nulos críticos en la columna `director`

```python
# Diagnóstico: ¿cuántos registros no tienen director?
sin_director = df_netflix['director'].isnull().sum()
total = len(df_netflix)
print(f"Sin director: {sin_director} ({sin_director/total:.1%})")
# Sin director: 2634 (29.9%)

# Estrategia 1: Rellenar con un valor centinela para no perder filas
df_netflix['director'] = df_netflix['director'].fillna('Desconocido')

# Estrategia 2: Eliminar filas donde el director es crítico para el análisis
df_con_director = df_netflix.dropna(subset=['director'])

# Estrategia 3: Para columnas numéricas, rellenar con la mediana
# df['columna_numerica'] = df['columna_numerica'].fillna(df['columna_numerica'].median())

# Verificar resultado
print(df_netflix['director'].isnull().sum())   # 0
```

> **💡 Decisión de negocio:** ¿Rellenar, eliminar o ignorar nulos? La respuesta depende del uso final. Si el análisis es sobre directores, eliminar las filas sin director tiene sentido. Si el análisis es sobre catálogo total, conviene mantenerlas con un valor centinela. **Siempre documenta la decisión** en el código o en un notebook adjunto.

### Problema 3: `duration` mezcla dos unidades en la misma columna

```python
# El problema: "90 min" para películas y "2 Seasons" para series
print(df_netflix[['type', 'duration']].head(8))
#    type     duration
# 0  Movie    90 min
# 1  TV Show  2 Seasons
# 2  TV Show  1 Season
# 3  Movie    66 min

# Solución: separar en dos columnas
df_netflix['duracion_numero'] = df_netflix['duration'].str.extract(r'(\d+)').astype(float)
df_netflix['duracion_unidad'] = df_netflix['duration'].str.extract(r'([a-zA-Z]+)')

# Resultado
print(df_netflix[['type', 'duration', 'duracion_numero', 'duracion_unidad']].head(4))
#    type     duration   duracion_numero  duracion_unidad
# 0  Movie    90 min     90.0             min
# 1  TV Show  2 Seasons  2.0              Seasons
```

> **⚠️ Advertencia:** Mezclar unidades en una columna es un *anti-pattern* clásico de bases de datos legacy. La regla de oro es: **una columna, una unidad, un tipo**. Si te encuentras con esto, separa siempre en columnas distintas antes de cualquier análisis cuantitativo.

### Guardar el DataFrame limpio

```python
# Guardar como CSV procesado
df_netflix.to_csv('netflix_limpio.csv', index=False, encoding='utf-8')

# Verificar que se guardó correctamente
df_verificacion = pd.read_csv('netflix_limpio.csv')
print(f"Filas guardadas: {len(df_verificacion):,}")
```

---

# Sección 2: JSON — El Estándar de APIs Modernas

## ¿Por qué JSON domina las APIs?

JSON (*JavaScript Object Notation*) es el formato nativo de las APIs REST modernas. Cuando tu pipeline consume datos de Spotify, un sistema de e-commerce o cualquier microservicio interno, el 90% de las veces la respuesta llega en JSON.

```json
{
  "results": [
    {
      "gender": "female",
      "name": { "title": "Ms", "first": "Ana", "last": "González" },
      "location": {
        "street": { "number": 1234, "name": "Av. Providencia" },
        "city": "Santiago",
        "country": "Chile",
        "postcode": "7500000"
      },
      "email": "ana.gonzalez@example.com",
      "dob": { "date": "1990-04-15T08:30:00.000Z", "age": 34 }
    }
  ]
}
```

### El reto para Data Engineering

```
JSON (profundidad anidada)              Tabla relacional (plana)
┌──────────────────────────┐            ┌────────────┬───────────┬──────────────────┐
│ name                     │            │ name.first │ name.last │ location.country │
│   ├── first: "Ana"       │    →→→     ├────────────┼───────────┼──────────────────┤
│   └── last: "González"   │            │ Ana        │ González  │ Chile            │
│ location                 │            └────────────┴───────────┴──────────────────┘
│   ├── city: "Santiago"   │
│   └── country: "Chile"   │   Los Data Warehouses necesitan tablas planas.
└──────────────────────────┘   JSON tiene profundidad variable: hay que aplanarlo.
```

---

## Consumiendo una API REST con Python

```python
import requests
import pandas as pd

# Hacer una petición GET a la API
url = "https://randomuser.me/api/?results=100&nat=cl,ar,mx"
respuesta = requests.get(url)

# Verificar que la petición fue exitosa
print(f"Status HTTP: {respuesta.status_code}")   # 200 = OK
if respuesta.status_code != 200:
    raise Exception(f"Error en la API: {respuesta.status_code}")

# Convertir la respuesta a diccionario Python
datos_json = respuesta.json()

# Explorar la estructura antes de procesar
print(type(datos_json))               # <class 'dict'>
print(datos_json.keys())              # dict_keys(['results', 'info'])
print(f"Usuarios recibidos: {len(datos_json['results'])}")

# Explorar un registro para entender la estructura
import json
print(json.dumps(datos_json['results'][0], indent=2))
```

### Navegación manual del JSON (para entender la estructura)

```python
primer_usuario = datos_json['results'][0]

# Acceder a campos de primer nivel
print(primer_usuario['gender'])               # female
print(primer_usuario['email'])                # ana.gonzalez@example.com

# Acceder a campos anidados
print(primer_usuario['name']['first'])        # Ana
print(primer_usuario['name']['last'])         # González
print(primer_usuario['location']['city'])     # Santiago
print(primer_usuario['location']['country'])  # Chile

# Acceder a arrays dentro del JSON
print(primer_usuario['dob']['age'])           # 34
```

---

## Flattening: Aplanando la estructura anidada

`pd.json_normalize()` es la herramienta que convierte la jerarquía del JSON en columnas planas, usando el punto `.` como separador para los campos anidados.

```python
# Aplanar automáticamente todos los niveles
df_usuarios = pd.json_normalize(datos_json['results'])

# Ver todas las columnas generadas
print(df_usuarios.columns.tolist())
# ['gender', 'email', 'phone', 'cell',
#  'name.title', 'name.first', 'name.last',
#  'location.street.number', 'location.street.name',
#  'location.city', 'location.state', 'location.country',
#  'location.postcode', 'location.coordinates.latitude',
#  'dob.date', 'dob.age',
#  'registered.date', 'registered.age',
#  'id.name', 'id.value',
#  'picture.large', 'picture.medium', 'picture.thumbnail']

print(f"Shape: {df_usuarios.shape}")   # (100, 22)
df_usuarios.head(3)
```

### Seleccionar y renombrar columnas relevantes

```python
# Seleccionar solo las columnas útiles
columnas_utiles = [
    'gender', 'email', 'phone',
    'name.first', 'name.last',
    'location.city', 'location.state', 'location.country',
    'dob.date', 'dob.age'
]

df_limpio = df_usuarios[columnas_utiles].copy()

# Renombrar: reemplazar puntos por guiones bajos (estándar snake_case)
df_limpio.columns = df_limpio.columns.str.replace('.', '_', regex=False)

print(df_limpio.columns.tolist())
# ['gender', 'email', 'phone', 'name_first', 'name_last',
#  'location_city', 'location_state', 'location_country',
#  'dob_date', 'dob_age']

df_limpio.head(3)
```

### Transformaciones adicionales sobre columnas JSON

```python
# Convertir fecha de nacimiento a datetime
df_limpio['dob_date'] = pd.to_datetime(df_limpio['dob_date'])

# Crear columna nombre completo
df_limpio['nombre_completo'] = df_limpio['name_first'] + ' ' + df_limpio['name_last']

# Crear columna de segmento etario
def segmento_etario(edad):
    if edad < 30:
        return 'Joven'
    elif edad < 50:
        return 'Adulto'
    else:
        return 'Senior'

df_limpio['segmento'] = df_limpio['dob_age'].apply(segmento_etario)

# Resumen del resultado
print(df_limpio[['nombre_completo', 'location_country', 'dob_age', 'segmento']].head(5))
```

---

## Desafío Práctico 2 — JSON: Aplanamiento y normalización completa

```python
import requests
import pandas as pd

# 1. Obtener datos de la API
url = "https://randomuser.me/api/?results=200&nat=cl,ar,mx,co,pe"
respuesta = requests.get(url)
datos = respuesta.json()

# 2. Aplanar la estructura JSON completa
df = pd.json_normalize(datos['results'])

# 3. Renombrar todas las columnas: puntos → guiones bajos
df.columns = df.columns.str.replace('.', '_', regex=False)

# 4. Seleccionar columnas relevantes para un pipeline de marketing
df_final = df[[
    'gender', 'email', 'cell',
    'name_first', 'name_last',
    'location_city', 'location_country',
    'dob_date', 'dob_age',
    'registered_date'
]].copy()

# 5. Convertir fechas
df_final['dob_date']        = pd.to_datetime(df_final['dob_date'])
df_final['registered_date'] = pd.to_datetime(df_final['registered_date'])

# 6. Columnas derivadas
df_final['nombre_completo']   = df_final['name_first'] + ' ' + df_final['name_last']
df_final['anios_registrado']  = (pd.Timestamp.now(tz='UTC') - df_final['registered_date']).dt.days // 365

# 7. Verificar resultado
print(df_final.info())
print(df_final.head(3))

# 8. Guardar DataFrame resultante
df_final.to_csv('usuarios_normalizados.csv', index=False)
print("Archivo guardado: usuarios_normalizados.csv")
```

### `pd.json_normalize()` con parámetros avanzados

Cuando el JSON tiene arrays de objetos anidados (por ejemplo, un pedido con múltiples líneas de detalle), se necesita el parámetro `record_path`:

```python
# Ejemplo: JSON con estructura de pedidos que contienen múltiples productos
pedidos_json = {
    "pedidos": [
        {
            "id": "P001",
            "cliente": "María González",
            "productos": [
                {"nombre": "Laptop", "precio": 1200000},
                {"nombre": "Mouse",  "precio": 15000}
            ]
        },
        {
            "id": "P002",
            "cliente": "Carlos Pérez",
            "productos": [
                {"nombre": "Teclado", "precio": 35000}
            ]
        }
    ]
}

# Aplanar expandiendo el array de productos (una fila por producto)
df_detalle = pd.json_normalize(
    data        = pedidos_json['pedidos'],
    record_path = ['productos'],           # Expandir este array
    meta        = ['id', 'cliente'],       # Mantener estos campos del nivel superior
    meta_prefix = 'pedido_'                # Prefijo para los campos meta
)

print(df_detalle)
#    nombre    precio  pedido_id  pedido_cliente
# 0  Laptop   1200000  P001       María González
# 1  Mouse      15000  P001       María González
# 2  Teclado    35000  P002       Carlos Pérez
```

> **💡 Caso real:** Las APIs de e-commerce (Shopify, MercadoLibre, WooCommerce) devuelven exactamente esta estructura: pedidos con un array anidado de líneas de producto. Saber aplanar este patrón es uno de los ejercicios más frecuentes en pipelines de datos transaccionales.

---

# Sección 3: XML — Sistemas Legacy en la Era Moderna

## ¿Por qué XML sigue siendo relevante?

A pesar de que JSON domina las APIs modernas, XML sigue siendo el estándar obligatorio en sectores críticos:

| Sector | Uso de XML | Ejemplo concreto |
|---|---|---|
| **Facturación electrónica** | Documentos tributarios B2B | DTE en Chile (SII), CFDI en México |
| **Sistemas bancarios** | Protocolo SOAP para transferencias | Mensajería SWIFT, ISO 20022 |
| **ERP empresarial** | SAP, Oracle, Microsoft Dynamics | Exportación de configuraciones y maestros |
| **Salud** | Registros clínicos electrónicos | HL7, FHIR |
| **Logística** | Tracking de envíos, manifiestos | EDI (*Electronic Data Interchange*) |

> **Realidad del Data Engineer:** si trabajas en banca, retail, manufactura o gobierno, vas a recibir XML. No es opcional aprenderlo.

---

## Estructura de un XML: los conceptos clave

```xml
<?xml version="1.0" encoding="UTF-8"?>   <!-- Declaración XML -->

<facturacion>                             <!-- Elemento raíz (root) -->

    <factura id="1001">                   <!-- Elemento con ATRIBUTO (id="1001") -->
        <cliente>Empresa Alpha</cliente>  <!-- Elemento con TEXTO -->
        <monto moneda="CLP">500000</monto><!-- Elemento con atributo Y texto -->
        <fecha>2026-03-21</fecha>
        <items>                           <!-- Elemento contenedor -->
            <item>
                <descripcion>Servicio de consultoría</descripcion>
                <cantidad>10</cantidad>
                <precio_unitario>50000</precio_unitario>
            </item>
        </items>
    </factura>

</facturacion>
```

**Conceptos fundamentales:**

| Concepto | Definición | Ejemplo |
|---|---|---|
| **Elemento** | Unidad básica delimitada por tags | `<cliente>Empresa Alpha</cliente>` |
| **Atributo** | Metadato dentro del tag de apertura | `id="1001"` en `<factura id="1001">` |
| **Texto** | Contenido del elemento | `Empresa Alpha` |
| **Nodo raíz** | El elemento que contiene a todos | `<facturacion>` |
| **Nodo hijo** | Elemento contenido dentro de otro | `<factura>` dentro de `<facturacion>` |
| **DOM** | Árbol completo en memoria | La estructura parseada completa |

---

## Creando el archivo XML en Colab

En Google Colab, el magic command `%%writefile` escribe el contenido de la celda directamente a un archivo, sin necesidad de código Python:

```python
%%writefile facturas.xml
<?xml version="1.0" encoding="UTF-8"?>
<facturacion>
    <factura id="1001">
        <cliente>Empresa Alpha</cliente>
        <monto moneda="CLP">500000</monto>
        <fecha>2026-01-15</fecha>
        <items>
            <item>
                <descripcion>Servicio de consultoría</descripcion>
                <cantidad>10</cantidad>
                <precio_unitario>50000</precio_unitario>
            </item>
        </items>
    </factura>
    <factura id="1002">
        <cliente>Tech Beta</cliente>
        <monto moneda="CLP">750000</monto>
        <fecha>2026-01-20</fecha>
        <items>
            <item>
                <descripcion>Licencia de software</descripcion>
                <cantidad>3</cantidad>
                <precio_unitario>250000</precio_unitario>
            </item>
        </items>
    </factura>
    <factura id="1003">
        <cliente>Industrias Gamma</cliente>
        <monto moneda="USD">2500</monto>
        <fecha>2026-02-01</fecha>
        <items>
            <item>
                <descripcion>Hardware servidor</descripcion>
                <cantidad>1</cantidad>
                <precio_unitario>2500</precio_unitario>
            </item>
        </items>
    </factura>
</facturacion>
```

---

## Parseando el DOM con `xml.etree.ElementTree`

Pandas tiene soporte básico para XML (`pd.read_xml()`), pero falla con estructuras profundas o cuando los datos están en atributos. La biblioteca estándar de Python `xml.etree.ElementTree` ofrece control total sobre el árbol.

```python
import xml.etree.ElementTree as ET
import pandas as pd

# Paso 1: Cargar y parsear el archivo
tree = ET.parse('facturas.xml')
root = tree.getroot()

# Explorar la estructura
print(f"Tag raíz: {root.tag}")                          # facturacion
print(f"Número de facturas: {len(root.findall('factura'))}")  # 3

# Ver el primer elemento hijo
primera_factura = root[0]
print(f"Tag: {primera_factura.tag}")                    # factura
print(f"Atributos: {primera_factura.attrib}")           # {'id': '1001'}
```

```python
# Paso 2: Extraer datos básicos (sin items)
datos = []

for factura in root.findall('factura'):
    fila = {
        # Atributo del tag <factura>
        'id_factura':  factura.attrib['id'],

        # Texto de los nodos hijo
        'cliente':     factura.find('cliente').text,
        'monto':       int(factura.find('monto').text),
        'fecha':       factura.find('fecha').text,

        # Atributo de un nodo hijo
        'moneda':      factura.find('monto').attrib.get('moneda', 'CLP')
    }
    datos.append(fila)

# Paso 3: Crear DataFrame
df_facturas = pd.DataFrame(datos)

# Paso 4: Conversión de tipos
df_facturas['fecha']       = pd.to_datetime(df_facturas['fecha'])
df_facturas['id_factura']  = df_facturas['id_factura'].astype(int)

print(df_facturas)
#    id_factura          cliente   monto      fecha moneda
# 0        1001    Empresa Alpha  500000 2026-01-15    CLP
# 1        1002        Tech Beta  750000 2026-01-20    CLP
# 2        1003 Industrias Gamma    2500 2026-02-01    USD
```

### Extrayendo nodos anidados (los `items`)

```python
# Extraer el detalle de ítems por factura (una fila por ítem)
datos_items = []

for factura in root.findall('factura'):
    id_factura = factura.attrib['id']
    cliente    = factura.find('cliente').text

    # Iterar sobre los items dentro de cada factura
    for item in factura.find('items').findall('item'):
        fila_item = {
            'id_factura':       id_factura,
            'cliente':          cliente,
            'descripcion':      item.find('descripcion').text,
            'cantidad':         int(item.find('cantidad').text),
            'precio_unitario':  float(item.find('precio_unitario').text),
        }
        # Calcular el subtotal
        fila_item['subtotal'] = fila_item['cantidad'] * fila_item['precio_unitario']
        datos_items.append(fila_item)

df_items = pd.DataFrame(datos_items)
print(df_items)
#   id_factura           cliente             descripcion  cantidad  precio_unitario   subtotal
# 0       1001     Empresa Alpha   Servicio de consultoría     10          50000.0   500000.0
# 1       1002         Tech Beta      Licencia de software      3         250000.0   750000.0
# 2       1003  Industrias Gamma         Hardware servidor      1           2500.0     2500.0
```

### Métodos clave de `ElementTree`

| Método | Descripción | Ejemplo |
|---|---|---|
| `ET.parse('archivo.xml')` | Carga y parsea el archivo | `tree = ET.parse('facturas.xml')` |
| `tree.getroot()` | Obtiene el nodo raíz | `root = tree.getroot()` |
| `root.findall('tag')` | Lista de todos los hijos con ese tag | `root.findall('factura')` |
| `elemento.find('tag')` | Primer hijo con ese tag | `factura.find('cliente')` |
| `elemento.text` | Contenido de texto del elemento | `factura.find('cliente').text` |
| `elemento.attrib` | Diccionario de atributos | `factura.attrib['id']` |
| `elemento.attrib.get('key', 'default')` | Atributo con valor por defecto | `monto.attrib.get('moneda', 'CLP')` |
| `elemento.tag` | Nombre del tag | `factura.tag` → `'factura'` |

> **⚠️ Buena práctica:** Usa siempre `attrib.get('clave', 'default')` en lugar de `attrib['clave']`. Si el atributo no existe, `attrib['clave']` lanza `KeyError`; `attrib.get(...)` devuelve un valor seguro. En XML real, los atributos opcionales son frecuentes.

---

## Desafío Práctico 3 — XML: Convertir fechas y derivar columnas temporales

### Modificar el ciclo `for` para extraer la fecha

```python
import xml.etree.ElementTree as ET
import pandas as pd

tree = ET.parse('facturas.xml')
root = tree.getroot()

datos = []
for factura in root.findall('factura'):
    fila = {
        'id_factura': factura.attrib['id'],
        'cliente':    factura.find('cliente').text,
        'monto':      int(factura.find('monto').text),
        'moneda':     factura.find('monto').attrib.get('moneda', 'CLP'),
        'fecha':      factura.find('fecha').text
    }
    datos.append(fila)

df = pd.DataFrame(datos)

# Convertir columna fecha a datetime de Pandas
df['fecha'] = pd.to_datetime(df['fecha'], format='%Y-%m-%d')

# Columnas derivadas ahora que tenemos fecha real
df['mes']         = df['fecha'].dt.month
df['trimestre']   = df['fecha'].dt.quarter
df['año']         = df['fecha'].dt.year

print(df.dtypes)
# id_factura            object
# cliente               object
# monto                  int64
# moneda                object
# fecha         datetime64[ns]   ← ahora sí es datetime
# mes                    int32
# trimestre              int32
# año                    int32

print(df)
```

---

# Sección 4: Arquitectura Data Engineer — Del Dato Crudo al Valor

## Los formatos en el contexto del pipeline completo

```
Fuentes de Datos                Procesamiento              Destino Final
─────────────────               ─────────────             ──────────────

CSV (archivos planos)  ──┐                                Data Warehouse
  ventas.csv              │                               PostgreSQL
  clientes.csv            ├──► INGESTA ──► LIMPIEZA ──►  BigQuery
                          │    pandas      dtype fix       Redshift
JSON (APIs REST)    ──────┤    requests    null handling
  /api/v1/pedidos         │    kaggle api  normalization  Data Lake
  /api/v1/usuarios        │                               S3 / GCS
                          │                               Parquet / Avro
XML (sistemas legacy)─────┘                               
  facturas.xml                                            Consumidores
  dte_sii.xml             ▲                              ─────────────
  soap_response.xml       │                               Data Scientists
                          │                               Analistas de Negocio
                          └── TÚ eres responsable             Dashboards
                              de este proceso
```

## Comparación de formatos: cuándo usar cada uno

| Criterio | CSV | JSON | XML |
|---|---|---|---|
| **Legibilidad** | ✅ Muy simple | ✅ Moderada | ⚠️ Verboso |
| **Datos estructurados** | ✅ Ideal | ⚠️ Requiere aplanamiento | ⚠️ Requiere parseo |
| **Datos anidados** | ❌ No soporta | ✅ Nativo | ✅ Nativo |
| **APIs REST** | Raro | ✅ Estándar | Raro |
| **Sistemas legacy** | ⚠️ Común | Raro | ✅ Dominante |
| **Tamaño en disco** | ✅ Compacto | ✅ Moderado | ❌ Muy verboso |
| **Validación de esquema** | ❌ Ninguna | ⚠️ JSON Schema | ✅ XSD nativo |
| **Velocidad de lectura con Pandas** | ✅ Muy rápida | ✅ Rápida | ⚠️ Lenta |
| **Producción / Data Warehouse** | ⚠️ Aceptable | ⚠️ Previa transformación | ⚠️ Previa transformación |
| **Formato optimizado** | → Parquet | → Parquet | → Parquet |

> **Nota sobre Parquet y Avro:** estos formatos columnares son el destino final en la mayoría de arquitecturas modernas. Por ahora, lo importante es entender que CSV, JSON y XML son siempre **formatos de ingesta**, no de almacenamiento en producción.

---

## El flujo de trabajo estándar de un Data Engineer

```python
# ============================================================
# Pipeline completo: del dato crudo al DataFrame limpio
# ============================================================

import pandas as pd
import requests
import xml.etree.ElementTree as ET
import json

def ingestar_csv(ruta, **kwargs):
    """Ingesta robusta de CSV con parámetros de producción."""
    df = pd.read_csv(
        ruta,
        na_values=['', 'NaN', 'N/A', 'null', 'NULL', 'None', 'Not Given'],
        encoding='utf-8',
        **kwargs
    )
    print(f"[CSV] Cargadas {len(df):,} filas, {df.shape[1]} columnas")
    return df


def ingestar_api_json(url, record_path=None, params=None):
    """Ingesta y aplanamiento de JSON desde API REST."""
    respuesta = requests.get(url, params=params)
    respuesta.raise_for_status()   # Lanza excepción si HTTP != 200
    datos = respuesta.json()

    if record_path:
        # Navegar hasta el array de registros
        for key in record_path:
            datos = datos[key]

    df = pd.json_normalize(datos)
    df.columns = df.columns.str.replace('.', '_', regex=False)
    print(f"[JSON] Cargados {len(df):,} registros, {df.shape[1]} columnas")
    return df


def ingestar_xml(ruta, tag_registros):
    """Parseo de XML a DataFrame plano."""
    tree = ET.parse(ruta)
    root = tree.getroot()
    registros = []

    for elemento in root.findall(tag_registros):
        fila = {}
        # Capturar atributos del elemento
        fila.update(elemento.attrib)
        # Capturar texto de los hijos directos
        for hijo in elemento:
            if len(hijo) == 0:   # Es un nodo hoja (sin hijos)
                fila[hijo.tag] = hijo.text
        registros.append(fila)

    df = pd.DataFrame(registros)
    print(f"[XML] Cargados {len(df):,} registros, {df.shape[1]} columnas")
    return df


# Uso del pipeline
df_netflix  = ingestar_csv('netflix_titles.csv')
df_usuarios = ingestar_api_json(
    'https://randomuser.me/api/',
    record_path=['results'],
    params={'results': 50}
)
df_facturas = ingestar_xml('facturas.xml', tag_registros='factura')
```

> **💡 Patrón de diseño:** las tres funciones tienen la misma firma conceptual: reciben una fuente, devuelven un DataFrame y emiten un log de cuántos registros se cargaron. Esta uniformidad es lo que permite construir pipelines complejos: un orquestador (Airflow, Prefect, Dagster) puede combinarlas sin saber el formato subyacente.

---

## Resumen de herramientas por formato

```python
# ── CSV ─────────────────────────────────────────────────────
import pandas as pd

df = pd.read_csv('archivo.csv', encoding='utf-8', na_values=['', 'N/A'])
df.info()                                    # Diagnóstico de tipos y nulos
df.isnull().sum()                            # Conteo de nulos por columna
df['col'] = pd.to_datetime(df['col'])        # Convertir a datetime
df['col'] = df['col'].fillna('default')      # Rellenar nulos
df.to_csv('salida.csv', index=False)         # Guardar resultado


# ── JSON ─────────────────────────────────────────────────────
import requests

respuesta = requests.get(url)
datos     = respuesta.json()                 # Dict Python
df        = pd.json_normalize(datos['key'])  # Aplanar jerarquía
df.columns = df.columns.str.replace('.', '_', regex=False)  # Limpiar nombres
df.to_csv('salida.csv', index=False)


# ── XML ──────────────────────────────────────────────────────
import xml.etree.ElementTree as ET

tree  = ET.parse('archivo.xml')
root  = tree.getroot()

for elemento in root.findall('tag'):
    valor_atributo = elemento.attrib['nombre_atributo']
    valor_texto    = elemento.find('hijo').text
    valor_con_def  = elemento.attrib.get('opcional', 'por_defecto')
```

---

## Ejercicios propuestos

### Ejercicio 1 — CSV: limpieza completa

**Enunciado:** Descarga el dataset de Netflix y construye un pipeline de limpieza que:

1. Cargue el CSV con detección correcta de nulos (`na_values`).
2. Convierta `date_added` a datetime.
3. Llene los nulos de `director` con `'Desconocido'`.
4. Separe la columna `duration` en `duracion_numero` (float) y `duracion_unidad` (str).
5. Guarde el resultado en `netflix_procesado.csv`.

<details>
<summary>Ver solución</summary>

```python
import pandas as pd

# 1. Cargar con nulos detectados
df = pd.read_csv('netflix_titles.csv', na_values=['', 'NaN', 'Not Given'])

# 2. Convertir fecha
df['date_added'] = pd.to_datetime(df['date_added'], format='mixed', errors='coerce')

# 3. Rellenar nulos de director
df['director'] = df['director'].fillna('Desconocido')

# 4. Separar duration
df['duracion_numero'] = df['duration'].str.extract(r'(\d+)').astype(float)
df['duracion_unidad'] = df['duration'].str.extract(r'([a-zA-Z]+)')

# 5. Guardar
df.to_csv('netflix_procesado.csv', index=False)
print(f"Guardado: {len(df):,} filas")
```

</details>

---

### Ejercicio 2 — JSON: pipeline de API con paginación

**Enunciado:** Construye una función que consuma la API `https://randomuser.me/api/` en 3 llamadas de 50 usuarios cada una, concatene los resultados y devuelva un DataFrame limpio con las columnas: `nombre_completo`, `email`, `pais`, `edad`.

<details>
<summary>Ver solución</summary>

```python
import requests
import pandas as pd

def obtener_usuarios(total=150, por_pagina=50):
    todos = []
    llamadas = total // por_pagina

    for i in range(llamadas):
        respuesta = requests.get(
            "https://randomuser.me/api/",
            params={'results': por_pagina, 'nat': 'cl,ar,mx,co,pe'}
        )
        datos = respuesta.json()
        df_pagina = pd.json_normalize(datos['results'])
        todos.append(df_pagina)
        print(f"  Página {i+1}/{llamadas}: {len(df_pagina)} usuarios")

    df = pd.concat(todos, ignore_index=True)
    df.columns = df.columns.str.replace('.', '_', regex=False)

    df['nombre_completo'] = df['name_first'] + ' ' + df['name_last']

    return df[['nombre_completo', 'email', 'location_country', 'dob_age']].rename(
        columns={'location_country': 'pais', 'dob_age': 'edad'}
    )

df_usuarios = obtener_usuarios()
print(df_usuarios.head())
print(f"\nTotal: {len(df_usuarios)} usuarios")
```

</details>

---

### Ejercicio 3 — XML: parseo con múltiples ítems por factura

**Enunciado:** Dado el XML de facturas con nodos `<items>`, escribe un pipeline que:

1. Extraiga una fila por cada ítem (no por factura).
2. Incluya `id_factura`, `cliente`, `moneda`, `fecha`, `descripcion`, `cantidad`, `precio_unitario`.
3. Calcule la columna `subtotal = cantidad * precio_unitario`.
4. Calcule el total por factura usando `groupby`.

<details>
<summary>Ver solución</summary>

```python
import xml.etree.ElementTree as ET
import pandas as pd

tree = ET.parse('facturas.xml')
root = tree.getroot()

filas = []
for factura in root.findall('factura'):
    id_f    = factura.attrib['id']
    cliente = factura.find('cliente').text
    moneda  = factura.find('monto').attrib.get('moneda', 'CLP')
    fecha   = factura.find('fecha').text

    items_node = factura.find('items')
    if items_node is not None:
        for item in items_node.findall('item'):
            cantidad        = int(item.find('cantidad').text)
            precio_unitario = float(item.find('precio_unitario').text)
            filas.append({
                'id_factura':      id_f,
                'cliente':         cliente,
                'moneda':          moneda,
                'fecha':           fecha,
                'descripcion':     item.find('descripcion').text,
                'cantidad':        cantidad,
                'precio_unitario': precio_unitario,
                'subtotal':        cantidad * precio_unitario
            })

df_items = pd.DataFrame(filas)
df_items['fecha'] = pd.to_datetime(df_items['fecha'])

# Total por factura
total_por_factura = df_items.groupby(['id_factura', 'cliente'])['subtotal'].sum().reset_index()
total_por_factura.columns = ['id_factura', 'cliente', 'total_factura']
print(total_por_factura)
```

</details>

---

*→ Próximo paso: continúa con el [Módulo 02: Programación en Python](../../modulo-02-programacion/README.md), donde profundizarás en las herramientas que aquí usamos (Pandas, NumPy, requests) desde sus fundamentos.*

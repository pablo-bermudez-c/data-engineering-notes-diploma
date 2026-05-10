# Clase 4 — Bases Documentales y MongoDB

---

## Tabla de contenidos

1. [¿Qué es una base documental?](#bloque-1)
2. [Por qué surgen y cuándo conviene](#bloque-2)
3. [XML vs JSON vs BSON](#bloque-3)
4. [Jerarquía: documento, campo, colección](#bloque-4)
5. [Anidamiento (embedding) vs referenciación](#bloque-5)
6. [Errores comunes al diseñar documentos](#bloque-6)
7. [Indexación y `$lookup`](#bloque-7)
8. [MQL: lenguaje de consultas de MongoDB](#bloque-8)
9. [SQL vs MongoDB: comparación práctica](#bloque-9)
10. [Caso integrador y enfoque DE](#bloque-10)

---

## Bloque 1 — ¿Qué es una base documental? {#bloque-1}

Una **base de datos documental** almacena la información como **documentos semi-estructurados**, usualmente en formatos como **JSON** o **XML**.

En lugar de separar estrictamente los datos en muchas tablas, un documento puede contener:

- pares clave-valor;
- objetos anidados;
- arreglos;
- listas de objetos;
- estructuras jerárquicas completas.

```json
{
  "nombre": "Ana",
  "email": "ana@example.com",
  "direccion": {
    "ciudad": "Santiago",
    "pais": "Chile"
  },
  "telefonos": [
    { "tipo": "personal", "numero": "+56912345678" },
    { "tipo": "trabajo",  "numero": "+56223456789" }
  ]
}
```

En una base relacional, esta información se separaría en varias tablas (`persona`, `direccion`, `telefono`). En una documental, **se guarda como una sola unidad lógica**.

> 💡 **Conexión con la Clase 2 del Módulo 01:** ya viste JSON como formato de ingesta desde APIs. Aquí JSON deja de ser un formato de transporte y se convierte en **el formato de almacenamiento mismo**.

---

## Bloque 2 — Por qué surgen y cuándo conviene {#bloque-2}

### Esquemas rígidos en bases relacionales

En una base relacional, antes de insertar datos debes definir una estructura fija:

```sql
CREATE TABLE cliente (
    id_cliente INT PRIMARY KEY,
    nombre VARCHAR(100),
    email VARCHAR(100),
    ciudad VARCHAR(80)
);
```

Esto funciona bien cuando la estructura es estable. Pero en sistemas modernos, los datos suelen cambiar con frecuencia.

```json
// Documento 1
{ "nombre": "Ana", "email": "ana@example.com", "preferencias": {"idioma": "es"} }

// Documento 2
{ "nombre": "Luis", "email": "luis@example.com", "redes_sociales": {"github": "luis-dev"} }
```

En una base documental, ambos pueden convivir en la misma colección aunque no tengan los mismos campos.

### Datos semi-estructurados

Muchos datos en ingeniería de datos llegan desde:

- APIs REST;
- eventos de aplicaciones;
- *logs*;
- formularios dinámicos;
- archivos JSON;
- integraciones SaaS;
- mensajes de Kafka.

```json
{
  "event_type": "product_viewed",
  "timestamp": "2026-05-04T10:15:00Z",
  "user_id": "U123",
  "properties": {
    "product_id": "P900",
    "category": "notebooks",
    "source": "mobile_app"
  }
}
```

Este tipo de dato se parece más a un documento que a una fila tradicional.

### Cercanía con la programación web

Las APIs REST trabajan naturalmente con JSON. Guardar la estructura en una base documental evita transformarla inmediatamente:

```text
API → JSON raw → base documental / data lake → transformación → modelo analítico
```

### Ventajas y limitaciones

| Ventaja | Explicación |
|---|---|
| Esquema flexible | Documentos no requieren los mismos campos |
| Escalabilidad horizontal | Diseñadas para distribución |
| Buen rendimiento en lecturas específicas | Una consulta puede recuperar todo de una sola vez |
| Cercanía con JSON | Integración natural con APIs y microservicios |
| Buen ajuste para semi-estructurados | Variabilidad sin rediseñar todo el esquema |

| Limitación | Riesgo |
|---|---|
| Menor soporte para relaciones complejas | No reemplaza un modelo altamente relacional |
| Redundancia | Duplicación de datos para optimizar consultas |
| Posibles inconsistencias | Si se duplica, actualizar exige disciplina |
| Consultas globales costosas | Agregaciones masivas pueden requerir índices |
| Documentos demasiado grandes | Afectan rendimiento, concurrencia y escalabilidad |

---

## Bloque 3 — XML vs JSON vs BSON {#bloque-3}

### XML

Las primeras bases documentales se apoyaron en **XML**:

```xml
<empleado>
  <nombre>Ana Gómez</nombre>
  <edad>40</edad>
</empleado>
```

**Ventajas históricas:** auto-descriptivo, soporte de validación (XSD), uso extendido en sistemas empresariales.

**Desventajas:** muchas etiquetas repetitivas, ocupa más espacio, parsing más costoso, verboso.

### JSON

```json
{
  "empleado": {
    "nombre": "Ana Gómez",
    "edad": 40
  }
}
```

**Ventajas:** liviano, fácil de leer, natural para web, fácil de serializar, compatible con muchas herramientas.

JSON se volvió dominante en aplicaciones modernas.

### BSON

**MongoDB usa internamente BSON** (*Binary JSON*). Es una codificación binaria de documentos JSON, con:

- menor costo de *parsing* que JSON textual;
- soporte para tipos adicionales (`ObjectId`, `DateTime`, `Binary`, `Decimal128`);
- mejor representación de fechas y datos binarios.

```text
JSON estándar:        number, string, boolean, array, object, null
BSON añade:           ObjectId, DateTime, Binary, Decimal128
```

> 💡 **Por qué importa BSON:** un campo de fecha en JSON estándar es solo un *string* (`"2026-05-04T10:00:00Z"`). En BSON, es un tipo `DateTime` real, lo que permite indexar, filtrar y operar fechas eficientemente sin parsear strings.

---

## Bloque 4 — Jerarquía: documento, campo, colección {#bloque-4}

| Concepto MongoDB | Equivalente relacional | Descripción |
|---|---|---|
| Documento | Fila | Unidad básica de almacenamiento |
| Campo | Columna | Par clave-valor dentro del documento |
| Colección | Tabla | Conjunto de documentos con propósito común |
| Base de datos | Base de datos | Contenedor de colecciones |

> ⚠️ **La equivalencia es aproximada.** Una colección **no obliga** a que todos los documentos tengan la misma estructura.

```json
// Documento 1 en colección "usuarios"
{
  "_id": "U1",
  "nombre": "Juan",
  "email": "juan@example.com"
}

// Documento 2 en la misma colección
{
  "_id": "U2",
  "nombre": "Ana",
  "email": "ana@example.com",
  "telefonos": ["+56911111111", "+56922222222"]
}
```

Ambos pertenecen a `usuarios`, aunque tengan campos distintos.

---

## Bloque 5 — Anidamiento (embedding) vs referenciación {#bloque-5}

Una de las decisiones más importantes al modelar documentos: ¿los datos relacionados van **anidados** dentro del mismo documento o **separados** y conectados con referencias?

### Anidamiento (embedding)

Guardar datos relacionados dentro del mismo documento:

```json
{
  "_id": "O1001",
  "fecha": "2026-05-04",
  "cliente": {
    "id": "U1",
    "nombre": "Juan Pérez"
  },
  "items": [
    {
      "producto_id": "P1",
      "nombre": "Laptop",
      "precio": 800,
      "cantidad": 1
    }
  ]
}
```

**Conviene cuando:**

- los datos se consultan juntos;
- la relación es de composición natural;
- el subdocumento no crece indefinidamente;
- se quiere evitar múltiples consultas;
- la actualización del subdocumento ocurre junto con el padre.

```text
pedido     → items del pedido
usuario    → direcciones frecuentes
producto   → atributos técnicos
```

### Referenciación

Guardar identificadores que apuntan a otros documentos:

```json
// Pedido
{ "_id": "O1001", "cliente_id": "U1", "fecha": "2026-05-04" }

// Cliente (en otra colección)
{ "_id": "U1", "nombre": "Juan Pérez", "email": "juan@example.com" }
```

**Conviene cuando:**

- los datos relacionados son grandes;
- cambian de forma independiente;
- se consultan por separado;
- hay relaciones muchos-a-muchos;
- el arreglo anidado podría crecer sin límite.

```text
post     → comentarios masivos
usuario  → pedidos históricos
autor    → libros
```

### Regla práctica

```text
Si se consulta junto y no crece sin control  → ANIDA
Si crece mucho, se reutiliza o cambia aparte → REFERENCIA
```

> 💡 **El error más común** es replicar el modelo relacional con muchas colecciones pequeñas y `$lookup` (JOINs documentales). MongoDB rinde mejor cuando el documento ya contiene lo necesario para las consultas frecuentes.

---

## Bloque 6 — Errores comunes al diseñar documentos {#bloque-6}

### Error 1: pensar en tablas

Diseño demasiado relacional:

```json
// usuarios
{ "_id": "U1", "nombre": "Juan Pérez" }

// productos
{ "_id": "P1", "nombre": "Laptop", "precio": 800 }

// pedidos
{ "_id": "O1001", "usuario_id": "U1" }

// items (tabla puente)
{ "_id": "OI1", "pedido_id": "O1001", "producto_id": "P1", "cantidad": 1 }
```

Esto fuerza a leer 4 colecciones para reconstruir un pedido.

**Mejor:**

```json
{
  "_id": "O1001",
  "fecha": "2026-05-04",
  "cliente": { "id": "U1", "nombre": "Juan Pérez" },
  "items": [
    { "producto_id": "P1", "nombre": "Laptop", "precio": 800, "cantidad": 1 }
  ]
}
```

Un solo documento contiene todo el pedido.

### Error 2: no diseñar según consultas

Si los comentarios siempre se ven con el post, embebe:

```json
{
  "_id": "P1",
  "titulo": "Bases de datos NoSQL",
  "comentarios": [
    { "usuario": "Juan", "texto": "...", "fecha": "2026-05-01" }
  ]
}
```

Pero si necesitas:

- *"comentarios recientes de todos los posts"*;
- *"buscar comentarios por usuario"*;
- *"moderar comentarios por fecha"*;

es mejor separar:

```json
// posts
{ "_id": "P1", "titulo": "Bases de datos NoSQL" }

// comentarios
{ "_id": "C1", "post_id": "P1", "usuario": "Juan", "fecha": "2026-05-01" }
```

Con índices en `post_id`, `usuario` y `fecha`.

### Error 3: documentos gigantes

> ⚠️ **MongoDB tiene límite máximo de tamaño por documento (16 MB).** Pero mucho antes del límite, los documentos gigantes ya causan problemas: lecturas ineficientes, escrituras costosas, problemas de concurrencia, mayor uso de red y memoria.

**Mal diseño** — un usuario con todos sus eventos embebidos:

```json
{
  "_id": "usuario_1",
  "nombre": "Ana",
  "eventos": [
    { "tipo": "login", "fecha": "2026-01-01" },
    { "tipo": "click", "fecha": "2026-01-01" },
    // ... millones de eventos crecen sin límite
  ]
}
```

**Mejor:**

```json
// usuarios
{ "_id": "usuario_1", "nombre": "Ana" }

// eventos_usuario
{ "usuario_id": "usuario_1", "tipo": "login", "fecha": "2026-01-01T10:00:00Z" }
```

---

## Bloque 7 — Indexación y `$lookup` {#bloque-7}

### Indexación

Sin índice, una consulta puede requerir un escaneo completo de la colección:

```text
collection scan → revisar documento por documento → O(n)
```

Con índice:

```text
index scan → estructura auxiliar → acceder a documentos relevantes
```

Crear índice:

```javascript
db.pedidos.createIndex({ "items.producto_id": 1 })
```

(`1` es ascendente, `-1` descendente.)

### Buenas prácticas de indexación

1. Indexa campos usados en filtros frecuentes.
2. Indexa campos usados en ordenamientos frecuentes.
3. Evita crear índices innecesarios: aceleran lectura pero encarecen escritura.
4. Revisa el plan de ejecución con `explain()`.
5. Usa índices compuestos cuando la consulta filtra por varios campos.

```javascript
// Índice compuesto
db.pedidos.createIndex({ cliente_id: 1, fecha: -1 })

// Aprovecha el índice anterior
db.pedidos.find({ cliente_id: "U1" }).sort({ fecha: -1 })
```

### `$lookup`: el join documental

MongoDB permite operaciones similares a un `JOIN` con `$lookup` dentro de un *pipeline* de agregación.

```javascript
db.pedidos.aggregate([
  {
    $lookup: {
      from: "usuarios",
      localField: "usuario_id",
      foreignField: "_id",
      as: "usuario"
    }
  },
  { $unwind: "$usuario" }    // convierte el array en objeto plano
])
```

Resultado:

```json
{
  "_id": "O1",
  "usuario_id": "U1",
  "total": 100,
  "usuario": {
    "_id": "U1",
    "nombre": "Juan"
  }
}
```

> ⚠️ **Aunque `$lookup` existe, no significa que debas modelar MongoDB como base relacional.** Es una herramienta para casos específicos, no la operación predeterminada. Si te encuentras usando `$lookup` constantemente, considera embeber los datos juntos.

---

## Bloque 8 — MQL: lenguaje de consultas de MongoDB {#bloque-8}

MongoDB usa un lenguaje inspirado en JSON, llamado **MQL**.

### CRUD básico

#### Crear o usar una base

```javascript
use empresa
```

MongoDB crea la base cuando se inserta el primer dato.

#### Crear una colección

```javascript
db.createCollection("empleado")
```

(En MongoDB no necesitas definir todas las columnas antes de insertar, aunque puedes usar **validación de esquema** si el proyecto lo requiere.)

#### Insertar

```javascript
db.empleado.insertOne({
  _id: 1,
  nombre: "Juan",
  edad: 28,
  grado: 9,
  dep: 10
})

db.empleado.insertMany([
  { _id: 2, nombre: "Pablo", edad: 25, grado: 10, dep: 10 },
  { _id: 3, nombre: "Ana",   edad: 26, grado: 9,  dep: 20 }
])
```

#### Actualizar

```javascript
// Una sola
db.empleado.updateOne(
  { _id: 2 },
  { $set: { grado: 11 } }
)

// Múltiples
db.empleado.updateMany(
  { dep: 10 },
  { $set: { activo: true } }
)
```

#### Eliminar

```javascript
db.empleado.deleteOne({ _id: 2 })
db.empleado.deleteMany({ dep: 10 })
```

#### Crear índice

```javascript
db.empleado.createIndex({ dep: 1 })
```

---

## Bloque 9 — SQL vs MongoDB: comparación práctica {#bloque-9}

### Filtros

```sql
-- SQL
SELECT * FROM empleado
WHERE edad < 30 AND grado > 8;
```

```javascript
// MongoDB
db.empleado.find({
  edad: { $lt: 30 },
  grado: { $gt: 8 }
})
```

### IN

```sql
-- SQL
SELECT * FROM empleado
WHERE nombre IN ('Pablo', 'Ana');
```

```javascript
// MongoDB
db.empleado.find({
  nombre: { $in: ["Pablo", "Ana"] }
})
```

### Búsqueda con patrón

```sql
-- SQL
SELECT * FROM empleado
WHERE nombre LIKE '%bio%';
```

```javascript
// MongoDB
db.empleado.find({
  nombre: /bio/
})

// Con expresión regular explícita y sensible a mayúsculas
db.empleado.find({
  nombre: { $regex: "bio", $options: "i" }
})
```

### Agregación simple

```sql
-- SQL
SELECT SUM(edad) AS total FROM empleado;
```

```javascript
// MongoDB
db.empleado.aggregate([
  {
    $group: {
      _id: null,
      total: { $sum: "$edad" }
    }
  }
])
```

### Agregación por grupo

```sql
-- SQL
SELECT dep, MAX(grado) AS mayor_grado
FROM empleado
GROUP BY dep;
```

```javascript
// MongoDB
db.empleado.aggregate([
  {
    $group: {
      _id: "$dep",
      mayor_grado: { $max: "$grado" }
    }
  }
])
```

### Resumen comparativo

| Aspecto | SQL relacional | MongoDB documental |
|---|---|---|
| Modelo | Tablas y filas | Documentos JSON/BSON |
| Estructura | Esquema rígido | Esquema flexible |
| Unidad principal | Fila | Documento |
| Relaciones | Claves foráneas | Referencias o *embedding* |
| *Joins* | Nativos y optimizados | `$lookup`, más costoso |
| Transacciones | Fuertes, ACID | Soportadas pero no son el foco |
| Consistencia | Alta, ACID | Variable según configuración |
| Escalabilidad | Vertical principalmente | Horizontal (*sharding*) |
| Normalización | Muy usada | Menos común; redundancia controlada |
| Casos ideales | ERP, finanzas, transaccional | APIs, *logs*, catálogos, perfiles |

> 💡 **La mentalidad correcta:**
>
> ```
> Relacional   → entidades → relaciones → normalización → consultas
> Documental   → consultas → documentos → índices → redundancia controlada
> ```

---

## Bloque 10 — Caso integrador y enfoque DE {#bloque-10}

### Sistema de pedidos en MongoDB

Una aplicación necesita responder principalmente:

1. Detalle completo de un pedido.
2. Pedidos recientes de un cliente.
3. Pedidos que contienen cierto producto.

**Diseño:**

```json
{
  "_id": "O1001",
  "cliente": {
    "id": "U1",
    "nombre": "Juan Pérez",
    "email": "juan@example.com"
  },
  "fecha": "2026-05-04T10:00:00Z",
  "estado": "pagado",
  "items": [
    {
      "producto_id": "P1",
      "nombre": "Laptop",
      "categoria": "computadores",
      "precio_unitario": 800,
      "cantidad": 1
    },
    {
      "producto_id": "P2",
      "nombre": "Mouse",
      "categoria": "accesorios",
      "precio_unitario": 20,
      "cantidad": 2
    }
  ],
  "total": 840
}
```

**Índices:**

```javascript
db.pedidos.createIndex({ "cliente.id": 1, fecha: -1 })
db.pedidos.createIndex({ "items.producto_id": 1 })
db.pedidos.createIndex({ estado: 1 })
```

**Consultas:**

```javascript
// 1. Detalle del pedido
db.pedidos.findOne({ _id: "O1001" })

// 2. Pedidos recientes de un cliente
db.pedidos
  .find({ "cliente.id": "U1" })
  .sort({ fecha: -1 })

// 3. Pedidos que contienen cierto producto
db.pedidos.find({ "items.producto_id": "P1" })
```

### MongoDB en pipelines de datos

#### Ingesta

Muchas fuentes entregan JSON:

```text
API REST → JSON → MongoDB / Data Lake / Kafka
```

#### Zona raw

En pipelines analíticos, es común guardar el documento original antes de transformar:

```text
raw_events
raw_orders
raw_users
```

Esto permite auditar datos originales, reprocesar pipelines, adaptarse a cambios de esquema y preservar campos no usados aún.

#### Transformación hacia analítica

```text
MongoDB → ETL/ELT → Data Warehouse → BI

fact_pedidos
dim_cliente
dim_producto
dim_fecha
```

Permite consultas OLAP eficientes con SQL.

#### Calidad de datos

> ⚠️ **El esquema flexible no significa ausencia de reglas.** En sistemas productivos conviene controlar:
>
> - campos obligatorios;
> - tipos de datos;
> - formatos de fecha;
> - claves de negocio;
> - duplicados;
> - versiones del documento.

```json
{
  "_id": "U1",
  "schema_version": 2,
  "nombre": "Ana",
  "email": "ana@example.com"
}
```

El campo `schema_version` permite que un *job* de ETL maneje versiones distintas del mismo tipo de documento.

---

## Buenas prácticas

- Diseñar desde las consultas, no desde tablas normalizadas.
- Usar *embedding* cuando los datos se leen juntos y no crecen indefinidamente.
- Usar referencias cuando los datos crecen mucho o se consultan de forma independiente.
- Crear índices alineados con filtros y ordenamientos reales.
- Evitar documentos gigantes.
- Aceptar redundancia, pero controlarla conscientemente.
- Documentar decisiones de modelado.
- Validar esquemas cuando el sistema requiere calidad estricta.
- Pensar en el *pipeline* completo: ingesta, operación, transformación y analítica.

## Errores frecuentes

| Error | Consecuencia | Mejor alternativa |
|---|---|---|
| Copiar el modelo relacional | Demasiadas colecciones y `$lookup` innecesarios | Modelar por consulta |
| Anidar arreglos infinitos | Documentos enormes y lentos | Separar en colección propia |
| No crear índices | Escaneos completos | Indexar campos consultados |
| Indexar todo | Escrituras más lentas y más uso de disco | Indexar con criterio |
| Duplicar sin estrategia | Inconsistencias | Redundancia controlada |
| Ignorar analítica | Dificulta BI y *reporting* | Diseñar extracción hacia warehouse |

---

<details>
<summary><strong>🟢 Ejercicio 1 — Diseño de blog (click para ver)</strong></summary>

Diseña colecciones para un sistema de blog que debe responder:

1. Mostrar el detalle de un post.
2. Mostrar los últimos comentarios de todos los posts.
3. Buscar posts por autor.
4. Moderar comentarios por usuario.

Decide qué anidar y qué referenciar.

**Solución:**

Como hay consultas globales sobre comentarios (2 y 4), conviene **separarlos** en una colección. Los autores se referencian por id pero se replica `nombre` para la consulta 3:

```javascript
// Colección posts
{
  "_id": "P1",
  "titulo": "Bases de datos NoSQL",
  "contenido": "...",
  "autor": { "id": "U1", "nombre": "Juan" },
  "fecha": "2026-05-01",
  "tags": ["nosql", "mongodb"]
}

// Colección comentarios
{
  "_id": "C1",
  "post_id": "P1",
  "usuario_id": "U2",
  "usuario_nombre": "Ana",
  "texto": "Excelente",
  "fecha": "2026-05-02"
}

// Índices
db.posts.createIndex({ "autor.id": 1, fecha: -1 })       // Q3
db.comentarios.createIndex({ fecha: -1 })                 // Q2
db.comentarios.createIndex({ usuario_id: 1, fecha: -1 })  // Q4
db.comentarios.createIndex({ post_id: 1 })                // listar comentarios de un post
```

</details>

<details>
<summary><strong>🟢 Ejercicio 2 — Catálogo de productos heterogéneos (click para ver)</strong></summary>

Diseña documentos para productos donde cada categoría tiene atributos distintos:

```text
notebook  → RAM, CPU, almacenamiento
zapato    → talla, color, material
libro     → autor, editorial, ISBN
```

**Solución:**

Una sola colección `productos` con campos comunes y un sub-objeto `atributos` flexible:

```javascript
{
  "_id": "P900",
  "nombre": "Laptop XYZ 15",
  "categoria": "notebook",
  "precio": 1200,
  "stock": 15,
  "atributos": {
    "ram_gb": 16,
    "cpu": "Intel i7-13700H",
    "almacenamiento_gb": 512
  }
}

{
  "_id": "P200",
  "nombre": "Zapato Deportivo Plus",
  "categoria": "zapato",
  "precio": 89,
  "stock": 50,
  "atributos": {
    "talla": 42,
    "color": "azul",
    "material": "cuero sintético"
  }
}
```

Índices recomendados:

```javascript
db.productos.createIndex({ categoria: 1, precio: 1 })
db.productos.createIndex({ "atributos.ram_gb": 1 })  // si se filtra
db.productos.createIndex({ nombre: "text" })         // búsqueda full-text
```

</details>

<details>
<summary><strong>🟢 Ejercicio 3 — Eventos de aplicación (click para ver)</strong></summary>

Diseña una colección para almacenar eventos JSON y propone índices para:

1. eventos por usuario;
2. eventos por rango de fecha;
3. compras por monto;
4. eventos por tipo.

```json
{ "event_type": "login", "user_id": "U1", "timestamp": "2026-05-04T10:00:00Z", "device": "mobile" }
{ "event_type": "purchase", "user_id": "U1", "timestamp": "2026-05-04T10:05:00Z", "amount": 840 }
```

**Solución:**

Una sola colección `eventos` con índices alineados:

```javascript
// Documento típico
{
  "_id": ObjectId(...),
  "event_type": "purchase",
  "user_id": "U1",
  "timestamp": ISODate("2026-05-04T10:05:00Z"),
  "amount": 840,
  "items": ["P1", "P2"]
}

// Índices
db.eventos.createIndex({ user_id: 1, timestamp: -1 })       // Q1
db.eventos.createIndex({ timestamp: -1 })                    // Q2
db.eventos.createIndex({ event_type: 1, amount: -1 })        // Q3 (compras por monto)
db.eventos.createIndex({ event_type: 1, timestamp: -1 })     // Q4
```

Considerar también particionamiento por fecha (TTL automático) si los eventos antiguos se archivan a otro almacenamiento:

```javascript
db.eventos.createIndex({ timestamp: 1 }, { expireAfterSeconds: 7776000 })  // 90 días
```

</details>

---

## Referencia rápida — MongoDB

```
JERARQUÍA
─────────────────────────────────────────────────────────────────
  Documento     ≈ fila (JSON/BSON)
  Campo         ≈ columna (par clave-valor)
  Colección     ≈ tabla (esquema flexible)
  Base de datos ≈ base de datos

EMBEDDING vs REFERENCING
─────────────────────────────────────────────────────────────────
  Embedding     se consultan juntos, no crece sin límite
  Referencing   crece mucho, cambia aparte, M:N

OPERADORES DE FILTRO
─────────────────────────────────────────────────────────────────
  $eq $ne                       igualdad
  $gt $gte $lt $lte             comparación
  $in $nin                      pertenencia
  $regex                        patrón de texto
  $exists                       presencia del campo
  $and $or $not                 lógicos

CRUD
─────────────────────────────────────────────────────────────────
  insertOne / insertMany
  find / findOne
  updateOne / updateMany        + $set, $inc, $push, $pull
  deleteOne / deleteMany

AGREGACIÓN (pipeline)
─────────────────────────────────────────────────────────────────
  $match     filtrar
  $group     agrupar
  $project   seleccionar campos
  $sort      ordenar
  $limit     limitar
  $lookup    JOIN documental
  $unwind    expandir array

ÍNDICES
─────────────────────────────────────────────────────────────────
  createIndex({ campo: 1 })            simple
  createIndex({ a: 1, b: -1 })         compuesto
  createIndex({ texto: "text" })       full-text
  createIndex({ ts: 1 }, { expireAfterSeconds: N })  TTL

MENTALIDAD
─────────────────────────────────────────────────────────────────
  ✗ Modelar tablas + JOINs
  ✓ Modelar consultas → documentos → índices
```

---

*→ Próxima clase: [Bases de Grafos y Neo4j](../clase-05-bases-de-grafos-neo4j/README.md)*

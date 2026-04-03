# Clase 3: Consultas SQL y JOINs en PostgreSQL

---

# Hoja de Ruta

| Sección | Tema | Contenido |
|---|---|---|
| **01** | Consultas básicas | `SELECT`, `FROM`, alias, filtros esenciales |
| **02** | Filtros avanzados | `BETWEEN`, `IN`, `LIKE`, manejo de `NULL` |
| **03** | Control de resultados | `ORDER BY`, `LIMIT`, Top-N queries |
| **04** | JOINs de tablas | `INNER JOIN`, `LEFT JOIN` y variantes |

> **Herramientas:** pgAdmin (interfaz gráfica oficial) o DBeaver (alternativa multiplataforma). Ambas ofrecen un Query Tool para escribir y ejecutar consultas directamente.

---

# Sección 1: Consultas Básicas

## `SELECT` y `FROM` — El punto de partida

Toda consulta SQL comienza con la misma estructura fundamental: **qué quieres ver** (`SELECT`) y **de dónde** (`FROM`).

```sql
-- Sintaxis básica
SELECT columnas
FROM tabla;
```

### Seleccionar todas las columnas vs. columnas específicas

```sql
-- Trae TODAS las columnas de la tabla
SELECT *
FROM clientes;

-- Trae solo las columnas que necesitas (recomendado)
SELECT id, nombre, ciudad, email
FROM clientes;
```

> **⚠️ Advertencia de rendimiento:** En bases de datos productivas con tablas de millones de filas y decenas de columnas, `SELECT *` transfiere datos innecesarios desde el disco al motor y luego a tu cliente. Siempre que sea posible, especifica solo las columnas que vas a usar. Es uno de los hábitos más importantes para escribir SQL eficiente.

**Analogía para Data Engineering:** `SELECT *` es como hacer `pd.read_csv('archivo_50gb.csv')` en Python sin especificar `usecols`. Funciona, pero es costoso.

---

## Alias con `AS` — Renombrando columnas al vuelo

`AS` permite darle un nombre más legible a una columna en el resultado de la consulta, sin modificar la tabla original.

```sql
SELECT
    nombre          AS nombre_cliente,
    fecha_nac       AS fecha_nacimiento,
    ciudad          AS ciudad_origen,
    saldo_cuenta    AS saldo_usd
FROM clientes
WHERE edad > 18;
```

**Resultado:**

| nombre_cliente | fecha_nacimiento | ciudad_origen | saldo_usd |
|---|---|---|---|
| María González | 1992-04-15 | Santiago | 1500.00 |
| Carlos Pérez | 1988-11-03 | Valparaíso | 320.50 |

### Alias en columnas calculadas

Los alias son especialmente útiles cuando la columna no tiene nombre natural, como en operaciones aritméticas:

```sql
SELECT
    nombre,
    precio                              AS precio_original,
    precio * 1.19                       AS precio_con_iva,
    precio * 0.10                       AS descuento_10pct,
    (precio * 1.19) - (precio * 0.10)   AS precio_final
FROM productos;
```

> **Tip:** En PostgreSQL los alias con espacios o caracteres especiales van entre comillas dobles: `precio * 1.19 AS "Precio con IVA"`. Para nombres simples, las comillas son opcionales.

---

## Filtrando con `WHERE`

`WHERE` es el filtro principal de SQL. Define qué filas deben incluirse en el resultado.

### Operadores de comparación

| Operador | Significado | Ejemplo |
|---|---|---|
| `=`  | Igual | `WHERE ciudad = 'Santiago'` |
| `!=` o `<>` | Distinto | `WHERE estado != 'cancelado'` |
| `>`  | Mayor que | `WHERE precio > 1000` |
| `<`  | Menor que | `WHERE stock < 10` |
| `>=` | Mayor o igual | `WHERE edad >= 18` |
| `<=` | Menor o igual | `WHERE descuento <= 0.5` |

### Operadores lógicos: `AND`, `OR`, `NOT`

```sql
-- AND: ambas condiciones deben ser verdaderas
SELECT nombre, ciudad, edad
FROM clientes
WHERE ciudad = 'Santiago'
  AND edad > 18;

-- OR: al menos una condición debe ser verdadera
SELECT nombre, categoria
FROM productos
WHERE categoria = 'Electrónica'
   OR categoria = 'Computación';

-- NOT: invierte la condición
SELECT nombre, estado
FROM pedidos
WHERE NOT estado = 'cancelado';
-- equivalente a: WHERE estado != 'cancelado'
```

### Agrupación con paréntesis

Los paréntesis controlan el orden de evaluación cuando se combinan `AND` y `OR`. Sin ellos, el comportamiento puede no ser el esperado.

```sql
-- ❌ Ambiguo: ¿qué se evalúa primero?
SELECT * FROM pedidos
WHERE ciudad = 'Santiago' OR ciudad = 'Valparaíso' AND monto > 5000;

-- ✅ Explícito y correcto:
SELECT * FROM pedidos
WHERE (ciudad = 'Santiago' OR ciudad = 'Valparaíso')
  AND monto > 5000;
```

> **Regla:** `AND` tiene mayor precedencia que `OR` en SQL, igual que la multiplicación sobre la suma en matemáticas. Usa paréntesis ante cualquier duda.

---

# Sección 2: Filtros Avanzados

## `BETWEEN` — Filtros de rango

`BETWEEN` es azúcar sintáctica para `>= AND <=`. Hace las consultas más legibles, especialmente con fechas y rangos numéricos. **El rango es inclusivo en ambos extremos.**

```sql
-- Rango numérico: productos entre $100 y $500
SELECT nombre, precio
FROM productos
WHERE precio BETWEEN 100 AND 500;

-- Equivalente explícito:
WHERE precio >= 100 AND precio <= 500

-- Rango de fechas: ventas del primer trimestre 2026
SELECT id_venta, fecha, monto
FROM ventas
WHERE fecha BETWEEN '2026-01-01' AND '2026-03-31';

-- Rango de texto: clientes cuyo nombre empieza entre A y M
SELECT nombre
FROM clientes
WHERE nombre BETWEEN 'A' AND 'M';

-- Negación: fuera del rango
SELECT nombre, precio
FROM productos
WHERE precio NOT BETWEEN 100 AND 500;
```

> **Caso Data Engineering:** `BETWEEN` es muy frecuente al definir **ventanas de tiempo** en pipelines de ingesta incremental. Por ejemplo, procesar solo los registros de ayer: `WHERE fecha_registro BETWEEN '2026-03-20' AND '2026-03-20 23:59:59'`.

---

## `IN` — Filtros de lista

`IN` evita encadenar múltiples condiciones `OR`. Es más legible y en muchos motores también más eficiente.

```sql
-- Sin IN (verboso y propenso a errores)
SELECT nombre, ciudad
FROM clientes
WHERE ciudad = 'Santiago'
   OR ciudad = 'Valparaíso'
   OR ciudad = 'Concepción'
   OR ciudad = 'Antofagasta';

-- Con IN (limpio y directo)
SELECT nombre, ciudad
FROM clientes
WHERE ciudad IN ('Santiago', 'Valparaíso', 'Concepción', 'Antofagasta');

-- Negación: clientes que NO son de esas ciudades
SELECT nombre, ciudad
FROM clientes
WHERE ciudad NOT IN ('Santiago', 'Valparaíso');

-- IN con números
SELECT id, nombre, categoria_id
FROM productos
WHERE categoria_id IN (1, 3, 7, 12);
```

### `IN` con subconsulta (avanzado)

```sql
-- Clientes que tienen al menos un pedido activo
SELECT nombre, email
FROM clientes
WHERE id IN (
    SELECT DISTINCT cliente_id
    FROM pedidos
    WHERE estado = 'activo'
);
```

---

## `LIKE` e `ILIKE` — Búsquedas de texto con patrones

Permiten buscar texto mediante patrones con comodines, sin necesidad de conocer el valor exacto.

### Comodines disponibles

| Comodín | Significado | Ejemplo |
|---|---|---|
| `%` | Cero o más caracteres cualquiera | `'Juan%'` → Juan, Juanito, Juanita |
| `_` | Exactamente un carácter cualquiera | `'J_an'` → Juan, Jean |

```sql
-- Nombres que EMPIEZAN con 'Mar'
SELECT nombre FROM clientes
WHERE nombre LIKE 'Mar%';
-- Encuentra: María, Marcos, Marcela, Martín

-- Nombres que TERMINAN con 'ez'
SELECT nombre FROM clientes
WHERE nombre LIKE '%ez';
-- Encuentra: Pérez, López, Gómez, Martínez

-- Nombres que CONTIENEN 'and' en cualquier posición
SELECT nombre FROM clientes
WHERE nombre LIKE '%and%';
-- Encuentra: Fernanda, Alessandro, Alejandra

-- Exactamente 4 caracteres
SELECT codigo FROM productos
WHERE codigo LIKE '____';
-- Encuentra: A001, B234, X999

-- Patrones combinados: empieza con 'S', luego 3 chars, luego 'go'
SELECT ciudad FROM clientes
WHERE ciudad LIKE 'S___go';
-- Encuentra: Santiago
```

### `LIKE` vs `ILIKE` — La diferencia clave en PostgreSQL

```sql
-- LIKE: sensible a mayúsculas (estándar SQL)
SELECT nombre FROM clientes WHERE nombre LIKE 'juan%';
-- NO encuentra: Juan, JUAN, JuAn

-- ILIKE: insensible a mayúsculas (extensión de PostgreSQL ★)
SELECT nombre FROM clientes WHERE nombre ILIKE 'juan%';
-- SÍ encuentra: juan, Juan, JUAN, JuAn

-- Caso real: búsqueda en datos sucios donde el casing es inconsistente
SELECT nombre, email
FROM clientes
WHERE email ILIKE '%@gmail.com';
-- Encuentra: usuario@gmail.com, USUARIO@GMAIL.COM, Usuario@Gmail.Com
```

> **Nota:** `ILIKE` es una extensión propia de PostgreSQL. Si necesitas portabilidad a otros motores (MySQL, SQL Server), usa `LOWER(columna) LIKE 'patron'` como alternativa estándar.

> **⚠️ Rendimiento:** Los patrones con `%` al inicio (`'%texto'`) no pueden usar índices y realizan un full scan de tabla. En tablas grandes, considera usar índices de texto completo (`tsvector`) o extensiones como `pg_trgm`.

---

## El Misterio de los Valores `NULL`

`NULL` no es cero, no es una cadena vacía, no es `false`. Es la **ausencia total de un valor**. Esto tiene implicaciones importantes al comparar.

### Por qué `= NULL` nunca funciona

```sql
-- ❌ INCORRECTO: siempre devuelve 0 filas
SELECT * FROM clientes WHERE telefono = NULL;

-- ✅ CORRECTO: usar IS NULL
SELECT * FROM clientes WHERE telefono IS NULL;

-- ✅ CORRECTO: excluir nulos
SELECT * FROM clientes WHERE telefono IS NOT NULL;
```

La razón: en SQL, cualquier comparación con `NULL` devuelve `UNKNOWN` (un tercer estado lógico, ni `TRUE` ni `FALSE`). Por eso `NULL = NULL` tampoco es `TRUE`.

### Casos de uso en Data Engineering

```sql
-- Detectar registros incompletos (datos sucios en ingesta)
SELECT id_cliente, nombre, email, telefono
FROM clientes
WHERE email IS NULL OR telefono IS NULL;

-- Contar cuántos registros tienen datos faltantes
SELECT
    COUNT(*)                            AS total_registros,
    COUNT(email)                        AS con_email,
    COUNT(*) - COUNT(email)             AS sin_email,
    COUNT(telefono)                     AS con_telefono,
    COUNT(*) - COUNT(telefono)          AS sin_telefono
FROM clientes;

-- Reemplazar NULL por un valor por defecto en la consulta (COALESCE)
SELECT
    nombre,
    COALESCE(telefono, 'Sin teléfono')  AS telefono,
    COALESCE(email,    'Sin email')     AS email
FROM clientes;

-- Filtrar solo los registros completos para un proceso de transformación
SELECT id, nombre, email, telefono
FROM clientes
WHERE nombre    IS NOT NULL
  AND email     IS NOT NULL
  AND telefono  IS NOT NULL;
```

> **`COALESCE(valor, alternativa)`** devuelve el primer argumento que no sea `NULL`. Es la forma más común de manejar nulos en transformaciones SQL.

---

# Sección 3: Control de Resultados

## `ORDER BY` — Ordenamiento de resultados

`ORDER BY` define el criterio de ordenamiento del resultado. Por defecto, SQL no garantiza ningún orden sin esta cláusula.

```sql
-- Orden ascendente (A→Z, menor a mayor) — es el valor por defecto
SELECT nombre, precio
FROM productos
ORDER BY precio ASC;

-- Orden descendente (Z→A, mayor a menor)
SELECT nombre, precio
FROM productos
ORDER BY precio DESC;

-- Ordenar por múltiples columnas:
-- primero por ciudad (ASC), luego por nombre dentro de cada ciudad (ASC)
SELECT nombre, ciudad, fecha_registro
FROM clientes
ORDER BY ciudad ASC, nombre ASC;

-- Mezcla: ordenar por fecha descendente, luego por monto descendente
SELECT fecha, id_pedido, monto
FROM pedidos
ORDER BY fecha DESC, monto DESC;

-- Ordenar usando alias o posición numérica de columna (menos legible)
SELECT nombre, precio * 1.19 AS precio_con_iva
FROM productos
ORDER BY precio_con_iva DESC;
-- equivalente a: ORDER BY 2 DESC  (columna #2 del SELECT)
```

---

## `LIMIT` — Control de cantidad de filas

`LIMIT` restringe cuántas filas devuelve la consulta. Esencial para explorar datos, construir rankings y evitar sobrecargar el sistema.

```sql
-- Ver solo las primeras 10 filas (exploración rápida)
SELECT * FROM ventas LIMIT 10;

-- Top 5 productos más caros
SELECT nombre, precio
FROM productos
WHERE precio IS NOT NULL
ORDER BY precio DESC
LIMIT 5;

-- Top 3 productos más baratos (con datos limpios)
SELECT nombre, precio
FROM productos
WHERE precio IS NOT NULL
ORDER BY precio ASC
LIMIT 3;

-- Paginación con OFFSET (útil en APIs y reportes paginados)
-- Página 1: filas 1-10
SELECT id, nombre FROM clientes ORDER BY id LIMIT 10 OFFSET 0;
-- Página 2: filas 11-20
SELECT id, nombre FROM clientes ORDER BY id LIMIT 10 OFFSET 10;
-- Página 3: filas 21-30
SELECT id, nombre FROM clientes ORDER BY id LIMIT 10 OFFSET 20;
```

### Orden de cláusulas en una consulta completa

Una consulta SQL completa sigue este orden de escritura:

```sql
SELECT   columnas o expresiones     -- 1. Qué mostrar
FROM     tabla                      -- 2. De dónde
WHERE    condición                  -- 3. Qué filas incluir
ORDER BY columna [ASC|DESC]         -- 4. En qué orden
LIMIT    n;                         -- 5. Cuántas filas máximo
```

### Ejemplo integrador — Sección 1, 2 y 3

**Objetivo:** obtener los 5 clientes más recientes de Santiago o Valparaíso, cuyo email esté registrado y cuya edad sea entre 25 y 45 años.

```sql
SELECT
    id                              AS id_cliente,
    nombre                         AS nombre_completo,
    ciudad,
    email,
    edad,
    fecha_registro                 AS se_registró_el
FROM clientes
WHERE ciudad          IN ('Santiago', 'Valparaíso')
  AND edad            BETWEEN 25 AND 45
  AND email           IS NOT NULL
ORDER BY fecha_registro DESC
LIMIT 5;
```

---

# Sección 4: Relacionando Tablas — Los JOINs

## El problema de la vida real: datos normalizados

Las bases de datos relacionales almacenan la información distribuida en múltiples tablas para evitar redundancia. Esta práctica se llama **normalización**.

**Tabla `clientes`:**

| id | nombre | ciudad |
|---|---|---|
| 1 | María González | Santiago |
| 2 | Carlos Pérez | Valparaíso |
| 3 | Ana Torres | Concepción |

**Tabla `pedidos`:**

| id | cliente_id | total | estado |
|---|---|---|---|
| 101 | 1 | 150.00 | pagado |
| 102 | 2 | 89.50 | pagado |
| 103 | 1 | 234.00 | pendiente |
| 104 | 99 | 450.00 | pagado |

**Pregunta:** ¿Cómo obtengo el nombre del cliente junto al total de su pedido si están en tablas distintas?

La respuesta: **JOINs**.

---

## ¿Qué son los JOINs?

Un JOIN combina filas de dos o más tablas basándose en una **condición de relación** entre ellas, generalmente una clave foránea (`cliente_id`) que referencia la clave primaria (`id`) de otra tabla.

```
  tabla clientes          tabla pedidos
  ┌────┬─────────┐       ┌─────┬────────────┬────────┐
  │ id │ nombre  │       │ id  │ cliente_id │ total  │
  ├────┼─────────┤       ├─────┼────────────┼────────┤
  │  1 │ María   │◄──────│ 101 │     1      │ 150.00 │
  │  2 │ Carlos  │◄──────│ 102 │     2      │  89.50 │
  │  1 │ María   │◄──────│ 103 │     1      │ 234.00 │
  │  3 │ Ana     │       │ 104 │    99      │ 450.00 │ ← sin match
  └────┴─────────┘       └─────┴────────────┴────────┘
     ▲                              ▲
    clave primaria                clave foránea
```

**Sintaxis general:**

```sql
SELECT columnas
FROM tabla_izquierda
[INNER | LEFT | RIGHT | FULL OUTER] JOIN tabla_derecha
    ON tabla_izquierda.columna_comun = tabla_derecha.columna_comun;
```

---

## `INNER JOIN` — El match perfecto

Devuelve **solo las filas que tienen coincidencia en ambas tablas**. Si un registro de la tabla izquierda no tiene correspondencia en la derecha (o viceversa), se excluye del resultado.

```
  clientes          pedidos
  ┌──────┐         ┌──────┐
  │  1   │─────────│  1   │  ✅ incluido
  │  2   │─────────│  2   │  ✅ incluido
  │  3   │         │      │  ❌ excluido (Ana no tiene pedidos)
  │      │         │  99  │  ❌ excluido (cliente_id=99 no existe)
  └──────┘         └──────┘
         solo la intersección
```

```sql
-- Obtener nombre del cliente junto a sus pedidos
SELECT
    c.nombre            AS cliente,
    p.id                AS id_pedido,
    p.total,
    p.estado
FROM clientes c
INNER JOIN pedidos p
    ON c.id = p.cliente_id;
```

**Resultado:**

| cliente | id_pedido | total | estado |
|---|---|---|---|
| María González | 101 | 150.00 | pagado |
| Carlos Pérez | 102 | 89.50 | pagado |
| María González | 103 | 234.00 | pendiente |

> Nota: Ana Torres no aparece (no tiene pedidos). El pedido 104 tampoco (cliente_id=99 no existe en clientes).

```sql
-- Caso Data Engineering: unir tabla de hechos con dimensiones
-- (esquema estrella típico de un Data Warehouse)
SELECT
    v.fecha                             AS fecha_venta,
    p.nombre                            AS producto,
    c.nombre                            AS cliente,
    r.nombre_region                     AS region,
    v.cantidad,
    v.monto_total
FROM ventas v
INNER JOIN productos   p ON v.producto_id   = p.id
INNER JOIN clientes    c ON v.cliente_id    = c.id
INNER JOIN regiones    r ON v.region_id     = r.id
WHERE v.fecha BETWEEN '2026-01-01' AND '2026-03-31'
ORDER BY v.fecha DESC;
```

---

## `LEFT JOIN` — El inclusivo

Devuelve **todas las filas de la tabla izquierda**, más las coincidencias de la tabla derecha cuando existen. Donde no hay coincidencia, los campos de la tabla derecha aparecen como `NULL`.

```
  clientes (izquierda)    pedidos (derecha)
  ┌──────┐               ┌──────┐
  │  1   │───────────────│  1   │  ✅ incluido con datos
  │  2   │───────────────│  2   │  ✅ incluido con datos
  │  3   │               │ NULL │  ✅ incluido, con NULLs en columnas de pedidos
  └──────┘               └──────┘
  todos los de           solo los que
  la izquierda           coinciden
```

```sql
-- Todos los clientes, con o sin pedidos
SELECT
    c.nombre            AS cliente,
    c.ciudad,
    p.id                AS id_pedido,
    p.total,
    p.estado
FROM clientes c
LEFT JOIN pedidos p
    ON c.id = p.cliente_id;
```

**Resultado:**

| cliente | ciudad | id_pedido | total | estado |
|---|---|---|---|---|
| María González | Santiago | 101 | 150.00 | pagado |
| María González | Santiago | 103 | 234.00 | pendiente |
| Carlos Pérez | Valparaíso | 102 | 89.50 | pagado |
| Ana Torres | Concepción | NULL | NULL | NULL |

### El truco del LEFT JOIN: encontrar "huérfanos"

Combinando `LEFT JOIN` con `WHERE columna_derecha IS NULL`, podemos encontrar registros que **no tienen correspondencia**, lo cual es muy valioso en calidad de datos.

```sql
-- Clientes que NUNCA han hecho un pedido
SELECT
    c.id,
    c.nombre,
    c.email
FROM clientes c
LEFT JOIN pedidos p
    ON c.id = p.cliente_id
WHERE p.id IS NULL;   -- ← la magia: solo los que no tuvieron match

-- Caso Data Engineering: detectar registros huérfanos tras una migración
-- Productos en la tabla de ventas que no existen en el catálogo de productos
SELECT
    v.producto_id,
    COUNT(*) AS veces_vendido
FROM ventas v
LEFT JOIN productos p
    ON v.producto_id = p.id
WHERE p.id IS NULL
GROUP BY v.producto_id;
```

---

## `RIGHT JOIN` y `FULL OUTER JOIN`

### `RIGHT JOIN`

Espejo del `LEFT JOIN`. Devuelve todas las filas de la tabla **derecha** y las coincidencias de la izquierda. En la práctica, se usa poco porque siempre es posible reescribirlo como un `LEFT JOIN` intercambiando el orden de las tablas.

```sql
-- RIGHT JOIN
SELECT c.nombre, p.id, p.total
FROM clientes c
RIGHT JOIN pedidos p ON c.id = p.cliente_id;

-- Equivalente con LEFT JOIN (más común y legible)
SELECT c.nombre, p.id, p.total
FROM pedidos p
LEFT JOIN clientes c ON p.cliente_id = c.id;
```

### `FULL OUTER JOIN`

Devuelve **todas las filas de ambas tablas**, con `NULL` donde no hay coincidencia en el lado opuesto. Es la unión completa.

```sql
-- Todos los clientes Y todos los pedidos, coincidan o no
SELECT
    c.nombre            AS cliente,
    p.id                AS id_pedido,
    p.total
FROM clientes c
FULL OUTER JOIN pedidos p
    ON c.id = p.cliente_id;
```

**Resultado:**

| cliente | id_pedido | total |
|---|---|---|
| María González | 101 | 150.00 |
| María González | 103 | 234.00 |
| Carlos Pérez | 102 | 89.50 |
| Ana Torres | NULL | NULL |
| NULL | 104 | 450.00 |

> **Caso de uso:** `FULL OUTER JOIN` es útil en reconciliación de datos, por ejemplo comparar dos fuentes de datos y detectar qué registros existen en una pero no en la otra.

---

## Mapa visual de los JOINs

```
       INNER JOIN              LEFT JOIN
    ┌────┬────────┐          ┌────┬────────┐
    │  A │ A ∩ B  │          │████│ A ∩ B  │
    │    └────────┤          │████└────────┤
    │             │          │████         │
    └─────────────┘          └─────────────┘
    solo intersección        todo A + intersección


      RIGHT JOIN            FULL OUTER JOIN
    ┌────┬────────┐          ┌────┬────────┐
    │    │ A ∩ B  │          │████│ A ∩ B  │
    │    └────────┤          │████└────────┤
    │         ████│          │         ████│
    └─────────────┘          └─────────────┘
    intersección + todo B    todo A + intersección + todo B


    LEFT JOIN (solo huérfanos)    RIGHT JOIN (solo huérfanos)
    ┌────┬────────┐                ┌────┬────────┐
    │████│        │                │    │        │
    │████└────────┤                │    └────────┤
    │████         │                │         ████│
    └─────────────┘                └─────────────┘
    A WHERE B IS NULL              B WHERE A IS NULL
```

---

## Buenas prácticas al escribir JOINs

### 1. Usa siempre alias para las tablas

```sql
-- ❌ Sin alias: verboso y propenso a ambigüedades
SELECT clientes.nombre, pedidos.total
FROM clientes
INNER JOIN pedidos ON clientes.id = pedidos.cliente_id;

-- ✅ Con alias: limpio y estándar
SELECT c.nombre, p.total
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id;
```

### 2. Califica siempre las columnas ambiguas

```sql
-- ❌ ERROR: columna "id" existe en ambas tablas
SELECT id, nombre, total
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id;

-- ✅ CORRECTO: especifica de qué tabla viene cada columna
SELECT c.id AS id_cliente, c.nombre, p.id AS id_pedido, p.total
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id;
```

### 3. Elige el JOIN correcto según el caso

```sql
-- Quiero solo registros que existen en AMBAS tablas → INNER JOIN
-- Quiero TODOS de la tabla principal + los relacionados que existen → LEFT JOIN
-- Quiero detectar registros sin correspondencia → LEFT JOIN + WHERE derecha IS NULL
-- Quiero la unión total de dos tablas → FULL OUTER JOIN
```

### 4. Construye la consulta incrementalmente

```sql
-- Paso 1: Escribe el SELECT básico y verifica las columnas
SELECT c.nombre, p.total
FROM clientes c;

-- Paso 2: Agrega el JOIN
SELECT c.nombre, p.total
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id;

-- Paso 3: Agrega los filtros WHERE
SELECT c.nombre, p.total
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id
WHERE p.estado = 'pagado';

-- Paso 4: Agrega ordenamiento y límite
SELECT c.nombre, p.total
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id
WHERE p.estado = 'pagado'
ORDER BY p.total DESC
LIMIT 10;
```

---

## Consulta integradora: todos los conceptos juntos

**Objetivo:** Obtener el top 5 de clientes con mayor gasto total en pedidos pagados durante el primer trimestre de 2026, que sean de Santiago o Valparaíso, mostrando nombre, ciudad y total gastado.

```sql
SELECT
    c.nombre                        AS cliente,
    c.ciudad,
    SUM(p.total)                    AS gasto_total
FROM clientes c
INNER JOIN pedidos p
    ON c.id = p.cliente_id
WHERE p.estado       =  'pagado'
  AND p.fecha        BETWEEN '2026-01-01' AND '2026-03-31'
  AND c.ciudad       IN ('Santiago', 'Valparaíso')
  AND c.email        IS NOT NULL
GROUP BY c.id, c.nombre, c.ciudad
ORDER BY gasto_total DESC
LIMIT 5;
```

> **Nota:** `SUM()` y `GROUP BY` son funciones de agregación. Por ahora, entiende que `SUM(p.total)` suma todos los montos de pedidos por cliente, y `GROUP BY` agrupa los resultados por cliente.

---

## Resumen de cláusulas y su función

```sql
SELECT   c.nombre, p.total          -- Columnas a mostrar (con alias opcionales)
FROM     clientes c                 -- Tabla principal
JOIN     pedidos p ON c.id = p.cliente_id  -- Unión con otra tabla
WHERE    p.estado = 'pagado'        -- Filtro de filas
  AND    p.total BETWEEN 100 AND 500
  AND    c.ciudad IN ('Santiago', 'Valparaíso')
  AND    c.email IS NOT NULL
  AND    c.nombre ILIKE 'mar%'
ORDER BY p.total DESC               -- Ordenamiento
LIMIT    10;                        -- Máximo de filas a retornar
```

---

## Ejercicios propuestos

### Ejercicio 1 — Consultas básicas y filtros
Usando la tabla `productos`, escribe consultas que respondan:
1. ¿Cuáles son todos los productos con precio mayor a $500 y stock mayor a 0?
2. ¿Cuáles son los 10 productos más caros que NO son de la categoría `'Accesorios'`?
3. ¿Cuántos productos tienen precio `NULL`?

<details>
<summary>Ver solución</summary>

```sql
-- 1. Productos con precio > 500 y stock > 0
SELECT nombre, precio, stock
FROM productos
WHERE precio > 500
  AND stock > 0
ORDER BY precio DESC;

-- 2. Top 10 más caros, excluyendo Accesorios
SELECT nombre, categoria, precio
FROM productos
WHERE categoria != 'Accesorios'
  AND precio IS NOT NULL
ORDER BY precio DESC
LIMIT 10;

-- 3. Productos con precio NULL
SELECT COUNT(*) AS sin_precio
FROM productos
WHERE precio IS NULL;
```
</details>

---

### Ejercicio 2 — Filtros avanzados
Usando la tabla `ventas` con columnas `fecha`, `region`, `producto`, `monto`, responde:
1. ¿Cuáles son las ventas entre $1.000 y $50.000 realizadas en `'Norte'` o `'Sur'`?
2. ¿Cuáles son las ventas donde el nombre del producto contiene la palabra `'Pro'` (sin importar mayúsculas)?
3. ¿Hay registros con `region` vacía (`NULL`)? ¿Cuántos?

<details>
<summary>Ver solución</summary>

```sql
-- 1. Ventas en rango de monto y regiones específicas
SELECT fecha, region, producto, monto
FROM ventas
WHERE monto BETWEEN 1000 AND 50000
  AND region IN ('Norte', 'Sur')
ORDER BY monto DESC;

-- 2. Productos que contienen 'Pro' (case-insensitive)
SELECT fecha, producto, monto
FROM ventas
WHERE producto ILIKE '%Pro%';

-- 3. Registros con region NULL
SELECT COUNT(*) AS ventas_sin_region
FROM ventas
WHERE region IS NULL;
```
</details>

---

### Ejercicio 3 — JOINs
Usando las tablas `clientes (id, nombre, ciudad, email)` y `pedidos (id, cliente_id, fecha, monto, estado)`:
1. Lista todos los pedidos con el nombre del cliente. Solo incluye pedidos que tengan cliente registrado.
2. Muestra todos los clientes, incluyendo los que no tienen pedidos. Para los que no tienen pedidos, muestra `'Sin pedidos'` en la columna de estado.
3. Encuentra los clientes que **nunca** han realizado un pedido.

<details>
<summary>Ver solución</summary>

```sql
-- 1. Pedidos con nombre de cliente (INNER JOIN)
SELECT
    c.nombre        AS cliente,
    p.id            AS id_pedido,
    p.fecha,
    p.monto,
    p.estado
FROM pedidos p
INNER JOIN clientes c ON p.cliente_id = c.id
ORDER BY p.fecha DESC;

-- 2. Todos los clientes con sus pedidos (LEFT JOIN + COALESCE)
SELECT
    c.nombre,
    c.ciudad,
    p.id                                    AS id_pedido,
    p.monto,
    COALESCE(p.estado, 'Sin pedidos')       AS estado
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
ORDER BY c.nombre;

-- 3. Clientes sin ningún pedido
SELECT
    c.id,
    c.nombre,
    c.email
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
WHERE p.id IS NULL
ORDER BY c.nombre;
```
</details>


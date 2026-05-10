# Clase 2 — Bases de Datos Relacionales y Transformación desde Modelo Conceptual

---

## Tabla de contenidos

1. [Modelo relacional: terminología y características](#bloque-1)
2. [Diseño top-down: del modelo conceptual al SQL](#bloque-2)
3. [Transformación de entidades fuertes y débiles](#bloque-3)
4. [Transformación de asociaciones 1:1, 1:N y M:N](#bloque-4)
5. [Asociaciones n-arias](#bloque-5)
6. [Transformación de herencia](#bloque-6)
7. [Desnormalización](#bloque-7)
8. [Diccionario de datos y reglas dinámicas](#bloque-8)
9. [SQL: consultas y agregaciones](#bloque-9)
10. [Índices y rendimiento](#bloque-10)

---

## Bloque 1 — Modelo relacional: terminología y características {#bloque-1}

El **modelo relacional** representa los datos como **relaciones** (tablas).

### Características principales

| Característica | Explicación |
|---|---|
| Independencia física | La forma de consultar no depende de cómo están almacenados físicamente. |
| Claves lógicas | Entidades identificadas mediante claves (primarias o candidatas). |
| Normalización | Técnica para reducir redundancia y evitar anomalías. |
| SQL | Lenguaje declarativo de alto nivel para consultar y manipular. |

En la práctica, esto significa que un usuario puede consultar una tabla con SQL sin saber si internamente el motor usa páginas en disco, índices B-tree, *hash indexes*, particiones o *buffers* en memoria.

### Terminología

En el modelo relacional, los términos cambian respecto al lenguaje tradicional de archivos:

| Concepto tradicional | Concepto relacional | Ejemplo |
|---|---|---|
| Archivo | Relación o tabla | `clientes` |
| Registro | Fila o tupla | Un cliente específico |
| Campo | Columna o atributo | `rut`, `nombre`, `email` |
| Conjunto de valores válidos | Dominio | `sexo IN ('M', 'F', 'X')`, fecha válida, entero positivo |

Una tabla es una estructura bidimensional:

```text
clientes
+------------+----------------+---------------------+
| rut        | nombre         | email               |
+------------+----------------+---------------------+
| 12345678-9 | Ana Castillo   | ana@example.com     |
| 11222333-4 | Juan Pérez     | juan@example.com    |
+------------+----------------+---------------------+
```

Reglas:

- Cada fila representa una ocurrencia de una entidad.
- Cada columna representa un atributo.
- Cada columna debe tener valores del mismo dominio.
- El orden de las filas no tiene significado lógico.
- Cada fila debe poder identificarse de forma única.

### Importancia para ingeniería de datos

Cuando construyes pipelines, las claves primarias permiten:

- detectar registros nuevos;
- actualizar datos existentes;
- hacer cargas incrementales;
- evitar duplicados;
- definir relaciones confiables entre tablas;
- crear modelos dimensionales consistentes.

Patrón típico para detectar registros nuevos en un pipeline:

```sql
-- Buscar registros nuevos en staging que aún no existen en la tabla final
SELECT s.*
FROM staging_clientes s
LEFT JOIN clientes c ON s.rut = c.rut
WHERE c.rut IS NULL;
```

> 💡 **El `LEFT JOIN ... WHERE NULL`** es una idiom clásico de ingeniería de datos para *anti-joins*. Es más legible y a menudo más rápido que `NOT EXISTS` o `NOT IN`.

---

## Bloque 2 — Diseño top-down: del modelo conceptual al SQL {#bloque-2}

El **diseño top-down** parte de un modelo conceptual y lo transforma progresivamente hasta llegar a una implementación relacional concreta:

```text
Requisitos del negocio
        ↓
Modelo conceptual           ← independiente del motor
        ↓
Elección del software
        ↓
Modelo lógico relacional    ← tablas, claves, FKs
        ↓
Modelo físico               ← índices, particiones, formatos
        ↓
Implementación SQL
```

### Elección del software

| Criterio | Preguntas |
|---|---|
| Tipo de carga | OLTP, OLAP, *batch*, *streaming*? |
| Volumen | GB, TB, PB? |
| Latencia | ms, s, min? |
| Escalabilidad | vertical, horizontal, distribuida? |
| Consistencia | ¿ACID estricto? |
| Equipo | ¿qué motor conoce? |
| Ecosistema | ¿integra con Airflow, dbt, Spark, Kafka, BI? |

```text
Sistema transaccional bancario  → motor relacional con ACID fuerte
Catálogo flexible de productos  → documental o relacional con JSON
Analítica histórica masiva      → data warehouse columnar (BigQuery, Snowflake)
```

Esta clase se enfoca en el paso **modelo conceptual → modelo relacional**.

---

## Bloque 3 — Transformación de entidades fuertes y débiles {#bloque-3}

### Entidad fuerte

Tiene existencia propia y posee identificador propio.

**Regla:** crear una tabla con sus atributos simples y elegir una clave primaria.

```text
Cliente
- rut
- nombre
- telefono
```

```sql
CREATE TABLE cliente (
    rut VARCHAR(12) PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    telefono VARCHAR(20)
);
```

Ejemplo con clave compuesta (`precio_producto` por producto y fecha):

```sql
CREATE TABLE precio_producto (
    id_producto INT,
    fecha_inicio DATE,
    precio NUMERIC(12,2) NOT NULL,
    PRIMARY KEY (id_producto, fecha_inicio)
);
```

### Entidad débil

No puede identificarse completamente sin otra entidad de la cual depende.

> *Una `linea_factura` depende de una `factura`. La línea 1 no es globalmente única, pero sí lo es dentro de una factura.*

**Regla:**

1. Crear una tabla para la entidad débil con sus atributos.
2. Incluir la clave primaria de la fuerte como FK.
3. La PK de la débil será la combinación: PK de la fuerte + identificador parcial.

```text
Factura
- id_factura
- fecha

LineaFactura  (depende de Factura)
- numero_linea
- cantidad
- precio_unitario
```

```sql
CREATE TABLE factura (
    id_factura INT PRIMARY KEY,
    fecha DATE NOT NULL
);

CREATE TABLE linea_factura (
    id_factura INT,
    numero_linea INT,
    cantidad INT NOT NULL,
    precio_unitario NUMERIC(12,2) NOT NULL,
    PRIMARY KEY (id_factura, numero_linea),
    FOREIGN KEY (id_factura) REFERENCES factura(id_factura)
);
```

> 💡 **Patrón en pipelines:** las entidades débiles aparecen como detalles — *items de pedido*, *eventos dentro de sesión*, *mediciones por dispositivo*. La clave compuesta evita confundir líneas de distintas entidades padre.

---

## Bloque 4 — Transformación de asociaciones 1:1, 1:N y M:N {#bloque-4}

### Asociación 1:1

Una fila de una tabla se relaciona con máximo una fila de otra (y viceversa).

```text
Persona 1 ──── 1 Pasaporte
```

**Regla:** elegir una de las dos tablas e incluir la PK de la otra como FK con restricción `UNIQUE`.

```sql
CREATE TABLE persona (
    id_persona INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE pasaporte (
    id_pasaporte INT PRIMARY KEY,
    numero VARCHAR(30) UNIQUE NOT NULL,
    id_persona INT UNIQUE NOT NULL,
    FOREIGN KEY (id_persona) REFERENCES persona(id_persona)
);
```

El `UNIQUE` sobre `id_persona` asegura que una persona no tenga más de un pasaporte.

**¿En qué lado poner la FK?**

- Si una entidad depende de la otra, colocar la FK en la dependiente.
- Si una participación es opcional y otra obligatoria, poner la FK en el lado opcional para evitar muchos `NULL`.
- Si la asociación tiene atributos relevantes, considerar una tabla propia.

### Asociación 1:N

Una fila puede relacionarse con muchas, pero cada fila del lado N se relaciona con una sola del lado 1.

```text
Autor 1 ──── N Libro
```

**Regla:** poner la FK en el lado N.

```sql
CREATE TABLE autor (
    id_autor INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE libro (
    id_libro INT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    id_autor INT NOT NULL,
    FOREIGN KEY (id_autor) REFERENCES autor(id_autor)
);
```

Patrón frecuente en sistemas transaccionales:

```text
cliente    1 ──── N pedido
pedido     1 ──── N detalle_pedido
categoria  1 ──── N producto
pais       1 ──── N ciudad
```

### Asociación M:N

Muchas filas se relacionan con muchas:

```text
Alumno M ──── N Curso
```

**Regla:** crear una tabla intermedia.

```sql
CREATE TABLE alumno (
    id_alumno INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE curso (
    id_curso INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE inscripcion (
    id_alumno INT,
    id_curso INT,
    fecha_inscripcion DATE NOT NULL,
    nota_final NUMERIC(4,2),
    PRIMARY KEY (id_alumno, id_curso),
    FOREIGN KEY (id_alumno) REFERENCES alumno(id_alumno),
    FOREIGN KEY (id_curso) REFERENCES curso(id_curso)
);
```

> 💡 **Las tablas intermedias suelen ser tablas de hechos.** En analítica, capturan eventos: usuario compra producto, alumno se inscribe en curso, cliente usa cupón. Modelar bien la M:N es modelar bien el negocio.

---

## Bloque 5 — Asociaciones n-arias {#bloque-5}

Una asociación **n-aria** involucra tres o más entidades:

```text
Producto - Bodega - Pedido
```

Significa: *en cierto pedido se solicita cierto producto desde cierta bodega*.

**Regla:** crear una tabla con FKs a todas las entidades participantes.

```sql
CREATE TABLE producto (
    id_producto INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE bodega (
    id_bodega INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE pedido (
    id_pedido INT PRIMARY KEY,
    fecha DATE NOT NULL
);

CREATE TABLE pedido_producto_bodega (
    id_pedido INT,
    id_producto INT,
    id_bodega INT,
    cantidad INT NOT NULL,
    PRIMARY KEY (id_pedido, id_producto, id_bodega),
    FOREIGN KEY (id_pedido) REFERENCES pedido(id_pedido),
    FOREIGN KEY (id_producto) REFERENCES producto(id_producto),
    FOREIGN KEY (id_bodega) REFERENCES bodega(id_bodega)
);
```

> ⚠️ **No toda relación ternaria se puede reemplazar por tres binarias.** La relación `Pedido - Producto - Bodega` no significa lo mismo que `Pedido - Producto`, `Pedido - Bodega` y `Producto - Bodega` por separado. Las binarias **pierden la conexión conjunta**.

---

## Bloque 6 — Transformación de herencia {#bloque-6}

```text
Persona
├── Estudiante
└── Profesor
```

Hay varias estrategias de transformación, cada una con *trade-offs*.

### Alternativa 1: Superclase y subclases sobreviven

Una tabla por cada entidad. La PK de cada subclase es también FK hacia la superclase.

```sql
CREATE TABLE persona (
    id_persona INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE
);

CREATE TABLE estudiante (
    id_persona INT PRIMARY KEY,
    carrera VARCHAR(100),
    promedio NUMERIC(4,2),
    FOREIGN KEY (id_persona) REFERENCES persona(id_persona)
);

CREATE TABLE profesor (
    id_persona INT PRIMARY KEY,
    departamento VARCHAR(100),
    sueldo NUMERIC(12,2),
    FOREIGN KEY (id_persona) REFERENCES persona(id_persona)
);
```

| Ventajas | Desventajas |
|---|---|
| Modelo normalizado | Requiere `JOIN` para datos completos |
| Sin duplicación de atributos comunes | Más costoso en consultas masivas |
| Buena para jerarquías complejas | |

### Alternativa 2: Las subclases absorben a la superclase

Una tabla por cada subclase, replicando los atributos comunes:

```sql
CREATE TABLE estudiante (
    id_persona INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE,
    carrera VARCHAR(100),
    promedio NUMERIC(4,2)
);

CREATE TABLE profesor (
    id_persona INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE,
    departamento VARCHAR(100),
    sueldo NUMERIC(12,2)
);
```

| Ventajas | Desventajas |
|---|---|
| Sin `JOIN` para datos completos | Atributos comunes duplicados |
| Consultas simples por subtipo | Consultar todas las personas requiere `UNION` |
| | Problemática si una entidad puede ser de varios subtipos |

### Alternativa 3.1: Superclase absorbe con tipo exclusivo

Una tabla con discriminador:

```sql
CREATE TABLE persona (
    id_persona INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE,
    tipo_persona VARCHAR(20) NOT NULL,
    carrera VARCHAR(100),
    promedio NUMERIC(4,2),
    departamento VARCHAR(100),
    sueldo NUMERIC(12,2),
    CHECK (tipo_persona IN ('ESTUDIANTE', 'PROFESOR'))
);
```

| Ventajas | Desventajas |
|---|---|
| Una sola tabla, sin `JOIN` | Muchas columnas pueden quedar `NULL` |
| Consultas simples | Menor claridad semántica |
| | Restricciones adicionales para combinaciones inválidas |

### Alternativa 3.2: Tipos no exclusivos (booleanos)

```sql
CREATE TABLE persona (
    id_persona INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    es_estudiante BOOLEAN NOT NULL DEFAULT FALSE,
    es_profesor BOOLEAN NOT NULL DEFAULT FALSE,
    -- atributos comunes y específicos
);
```

Permite que una persona sea estudiante **y** profesor al mismo tiempo. Útil para roles múltiples no excluyentes.

> 💡 **Para sistemas con muchos roles** suele ser mejor extraerlos a una tabla de roles:
>
> ```sql
> CREATE TABLE usuario_rol (
>     id_usuario INT,
>     id_rol INT,
>     PRIMARY KEY (id_usuario, id_rol)
> );
> ```
>
> Es más escalable: agregar nuevos roles no requiere modificar el esquema.

---

## Bloque 7 — Desnormalización {#bloque-7}

La **desnormalización** consiste en agregar redundancia controlada para mejorar rendimiento.

### Ejemplo: total de factura

Modelo normalizado:

```text
factura(id_factura, fecha, rut_cliente)
detalle_factura(id_factura, id_producto, cantidad, subtotal)
```

El total se calcula:

```sql
SELECT id_factura, SUM(subtotal) AS total
FROM detalle_factura
GROUP BY id_factura;
```

Si esta consulta se ejecuta constantemente, se puede almacenar el total directamente:

```sql
CREATE TABLE factura (
    id_factura INT PRIMARY KEY,
    fecha DATE NOT NULL,
    rut_cliente VARCHAR(12) NOT NULL,
    total NUMERIC(12,2) NOT NULL DEFAULT 0    -- desnormalizado
);
```

| Ventaja | Riesgo |
|---|---|
| Lectura más rápida | Posible inconsistencia: `factura.total ≠ SUM(detalle.subtotal)` |

Si se desnormaliza, se necesitan controles: *triggers*, *jobs* de reconciliación, *constraints* o pruebas de calidad.

### Desnormalización en analítica

En sistemas analíticos es **común y deseable** desnormalizar:

```text
tabla_hechos_ventas
- fecha
- id_producto
- nombre_producto         ← duplicado de dim_producto
- categoria_producto      ← duplicado de dim_producto
- id_cliente
- segmento_cliente        ← duplicado de dim_cliente
- cantidad
- monto
```

| Sistema | Preferencia |
|---|---|
| OLTP | Normalización para integridad |
| OLAP / BI | Desnormalización controlada para rendimiento |
| Data Lake | Flexibilidad, esquemas evolutivos |
| Lakehouse | Mezcla: estructura + particiones + formatos columnares |

> 💡 **La regla de oro:** si la consulta es frecuente y la inconsistencia es tolerable (o controlable), desnormaliza. Si la integridad es crítica, normaliza y vive con el costo del JOIN.

---

## Bloque 8 — Diccionario de datos y reglas dinámicas {#bloque-8}

El modelo de datos describe principalmente reglas estáticas: entidades, atributos, claves, relaciones, cardinalidades, restricciones. Pero un sistema real necesita documentar más.

### Diccionario de datos

Cada tabla y columna debe tener una descripción.

| Nombre | Tipo | Longitud | Clave | Descripción |
|---|---:|---:|---|---|
| `nro_factura` | INT | 6 | PK | Identificador de la factura |
| `fecha` | DATE | - | No | Fecha de emisión |
| `total` | INT | 6 | No | Monto bruto a cobrar |
| `rut_cliente` | CHAR | 10 | FK | Identificador del cliente |

En ingeniería de datos, conviene además documentar:

| Campo documental | Por qué importa |
|---|---|
| Fuente original | Permite rastrear linaje. |
| Frecuencia de actualización | Diseñar cargas *batch* o *streaming*. |
| Regla de nulabilidad | Validaciones de calidad. |
| Sensibilidad | Identificar PII o datos regulados. |
| Regla de negocio | Evita interpretaciones incorrectas. |
| Dueño del dato | Resolver dudas y errores. |

Versión enriquecida:

| Columna | Tipo | PK | FK | Nullable | Descripción | Regla de calidad |
|---|---|---|---|---|---|---|
| `id_cliente` | VARCHAR(12) | Sí | No | No | Identificador único | No nulo, no duplicado |
| `email` | VARCHAR(150) | No | No | Sí | Correo del cliente | Formato válido si existe |
| `fecha_alta` | DATE | No | No | No | Fecha de creación | No futura |

### Reglas dinámicas

Especificaciones que preservan integridad ante operaciones DML (`INSERT`, `UPDATE`, `DELETE`).

**Estructura recomendada:**

| Elemento | Descripción |
|---|---|
| Regla de usuario | Declaración clara |
| Evento | Operación que activa |
| Entidad | Tabla afectada |
| Condición | Cuándo se ejecuta |
| Acción | Qué pasa |

**Ejemplo: stock bajo el mínimo**

| Elemento | Valor |
|---|---|
| Regla | Un producto no puede estar bajo su stock mínimo sin generar pedido |
| Evento | `UPDATE` |
| Entidad | `producto` |
| Condición | `stock_actual < stock_minimo` |
| Acción | Insertar registro en `pedido_compra_pendiente` |

Implementación con *trigger* en PostgreSQL:

```sql
CREATE TABLE pedido_compra_pendiente (
    id_pedido SERIAL PRIMARY KEY,
    id_producto INT NOT NULL,
    fecha_creacion TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    motivo VARCHAR(200) NOT NULL,
    FOREIGN KEY (id_producto) REFERENCES producto(id_producto)
);

CREATE OR REPLACE FUNCTION generar_pedido_stock_bajo()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.stock_actual < NEW.stock_minimo THEN
        INSERT INTO pedido_compra_pendiente (id_producto, motivo)
        VALUES (NEW.id_producto, 'Stock actual bajo el mínimo');
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_stock_bajo
AFTER UPDATE OF stock_actual ON producto
FOR EACH ROW
EXECUTE FUNCTION generar_pedido_stock_bajo();
```

> ⚠️ **Las reglas dinámicas mejoran la integridad pero ocultan lógica.** Si un pipeline no sabe que existe el *trigger*, puede sorprenderse al ver registros aparecer "mágicamente" en otra tabla. **Documenta siempre los triggers** y considera si es mejor tener la lógica en la aplicación o en el ETL.

---

## Bloque 9 — SQL: consultas y agregaciones {#bloque-9}

> 💡 **Repaso:** ya viste lo básico de SQL en la [Clase 3 del Módulo 01](../../modulo-01-fundamentos/clase-03-consultas-sql-join-postgresql/README.md). Aquí profundizamos con casos más analíticos.

### Tablas de ejemplo

```sql
CREATE TABLE autores (
    id_autor INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    nacionalidad VARCHAR(50)
);

CREATE TABLE libros (
    id_libro INT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    anio_publicacion INT,
    id_autor INT,
    FOREIGN KEY (id_autor) REFERENCES autores(id_autor)
);

CREATE INDEX idx_libros_autor ON libros(id_autor);
```

```sql
INSERT INTO autores VALUES
(1, 'Isabel Allende',          'Chilena'),
(2, 'Gabriel García Márquez',  'Colombiana'),
(3, 'Jorge Luis Borges',       'Argentina');

INSERT INTO libros VALUES
(1, 'La casa de los espíritus',         1982, 1),
(2, 'Largo pétalo de mar',              2019, 1),
(3, 'Cien años de soledad',             1967, 2),
(4, 'El amor en los tiempos del cólera',1985, 2),
(5, 'Historia universal de la infamia', 1935, 3),
(6, 'Ficciones',                        1944, 3),
(7, 'El Aleph',                         1949, 3);
```

### Consulta 1: filtros

```sql
-- Libros publicados después de 2015
SELECT titulo, anio_publicacion
FROM libros
WHERE anio_publicacion > 2015;
```

### Consulta 2: búsqueda por patrón

```sql
SELECT *
FROM libros
WHERE titulo LIKE '%historia%';
```

> ⚠️ **Una búsqueda con `%texto%` no aprovecha bien un índice B-tree**. Para texto completo en PostgreSQL: índices trigram (`pg_trgm`), *full-text search* (`tsvector`), o motores externos como Elasticsearch.

### Consulta 3: JOIN

```sql
-- Libros con sus autores
SELECT
    l.titulo,
    l.anio_publicacion,
    a.nombre AS autor
FROM libros l
JOIN autores a ON l.id_autor = a.id_autor;
```

### Consulta 4: GROUP BY + LEFT JOIN

```sql
-- Cuántos libros tiene cada autor (incluye autores sin libros)
SELECT
    a.nombre,
    COUNT(l.id_libro) AS total_libros
FROM autores a
LEFT JOIN libros l ON a.id_autor = l.id_autor
GROUP BY a.nombre;
```

> 💡 **`LEFT JOIN` vs `JOIN`:** si usaras `JOIN`, los autores sin libros desaparecerían del resultado. `LEFT JOIN` los conserva, mostrando `count = 0`.

### Consulta 5: HAVING

```sql
-- Autores con más de dos libros
SELECT
    a.nombre,
    COUNT(l.id_libro) AS total_libros
FROM autores a
JOIN libros l ON a.id_autor = l.id_autor
GROUP BY a.nombre
HAVING COUNT(l.id_libro) > 2;
```

| Cláusula | Cuándo filtra | Ejemplo |
|---|---|---|
| `WHERE` | Antes de agrupar | `WHERE anio > 2015` |
| `HAVING` | Después de agrupar | `HAVING COUNT(*) > 2` |

### Consulta 6: subconsulta vs función ventana

**Versión con subconsulta** (no maneja empates):

```sql
-- Libros del autor con más libros
SELECT titulo
FROM libros
WHERE id_autor = (
    SELECT id_autor
    FROM libros
    GROUP BY id_autor
    ORDER BY COUNT(*) DESC
    LIMIT 1
);
```

**Versión con función ventana** (maneja empates):

```sql
WITH conteo AS (
    SELECT
        id_autor,
        COUNT(*) AS total_libros,
        RANK() OVER (ORDER BY COUNT(*) DESC) AS ranking
    FROM libros
    GROUP BY id_autor
)
SELECT
    l.titulo,
    a.nombre AS autor
FROM libros l
JOIN autores a ON l.id_autor = a.id_autor
JOIN conteo c ON l.id_autor = c.id_autor
WHERE c.ranking = 1;
```

> 💡 **Las funciones ventana** (`OVER`, `PARTITION BY`, `RANK`, `ROW_NUMBER`, `LAG`, `LEAD`) son el caballo de batalla de la analítica SQL moderna. Permiten cálculos por grupos sin colapsar las filas. Aprenderlas bien acelera mucho el trabajo de un Data Engineer.

---

## Bloque 10 — Índices y rendimiento {#bloque-10}

```sql
CREATE INDEX idx_libros_autor ON libros(id_autor);
```

Este índice ayuda en consultas como:

```sql
SELECT * FROM libros WHERE id_autor = 3;

-- O JOINs que filtran por id_autor
SELECT l.titulo, a.nombre
FROM libros l
JOIN autores a ON l.id_autor = a.id_autor;
```

### ¿Cómo funciona conceptualmente?

Sin índice, el motor revisa fila por fila (*scan completo*). Con índice, accede rápidamente a las filas candidatas. **Analogía:** es como el índice de un libro — no necesitas leer todo para encontrar un tema.

### Cuándo un índice puede no ayudar

Los índices no son gratis:

- ocupan espacio;
- hacen más lentos `INSERT`, `UPDATE`, `DELETE`;
- requieren mantenimiento;
- no siempre son usados por el optimizador.

Casos donde puede no ayudar:

- `WHERE titulo LIKE '%historia%'` (patrón con wildcard al inicio).
- Tablas muy pequeñas.
- Cuando la columna tiene baja cardinalidad (ej: `sexo`).

### Verificar con `EXPLAIN`

```sql
EXPLAIN ANALYZE
SELECT l.titulo, a.nombre
FROM libros l
JOIN autores a ON l.id_autor = a.id_autor;
```

Esto muestra el plan de ejecución del motor: qué índices usa, cuántas filas estima leer, cuánto tiempo toma cada paso.

> 💡 **`EXPLAIN ANALYZE` es la herramienta más útil de un Data Engineer trabajando con SQL.** Antes de optimizar a ciegas (agregar índices, reescribir consultas), míralo. Te dirá exactamente dónde está el cuello de botella.

---

## Buenas prácticas

1. Define claves primarias claras.
2. Usa claves foráneas cuando el sistema lo permita.
3. Evita columnas ambiguas (`codigo`, `valor`, `estado` sin descripción).
4. Documenta dominios y reglas de negocio.
5. Usa nombres consistentes.
6. No mezcles distintos niveles de granularidad en una misma tabla.
7. Distingue tablas maestras, transaccionales, intermedias y de hechos.
8. No desnormalices sin justificarlo.
9. Agrega índices basados en patrones reales de consulta.
10. Versiona los scripts DDL en Git.

## Errores comunes

| Error | Consecuencia |
|---|---|
| No definir PK | Duplicados difíciles de controlar. |
| Modelar M:N sin tabla intermedia | Pérdida de semántica o datos repetidos. |
| Usar `SELECT *` en pipelines | Pipelines frágiles ante cambios de esquema. |
| No documentar reglas dinámicas | Comportamientos inesperados en cargas. |
| Desnormalizar sin control | Inconsistencias entre campos derivados y datos base. |
| Ignorar cardinalidades | *Joins* que multiplican filas incorrectamente. |
| No considerar empates en *rankings* | Resultados analíticos incompletos. |
| Usar índices sin medir | Mayor costo de escritura sin mejora real. |

---

<details>
<summary><strong>🟢 Ejercicio 1 — Transformar 1:N (click para ver)</strong></summary>

**Caso:** Una categoría puede tener muchos productos. Cada producto pertenece a una sola categoría.

**Tareas:**

1. Identifica entidades y cardinalidad.
2. Crea el modelo relacional.
3. Escribe el SQL `CREATE TABLE`.

**Solución:**

Entidades: `categoria`, `producto`. Cardinalidad: 1:N. La FK va en el lado N.

```sql
CREATE TABLE categoria (
    id_categoria INT PRIMARY KEY,
    nombre VARCHAR(80) NOT NULL UNIQUE
);

CREATE TABLE producto (
    id_producto INT PRIMARY KEY,
    nombre VARCHAR(120) NOT NULL,
    precio NUMERIC(12,2) NOT NULL CHECK (precio >= 0),
    id_categoria INT NOT NULL,
    FOREIGN KEY (id_categoria) REFERENCES categoria(id_categoria)
);
```

</details>

<details>
<summary><strong>🟢 Ejercicio 2 — Transformar M:N + consulta agregada (click para ver)</strong></summary>

**Caso:** Un estudiante puede matricular muchos ramos. Un ramo puede tener muchos estudiantes. Para cada matrícula se registra fecha y nota final.

**Tareas:**

1. Define la tabla intermedia.
2. Escribe una consulta para promedio de nota por ramo.

**Solución:**

```sql
CREATE TABLE estudiante (
    id_estudiante INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE ramo (
    id_ramo INT PRIMARY KEY,
    nombre VARCHAR(120) NOT NULL
);

CREATE TABLE matricula (
    id_estudiante INT,
    id_ramo INT,
    fecha DATE NOT NULL,
    nota_final NUMERIC(4,2),
    PRIMARY KEY (id_estudiante, id_ramo),
    FOREIGN KEY (id_estudiante) REFERENCES estudiante(id_estudiante),
    FOREIGN KEY (id_ramo) REFERENCES ramo(id_ramo)
);

-- Promedio por ramo
SELECT
    r.nombre,
    ROUND(AVG(m.nota_final)::numeric, 2) AS promedio
FROM ramo r
LEFT JOIN matricula m ON r.id_ramo = m.id_ramo
GROUP BY r.nombre
ORDER BY promedio DESC NULLS LAST;
```

</details>

<details>
<summary><strong>🟢 Ejercicio 3 — Herencia (click para ver)</strong></summary>

**Caso:** Una persona puede ser cliente o empleado. Un cliente tiene segmento comercial. Un empleado tiene cargo y salario.

**Tareas:**

1. Modela usando alternativa 1 (superclase + subclases).
2. Modela usando alternativa 3.1 (una sola tabla con tipo).
3. Compara ventajas.

**Solución alternativa 1:**

```sql
CREATE TABLE persona (
    id_persona INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE cliente (
    id_persona INT PRIMARY KEY,
    segmento VARCHAR(50),
    FOREIGN KEY (id_persona) REFERENCES persona(id_persona)
);

CREATE TABLE empleado (
    id_persona INT PRIMARY KEY,
    cargo VARCHAR(80),
    salario NUMERIC(12,2),
    FOREIGN KEY (id_persona) REFERENCES persona(id_persona)
);
```

**Solución alternativa 3.1:**

```sql
CREATE TABLE persona (
    id_persona INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    tipo VARCHAR(20) NOT NULL CHECK (tipo IN ('CLIENTE', 'EMPLEADO')),
    segmento VARCHAR(50),
    cargo VARCHAR(80),
    salario NUMERIC(12,2)
);
```

| Alternativa | Ventaja | Desventaja |
|---|---|---|
| 1 (3 tablas) | Sin NULLs innecesarios, normalizada | Requiere JOIN |
| 3.1 (1 tabla) | Consultas simples, sin JOIN | NULLs en columnas no aplicables |

Si la jerarquía cambia poco y los subtipos son exclusivos, 3.1 es práctica. Si los subtipos pueden cambiar o coexistir, mejor 1.

</details>

<details>
<summary><strong>🟢 Ejercicio 4 — SQL aplicado (click para ver)</strong></summary>

Usando las tablas `autores` y `libros`, escribe consultas para:

1. Listar autores chilenos.
2. Contar libros publicados por año.
3. Mostrar autores sin libros.
4. Obtener el libro más reciente por autor.

**Solución:**

```sql
-- 1. Autores chilenos
SELECT * FROM autores WHERE nacionalidad = 'Chilena';

-- 2. Libros por año
SELECT anio_publicacion, COUNT(*) AS total
FROM libros
GROUP BY anio_publicacion
ORDER BY anio_publicacion;

-- 3. Autores sin libros
SELECT a.*
FROM autores a
LEFT JOIN libros l ON a.id_autor = l.id_autor
WHERE l.id_libro IS NULL;

-- 4. Libro más reciente por autor (con función ventana)
WITH ranking AS (
    SELECT
        l.*,
        ROW_NUMBER() OVER (PARTITION BY id_autor ORDER BY anio_publicacion DESC) AS rn
    FROM libros l
)
SELECT id_autor, titulo, anio_publicacion
FROM ranking
WHERE rn = 1;
```

</details>

---

## Referencia rápida — Modelo relacional

```
TERMINOLOGÍA
─────────────────────────────────────────────────────────────────
  Tabla / relación        archivo
  Fila / tupla            registro
  Columna / atributo      campo
  Dominio                 valores válidos

TRANSFORMACIÓN CONCEPTUAL → RELACIONAL
─────────────────────────────────────────────────────────────────
  Entidad fuerte      → tabla con PK propia
  Entidad débil       → tabla con PK = (PK fuerte, ID parcial)
  1:1                 → FK + UNIQUE
  1:N                 → FK en el lado N
  M:N                 → tabla intermedia con PK compuesta
  N-aria              → tabla intermedia con FK a todas las entidades
  Herencia            → varias estrategias (ver bloque 6)

INTEGRIDAD
─────────────────────────────────────────────────────────────────
  PRIMARY KEY         único e inmutable
  FOREIGN KEY         relación referencial
  UNIQUE              alternativa
  NOT NULL            obligatoriedad
  CHECK               restricción de dominio

CONSULTAS CLAVE
─────────────────────────────────────────────────────────────────
  WHERE          filtro previo a agrupar
  GROUP BY       agrupación
  HAVING         filtro post-agrupación
  JOIN           combina tablas
  LEFT JOIN      conserva todas las filas del lado izquierdo
  WINDOW         OVER, PARTITION BY, ORDER BY

DESNORMALIZACIÓN
─────────────────────────────────────────────────────────────────
  OLTP    → preferir normalización
  OLAP    → desnormalización controlada (estrellas)
  Riesgo  → inconsistencia entre datos derivados y base

DICCIONARIO DE DATOS (campos críticos)
─────────────────────────────────────────────────────────────────
  Nombre, tipo, PK/FK, nullable, descripción
  Fuente, frecuencia, sensibilidad, dueño, regla de calidad

ÍNDICES
─────────────────────────────────────────────────────────────────
  CREATE INDEX  acelera filtros y JOINs frecuentes
  Costos        espacio + escrituras más lentas
  Verificar     EXPLAIN ANALYZE
```

---

*→ Próxima clase: [NoSQL y Cassandra](../clase-03-nosql-y-cassandra/README.md)*

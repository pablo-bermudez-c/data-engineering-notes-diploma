# Clase 5 — Bases de Grafos y Neo4j

---

## Tabla de contenidos

1. [Motivación: por qué grafos](#bloque-1)
2. [Conceptos: nodos, relaciones y propiedades](#bloque-2)
3. [Cuándo conviene una base de grafos](#bloque-3)
4. [RDF vs grafo de propiedades](#bloque-4)
5. [Neo4j y Cypher](#bloque-5)
6. [Consultas básicas en Cypher](#bloque-6)
7. [Recorridos y consultas multi-nivel](#bloque-7)
8. [Diseño de un grafo](#bloque-8)
9. [Carga de datos desde DE](#bloque-9)
10. [Casos prácticos: linaje y red social](#bloque-10)

---

## Bloque 1 — Motivación: por qué grafos {#bloque-1}

> 💡 **Conexión con la Clase 5 del Módulo 03:** ya viste qué es un grafo a nivel de estructura de datos (vértices, aristas, DFS, BFS, Dijkstra). Aquí aplicamos esos conceptos al mundo de las **bases de datos**: cómo persistir grafos de forma duradera, consultarlos eficientemente y escalarlos.

Algunas preguntas son **naturalmente relacionales**:

- ¿Cómo encuentran las redes sociales contactos de segundo nivel?
- ¿Cómo recomienda Netflix contenido a partir de gustos similares?
- ¿Cómo detectar redes de fraude financiero?
- ¿Cómo encontrar dependencias entre servicios, tablas, *pipelines* y *dashboards*?
- ¿Cómo saber qué reportes se ven afectados si cambia una columna?

En estos casos, el interés principal **no está solo en guardar las entidades**, sino en analizar las **conexiones** entre ellas:

```text
Usuario   → vio          → Película
Usuario   → sigue        → Usuario
Cliente   → compró       → Producto
Cuenta    → transfirió_a → Cuenta
Pipeline  → escribe      → Tabla
Dashboard → consume      → Tabla
```

En SQL, estas relaciones pueden modelarse con FKs y tablas intermedias. **El problema aparece** cuando las consultas requieren muchos saltos, patrones variables o recorridos profundos.

---

## Bloque 2 — Conceptos: nodos, relaciones y propiedades {#bloque-2}

Un **grafo** está formado por:

- **Nodos** (o vértices): representan entidades.
- **Aristas** (o relaciones): conectan nodos.
- **Propiedades**: atributos asociados a nodos o relaciones.

Ejemplo:

```text
(Juan)-[:CONOCE {desde: 2022}]->(Ana)
```

Aquí:

- `Juan` es un nodo (con etiqueta `Persona`).
- `Ana` es otro nodo.
- `CONOCE` es una relación dirigida.
- `desde: 2022` es una propiedad de la relación.

| Concepto | Descripción | Ejemplo |
|---|---|---|
| **Nodo** | Entidad del dominio | Persona, Película, Producto |
| **Relación** | Conexión entre dos nodos | `:CONOCE`, `:COMPRO`, `:DEPENDE_DE` |
| **Propiedad** | Dato asociado a nodo o relación | `nombre`, `fecha`, `peso`, `monto` |
| **Etiqueta** | Categoría de un nodo | `:Persona`, `:Movie`, `:Cliente` |
| **Tipo de relación** | Categoría de una arista | `:ACTED_IN`, `:DIRECTED` |
| **Camino** | Secuencia de nodos y relaciones | `Juan → Ana → Pedro` |
| **Recorrido** | Exploración del grafo siguiendo relaciones | "amigos de amigos" |

---

## Bloque 3 — Cuándo conviene una base de grafos {#bloque-3}

### El problema en SQL

Buscar amigos de segundo nivel en SQL requiere *autojoins*:

```sql
SELECT u3.nombre
FROM usuarios u1
JOIN amistades a1 ON u1.id_usuario = a1.id_usuario_origen
JOIN usuarios u2 ON a1.id_usuario_destino = u2.id_usuario
JOIN amistades a2 ON u2.id_usuario = a2.id_usuario_origen
JOIN usuarios u3 ON a2.id_usuario_destino = u3.id_usuario
WHERE u1.nombre = 'Juan';
```

Esto funciona, pero se vuelve difícil cuando:

- hay muchos niveles de profundidad;
- el número de saltos no es fijo;
- se buscan patrones complejos;
- el rendimiento depende de muchos *joins*;
- el dominio se entiende mejor como red.

### Misma idea en Cypher

```cypher
MATCH (juan:Persona {nombre: 'Juan'})-[:CONOCE]->(:Persona)-[:CONOCE]->(recomendado:Persona)
RETURN recomendado;
```

Más directo, más legible, y eficiente para grafos.

### Ventajas y limitaciones

**Ventajas:**

- Modelan relaciones de forma natural.
- Esquema flexible.
- Eficientes para consultas de relaciones complejas.
- Permiten recorridos profundos.
- Útiles para descubrir patrones, comunidades, caminos.

**Limitaciones:**

- No son la mejor opción para datos tabulares simples.
- La escalabilidad horizontal es más compleja que en otros NoSQL.
- No existe un único lenguaje universal equivalente a SQL.
- No siempre son óptimas para agregaciones masivas tipo OLAP.
- **No reemplazan a un data warehouse:** lo complementan.

### Casos de uso en ingeniería de datos

#### Recomendaciones

```text
(Usuario)-[:VIO]->(Película)
(Usuario)-[:CALIFICO {score: 5}]->(Película)
(Película)-[:PERTENECE_A]->(Género)
```

> *"Encontrar películas vistas por usuarios similares que el usuario actual aún no ha visto."*

#### Detección de fraude

```text
(Persona)-[:USA]->(Tarjeta)
(Persona)-[:TIENE]->(Cuenta)
(Cuenta)-[:TRANSFIERE_A {monto, fecha}]->(Cuenta)
(Persona)-[:USA_IP]->(IP)
```

> - ¿Muchas cuentas usan la misma IP?
> - ¿Una cuenta transfiere a muchas cuentas nuevas?
> - ¿Existen ciclos de transferencias sospechosas?

#### Linaje de datos

```text
(Pipeline)-[:LEE]->(Tabla)
(Pipeline)-[:ESCRIBE]->(Tabla)
(Dashboard)-[:CONSUME]->(Tabla)
(Columna)-[:DERIVA_DE]->(Columna)
```

> *"Si cambia la columna `cliente.email`, ¿qué pipelines, tablas y dashboards se verán afectados?"*

```cypher
MATCH path = (:Columna {nombre: 'cliente.email'})-[:DERIVA_DE|LEE|ESCRIBE|CONSUME*1..5]->(impactado)
RETURN path;
```

> 💡 **El linaje de datos** es uno de los casos más relevantes para Data Engineers en 2026. Herramientas como OpenLineage, DataHub, Marquez o Atlan usan grafos por debajo. Saber Cypher te permite escribir consultas de impacto sobre estos catálogos.

#### Catálogo y gobierno de datos

```text
(Dataset)-[:PERTENECE_A]->(Dominio)
(Dataset)-[:TIENE_OWNER]->(Persona)
(Dataset)-[:CONTIENE]->(Columna)
(Columna)-[:CLASIFICADA_COMO]->(DatoSensible)
```

---

## Bloque 4 — RDF vs grafo de propiedades {#bloque-4}

### RDF (Resource Description Framework)

Estándar web creado para modelar recursos. Representa conocimiento como **triples**:

```text
sujeto - predicado - objeto

Juan      - conoce_a    - Ana
Ana       - trabaja_en  - EmpresaX
EmpresaX  - ubicada_en  - Santiago
```

Común en: web semántica, grafos de conocimiento, ontologías, *linked data*. Puede almacenarse en XML, Turtle, JSON-LD o tablas relacionales.

### Grafo de propiedades

El modelo usado por **Neo4j**. Permite asociar propiedades **tanto a nodos como a relaciones**:

```text
(:Empleado {nombre: 'Amy', fecha_nacimiento: '1964-03-01'})
  -[:ES_CEO_DE {desde: '2008-01-20'}]->
(:Empresa {nombre: 'Acme Corp'})
```

| Modelo | Cómo representa |
|---|---|
| RDF | Triples sujeto-predicado-objeto |
| Grafo de propiedades | Nodos y relaciones con propiedades enriquecidas |

> 💡 **Para casos de ingeniería de datos**, el grafo de propiedades suele ser más intuitivo: las relaciones tienen atributos propios (fecha, peso, monto), no solo conectan entidades.

---

## Bloque 5 — Neo4j y Cypher {#bloque-5}

**Neo4j** es una base de grafos basada en el modelo de grafos de propiedades.

**Características:**

- Implementada sobre Java.
- Soporta nodos, relaciones, etiquetas y propiedades.
- Usa **Cypher** como lenguaje principal de consulta.
- Soporta transacciones ACID.
- Maneja grafos con grandes cantidades de nodos y relaciones.

### Cypher: lenguaje declarativo para grafos

La sintaxis se parece a **dibujar patrones**:

```cypher
(a)-[:RELACION]->(b)
```

Donde:

- `(a)` representa un nodo.
- `[:RELACION]` representa una relación.
- `->` indica dirección.
- `(b)` representa otro nodo.

> 💡 **Esta es la idea clave de Cypher.** En vez de describir paso a paso cómo recorrer el grafo (estilo Gremlin), describes el patrón que buscas y Neo4j encuentra los caminos. Es mucho más declarativo, similar a SQL.

### Gremlin como alternativa

**Gremlin** es otro lenguaje, más procedural:

```groovy
g.V().has('Persona', 'nombre', 'Juan').out('CONOCE').out('CONOCE')
```

| Lenguaje | Estilo |
|---|---|
| **Cypher** | Declarativo: describes el patrón |
| **Gremlin** | Procedural: describes el recorrido paso a paso |

---

## Bloque 6 — Consultas básicas en Cypher {#bloque-6}

### Crear nodos

```cypher
CREATE (a:Persona {nombre: 'Juan'});
CREATE (b:Persona {nombre: 'Ana'});
```

### Crear una relación

```cypher
MATCH (a:Persona {nombre: 'Juan'}),
      (b:Persona {nombre: 'Ana'})
CREATE (a)-[:CONOCE]->(b);
```

Resultado: `(Juan)-[:CONOCE]->(Ana)`.

### Consultar

```cypher
MATCH (a:Persona)-[:CONOCE]->(b:Persona)
RETURN a.nombre AS persona, b.nombre AS conocido;
```

| persona | conocido |
|---|---|
| Juan | Ana |

### Ejemplo completo: películas y actores

```cypher
CREATE (theMatrix:Movie {
    title: 'The Matrix',
    released: 1999,
    tagline: 'Welcome to the Real World'
});

CREATE (johnWick:Movie {
    title: 'John Wick',
    released: 2014
});

CREATE (keanu:Person {
    name: 'Keanu Reeves',
    born: 1964
});

CREATE (lana:Person {
    name: 'Lana Wachowski',
    born: 1965
});

CREATE (keanu)-[:ACTED_IN {roles: ['Neo']}]->(theMatrix);
CREATE (keanu)-[:ACTED_IN {roles: ['John Wick']}]->(johnWick);
CREATE (lana)-[:DIRECTED]->(theMatrix);
```

**Consultas útiles:**

```cypher
-- Todos los actores y películas
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name AS actor, m.title AS pelicula;

-- Películas de un actor específico
MATCH (p:Person {name: 'Keanu Reeves'})-[:ACTED_IN]->(m:Movie)
RETURN m.title AS pelicula;

-- Director y actores de cada película (patrón triple)
MATCH (director:Person)-[:DIRECTED]->(movie:Movie)<-[:ACTED_IN]-(actor:Person)
RETURN director.name AS director, movie.title AS pelicula, actor.name AS actor;
```

> 💡 **Observa la flecha bidireccional `<-[:ACTED_IN]-`** en la última consulta. Cypher permite expresar patrones donde unas relaciones van en un sentido y otras en el contrario, todo en la misma cláusula `MATCH`. Eso es muy difícil de expresar con la misma claridad en SQL.

---

## Bloque 7 — Recorridos y consultas multi-nivel {#bloque-7}

### Segundo nivel

Para el grafo `Juan → Ana → Pedro`:

```cypher
MATCH (a:Persona)-[:CONOCE]->(:Persona)-[:CONOCE]->(c:Persona)
WHERE a.nombre = 'Juan'
RETURN c.nombre AS contacto_segundo_nivel;
```

| contacto_segundo_nivel |
|---|
| Pedro |

Esta consulta retorna personas que Juan no conoce directamente, pero que son conocidas por alguien que Juan conoce — el patrón básico detrás de **"Personas que quizás conozcas"** en redes sociales.

### Recorridos de profundidad variable

```cypher
-- Conocidos a entre 1 y 3 saltos de Juan
MATCH (juan:Persona {nombre: 'Juan'})-[:CONOCE*1..3]->(persona:Persona)
WHERE persona.nombre <> 'Juan'
RETURN DISTINCT persona.nombre;
```

`[:CONOCE*1..3]` significa: seguir relaciones `CONOCE` entre 1 y 3 veces.

### Caminos más cortos

```cypher
-- ¿Cómo está conectado Juan con María?
MATCH path = shortestPath(
    (a:Persona {nombre: 'Juan'})-[:CONOCE*]-(b:Persona {nombre: 'María'})
)
RETURN path;
```

`shortestPath` encuentra la conexión más corta entre dos nodos siguiendo cualquier número de relaciones `:CONOCE`.

> 💡 **Esto es Dijkstra integrado en el lenguaje de consulta.** Lo que en la Clase 5 del Módulo 03 implementaste manualmente en Python con `heapq`, aquí lo expresas en una línea declarativa.

---

## Bloque 8 — Diseño de un grafo {#bloque-8}

Diseñar una base de grafos no es simplemente convertir tablas en nodos. Hay que pensar en las **preguntas** que se quieren responder mediante conexiones.

### Paso 1: identificar entidades

Para una plataforma de *streaming*:

```text
Usuario, Película, Serie, Género, Actor, Director
```

### Paso 2: identificar relaciones importantes

```text
Usuario  - VIO       -> Película
Usuario  - CALIFICO  -> Película
Película - PERTENECE_A -> Género
Actor    - ACTUO_EN  -> Película
Director - DIRIGIO   -> Película
```

### Paso 3: definir propiedades

```text
Usuario:   id_usuario, nombre, país
Película:  id_pelicula, título, año
VIO:       fecha, duración_vista
CALIFICO:  score, fecha
```

### Paso 4: escribir consultas objetivo

```text
Q1: recomendar películas similares a las vistas por un usuario.
Q2: encontrar actores frecuentes en películas que le gustan al usuario.
Q3: detectar comunidades de usuarios con gustos similares.
```

### Paso 5: crear restricciones e índices

En Neo4j conviene crear restricciones de unicidad:

```cypher
CREATE CONSTRAINT usuario_id_unique IF NOT EXISTS
FOR (u:Usuario)
REQUIRE u.id_usuario IS UNIQUE;

CREATE CONSTRAINT pelicula_id_unique IF NOT EXISTS
FOR (p:Pelicula)
REQUIRE p.id_pelicula IS UNIQUE;
```

> 💡 **Los `CREATE CONSTRAINT` cumplen un doble rol:** garantizan unicidad y crean automáticamente un índice. Sin esto, las búsquedas por `id_usuario` revisan todos los nodos `:Usuario` linealmente.

---

## Bloque 9 — Carga de datos desde DE {#bloque-9}

Un flujo típico:

```text
Fuentes → staging → limpieza → entidades → relaciones → grafo → consultas/analítica
```

### Carga desde CSV

```cypher
LOAD CSV WITH HEADERS FROM 'file:///usuarios.csv' AS row
MERGE (u:Usuario {id_usuario: row.id_usuario})
SET u.nombre = row.nombre,
    u.pais = row.pais;
```

```cypher
LOAD CSV WITH HEADERS FROM 'file:///visualizaciones.csv' AS row
MATCH (u:Usuario {id_usuario: row.id_usuario})
MATCH (p:Pelicula {id_pelicula: row.id_pelicula})
MERGE (u)-[r:VIO]->(p)
SET r.fecha = date(row.fecha);
```

### `CREATE` vs `MERGE`

| Operación | Comportamiento |
|---|---|
| `CREATE` | **Siempre** crea un nuevo nodo o relación |
| `MERGE` | Busca; si existe, lo reutiliza. Si no, lo crea |

> 💡 **En pipelines de datos, casi siempre quieres `MERGE`.** Las cargas incrementales pueden re-procesar los mismos registros (reintentos, *backfills*); con `CREATE` acabas duplicando todo. `MERGE` es idempotente.

---

## Bloque 10 — Casos prácticos: linaje y red social {#bloque-10}

### Caso 1: red social simple

**Modelo:**

```text
(:Persona)-[:CONOCE]->(:Persona)
(:Persona)-[:TRABAJA_EN]->(:Empresa)
(:Empresa)-[:UBICADA_EN]->(:Ciudad)
```

**Carga:**

```cypher
CREATE (juan:Persona {nombre: 'Juan'});
CREATE (ana:Persona {nombre: 'Ana'});
CREATE (pedro:Persona {nombre: 'Pedro'});
CREATE (empresa:Empresa {nombre: 'DataCorp'});
CREATE (ciudad:Ciudad {nombre: 'Santiago'});

CREATE (juan)-[:CONOCE]->(ana);
CREATE (ana)-[:CONOCE]->(pedro);
CREATE (pedro)-[:TRABAJA_EN]->(empresa);
CREATE (empresa)-[:UBICADA_EN]->(ciudad);
```

**Consultas:**

```cypher
-- Q1: Contactos directos de Juan
MATCH (:Persona {nombre: 'Juan'})-[:CONOCE]->(contacto:Persona)
RETURN contacto.nombre;

-- Q2: Contactos de segundo nivel
MATCH (:Persona {nombre: 'Juan'})-[:CONOCE]->(:Persona)-[:CONOCE]->(contacto:Persona)
RETURN contacto.nombre AS contacto_segundo_nivel;

-- Q3: Empresas donde trabajan los contactos de segundo nivel
MATCH (:Persona {nombre: 'Juan'})-[:CONOCE]->(:Persona)-[:CONOCE]->(contacto:Persona)
MATCH (contacto)-[:TRABAJA_EN]->(empresa:Empresa)
RETURN contacto.nombre AS contacto, empresa.nombre AS empresa;
```

### Caso 2: linaje de datos

**Escenario:**

- Pipeline `etl_ventas_diarias` lee `raw_ventas`.
- Escribe `dw_fact_ventas`.
- Dashboard `ventas_ejecutivo` consume `dw_fact_ventas`.

**Modelo y carga:**

```cypher
CREATE (raw:Tabla {nombre: 'raw_ventas'});
CREATE (dw:Tabla {nombre: 'dw_fact_ventas'});
CREATE (etl:Pipeline {nombre: 'etl_ventas_diarias'});
CREATE (dash:Dashboard {nombre: 'ventas_ejecutivo'});

CREATE (etl)-[:LEE]->(raw);
CREATE (etl)-[:ESCRIBE]->(dw);
CREATE (dash)-[:CONSUME]->(dw);
```

**Consulta de impacto** — *"¿Qué se ve afectado si cambia `raw_ventas`?"*:

```cypher
MATCH path = (:Tabla {nombre: 'raw_ventas'})<-[:LEE]-(p:Pipeline)-[:ESCRIBE]->(t:Tabla)<-[:CONSUME]-(d:Dashboard)
RETURN path;
```

Resultado conceptual:

```text
raw_ventas
   ↑ [:LEE]
etl_ventas_diarias
   ↓ [:ESCRIBE]
dw_fact_ventas
   ↑ [:CONSUME]
ventas_ejecutivo
```

Si `raw_ventas` cambia, podría verse afectado: el *pipeline*, la tabla derivada y el *dashboard*.

> 💡 **Este patrón es la base de los catálogos de datos modernos.** Herramientas como DataHub, OpenMetadata, Atlan y Marquez almacenan este tipo de grafos para responder preguntas de impacto, *governance* y *compliance* (qué datos personales fluyen hacia qué consumidores).

---

## Comparación: relacional vs grafo

| Aspecto | Relacional | Grafo |
|---|---|---|
| Unidad principal | Tabla / fila | Nodo / relación |
| Relación entre datos | Claves foráneas | Aristas nativas |
| Consulta típica | `JOIN`s | Recorridos de caminos |
| Fortaleza | Transacciones, datos estructurados, agregaciones tabulares | Relaciones complejas y patrones conectados |
| Debilidad | Muchos *joins* para relaciones profundas | Menos ideal para agregaciones OLAP masivas |
| Lenguaje común | SQL | Cypher, Gremlin, SPARQL |
| Casos típicos | ERP, facturación, inventario | Recomendaciones, fraude, redes, linaje |

---

## Buenas prácticas

- Modela como nodos solo las entidades que necesitas conectar o consultar.
- Usa relaciones con nombres claros y verbales: `COMPRO`, `CONOCE`, `PERTENECE_A`, `DEPENDE_DE`.
- Coloca propiedades en relaciones cuando describen el vínculo, no la entidad.
- Define identificadores estables para nodos.
- Crea índices o restricciones para búsquedas frecuentes por identificador.
- Evita convertir cada columna de una tabla en un nodo sin motivo.
- Diseña a partir de preguntas de negocio.
- Mantén consistencia en la dirección de las relaciones.
- Documenta el significado de cada etiqueta y tipo de relación.

## Errores comunes

| Error | Mejor alternativa |
|---|---|
| Pensar en tablas y no en relaciones | Modelar nodos y aristas explícitas |
| Usar grafos para datos sin recorridos | Usar warehouse columnar |
| Crear relaciones genéricas (`:RELACIONADO_CON`) | Usar relaciones específicas (`:COMPRO`, `:CONOCE`) |
| No definir identificadores únicos | Usar `MERGE` con clave única en cargas |

### ¿Cuándo NO usar grafos?

- El problema es principalmente transaccional tabular.
- Solo necesitas agregaciones masivas por fecha, región o producto.
- El modelo relacional simple resuelve bien el caso.
- No necesitas navegar relaciones profundas.
- Tu equipo y plataforma no tienen experiencia operando grafos.

---

<details>
<summary><strong>🟢 Ejercicio 1 — Patrón básico (click para ver)</strong></summary>

Tienes el siguiente grafo:

```text
Juan → Ana
Juan → Carlos
Ana  → Pedro
Carlos → Pedro
```

Escribe consultas Cypher para:

1. Obtener los conocidos directos de Juan.
2. Obtener los conocidos de segundo nivel de Juan (sin duplicar).

**Solución:**

```cypher
-- 1. Conocidos directos
MATCH (:Persona {nombre: 'Juan'})-[:CONOCE]->(c:Persona)
RETURN c.nombre;

-- 2. Conocidos de segundo nivel únicos
MATCH (:Persona {nombre: 'Juan'})-[:CONOCE*2]->(c:Persona)
WHERE c.nombre <> 'Juan'
RETURN DISTINCT c.nombre;
```

`[:CONOCE*2]` significa exactamente 2 saltos. `DISTINCT` evita duplicados (Pedro es alcanzado por dos caminos).

</details>

<details>
<summary><strong>🟢 Ejercicio 2 — Recomendación simple (click para ver)</strong></summary>

Tienes:

```text
(:Usuario)-[:VIO]->(:Película)
```

Y datos de cuatro usuarios con películas vistas. Escribe una consulta que recomiende películas a `Juan` basándose en lo que vieron usuarios que vieron las mismas películas que él.

**Solución:**

```cypher
MATCH (juan:Usuario {nombre: 'Juan'})-[:VIO]->(:Película)<-[:VIO]-(otro:Usuario)-[:VIO]->(recomendada:Película)
WHERE NOT (juan)-[:VIO]->(recomendada)
  AND otro <> juan
RETURN recomendada.titulo, COUNT(*) AS afinidad
ORDER BY afinidad DESC
LIMIT 10;
```

**Explicación:**

1. `juan` vio una película.
2. Otros usuarios también la vieron (`<-[:VIO]-`).
3. Esos usuarios además vieron otras películas (`-[:VIO]->`).
4. Excluimos las que `juan` ya vio.
5. Ordenamos por cuántos usuarios coincidentes la vieron (`COUNT(*)`).

Esta es la lógica básica de los **filtros colaborativos** que usan Netflix, Spotify y otros.

</details>

<details>
<summary><strong>🟢 Ejercicio 3 — Linaje aplicado (click para ver)</strong></summary>

Modela este escenario y escribe la consulta de impacto:

- Pipeline `bronze_orders` lee `raw_orders` (de tipo `:KafkaTopic`).
- Pipeline `silver_orders` lee `bronze_orders` (de tipo `:Tabla`) y escribe `silver_orders` (de tipo `:Tabla`).
- Pipeline `gold_sales` lee `silver_orders` y escribe `gold_sales` (de tipo `:Tabla`).
- Dashboard `ventas_diarias` consume `gold_sales`.

Pregunta: si cambia `raw_orders`, ¿qué se ve afectado?

**Solución:**

```cypher
-- Carga
CREATE (raw:KafkaTopic {nombre: 'raw_orders'});
CREATE (b:Tabla {nombre: 'bronze_orders'});
CREATE (s:Tabla {nombre: 'silver_orders'});
CREATE (g:Tabla {nombre: 'gold_sales'});
CREATE (p1:Pipeline {nombre: 'bronze_orders'});
CREATE (p2:Pipeline {nombre: 'silver_orders'});
CREATE (p3:Pipeline {nombre: 'gold_sales'});
CREATE (d:Dashboard {nombre: 'ventas_diarias'});

CREATE (p1)-[:LEE]->(raw);
CREATE (p1)-[:ESCRIBE]->(b);
CREATE (p2)-[:LEE]->(b);
CREATE (p2)-[:ESCRIBE]->(s);
CREATE (p3)-[:LEE]->(s);
CREATE (p3)-[:ESCRIBE]->(g);
CREATE (d)-[:CONSUME]->(g);

-- Consulta de impacto: qué se ve afectado si cambia raw_orders
MATCH path = (start {nombre: 'raw_orders'})<-[:LEE|ESCRIBE|CONSUME*]-(impactado)
RETURN path;
```

`<-[:LEE|ESCRIBE|CONSUME*]-` sigue cualquiera de las tres relaciones, en cualquier número de saltos, en sentido contrario (consumidores).

</details>

---

## Referencia rápida — Bases de grafos y Neo4j

```
CONCEPTOS CLAVE
─────────────────────────────────────────────────────────────────
  Nodo        entidad          (:Etiqueta {prop: valor})
  Relación    arista dirigida  -[:TIPO {prop: valor}]->
  Etiqueta    categoría de nodo
  Propiedad   atributo de nodo o relación

CYPHER BÁSICO
─────────────────────────────────────────────────────────────────
  CREATE      crea nuevo nodo o relación (siempre)
  MERGE       crea si no existe (idempotente)
  MATCH       busca patrón
  WHERE       filtra
  RETURN      devuelve resultados
  SET         actualiza propiedades
  DELETE      elimina

PATRONES
─────────────────────────────────────────────────────────────────
  (a)-[:R]->(b)              relación dirigida
  (a)<-[:R]-(b)              relación en sentido contrario
  (a)-[:R*1..3]->(b)         entre 1 y 3 saltos
  (a)-[:R*]->(b)             cualquier cantidad de saltos
  shortestPath((a)-[*]-(b))  camino más corto

RESTRICCIONES E ÍNDICES
─────────────────────────────────────────────────────────────────
  CREATE CONSTRAINT ... FOR (n:Etiqueta) REQUIRE n.id IS UNIQUE
  CREATE INDEX ... FOR (n:Etiqueta) ON (n.propiedad)

CARGA DESDE CSV
─────────────────────────────────────────────────────────────────
  LOAD CSV WITH HEADERS FROM 'file:///x.csv' AS row
  MERGE (n:Tipo {id: row.id})
  SET n.prop = row.prop

CASOS DE USO TÍPICOS
─────────────────────────────────────────────────────────────────
  Recomendaciones        usuarios + ítems + interacciones
  Detección de fraude    cuentas + transferencias + IPs
  Linaje de datos        pipelines + tablas + dashboards
  Catálogo de datos      datasets + dueños + clasificaciones
  Redes sociales         personas + relaciones
  Búsqueda de caminos    rutas, geolocalización

CUÁNDO NO USAR GRAFOS
─────────────────────────────────────────────────────────────────
  ✗ Datos puramente tabulares
  ✗ Agregaciones OLAP masivas
  ✗ Consultas que no involucran recorridos
  ✗ Sin experiencia operacional con Neo4j
```

---

*→ Volver al [Módulo 04: Modelos y Gestión de Bases de Datos](../README.md)*

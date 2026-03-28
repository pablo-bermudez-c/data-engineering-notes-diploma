# Clase 02: Terminal y Línea de Comandos para Data Engineering

## Objetivos de aprendizaje

- Navegar el sistema de archivos de Linux con confianza
- Explorar y manipular archivos de datos grandes sin cargarlos en memoria
- Construir pipelines de procesamiento combinando comandos con tuberías (*pipes*)
- Gestionar permisos de archivos y proteger credenciales con variables de entorno
- Automatizar tareas repetitivas mediante scripts de Bash (*shell scripts*)
- Programar ejecuciones automáticas con Cron

---

# Sección 1: Fundamentos y Navegación

## ¿Por qué un Data Engineer debe dominar la terminal?

Antes de escribir un solo comando, es importante entender **por qué** la terminal es una habilidad no negociable en el mundo de los datos.

| Razón | Detalle |
|---|---|
| **El servidor es tu casa** | El 90% de la nube (AWS, GCP, Azure) corre sobre Linux sin interfaz gráfica. La terminal es tu única puerta de entrada. |
| **Automatización y Orquestación** | Git, Docker, Kubernetes, Apache Airflow operan de forma nativa desde la línea de comandos. |
| **Procesamiento ligero y rápido** | Puedes limpiar, filtrar y explorar archivos CSV de 50 GB con herramientas nativas de Linux, frecuentemente más rápido que con Python o Pandas. |
| **Eficiencia** | Una vez superada la curva de aprendizaje, es la forma más directa y precisa de decirle a una computadora qué hacer. |

> **Analogía:** Usar solo una interfaz gráfica para trabajar en servidores es como intentar construir un puente con guantes de cocina. Funciona, pero nunca al nivel que se necesita.

---

## El Árbol de Directorios de Linux (FHS)

Linux organiza todos sus archivos en una estructura jerárquica llamada **Filesystem Hierarchy Standard (FHS)**. Parte siempre desde la raíz `/`, y todo lo demás cuelga de ella.

```
/
├── home/        → Directorios personales de cada usuario (/home/usuario)
├── root/        → Directorio home del superusuario (root)
├── etc/         → Archivos de configuración del sistema
├── bin/         → Binarios esenciales (ls, cp, mv, etc.)
├── sbin/        → Binarios del sistema (solo para root)
├── var/         → Datos variables que crecen con el tiempo
│   └── log/     → ← MUY IMPORTANTE: logs de errores de tus pipelines
├── tmp/         → Archivos temporales (se borran al reiniciar)
├── opt/         → Software opcional instalado manualmente
├── lib/         → Bibliotecas compartidas del sistema
└── usr/         → Aplicaciones y software de usuario
```

### Directorios clave para Data Engineering

```bash
/var/log/           # Aquí viven los logs. Fundamental para debuggear pipelines.
/home/tu_usuario/   # Tu espacio de trabajo personal.
/etc/               # Configuraciones: cron, bases de datos, servicios.
/tmp/               # Útil para archivos intermedios durante un proceso ETL.
/opt/               # Donde suelen instalarse herramientas como Airflow, Spark, etc.
```

---

## Navegación Básica: ¿Dónde estoy y qué hay aquí?

### Rutas: Absolutas vs. Relativas

- **Ruta Absoluta:** siempre comienza desde la raíz `/`. Funciona desde cualquier lugar del sistema.
  ```bash
  /home/usuario/proyectos/etl/pipeline.py
  ```
- **Ruta Relativa:** parte desde tu ubicación actual. Depende de dónde estés parado.
  ```bash
  # Si ya estás en /home/usuario/proyectos/
  etl/pipeline.py
  ```

### Comando `pwd` — Print Working Directory

Te dice exactamente en qué ruta absoluta estás en este momento.

```bash
$ pwd
/home/usuario/proyectos/etl
```

> **Tip:** Cuando te conectas por SSH a un servidor nuevo, lo primero que debes hacer es `pwd` para orientarte.

---

### Comando `ls` — List

Muestra el contenido del directorio actual.

```bash
# Listado simple
$ ls
datos_crudos/  pipeline.py  reporte_final.csv  logs/

# Formato largo: permisos, dueño, tamaño, fecha
$ ls -l
drwxr-xr-x 2 usuario grupo-usuarios 4096 mar 20 10:30 datos_crudos/
-rw-r--r-- 1 usuario grupo-usuarios 2048 mar 21 09:15 pipeline.py
-rw-r--r-- 1 usuario grupo-usuarios 15MB mar 21 09:45 reporte_final.csv

# Mostrar archivos ocultos (como .env con credenciales)
$ ls -a
.  ..  .env  datos_crudos/  pipeline.py

# Tamaño legible para humanos (KB, MB, GB)
$ ls -lh
-rw-r--r-- 1 usuario grupo-usuarios 2.5G mar 20 08:00 ventas_historicas.csv

# Combinar flags: lista larga + ocultos + legible
$ ls -lah
```

> **Caso real:** `ls -lh` es fundamental para revisar el tamaño de archivos de datos antes de procesarlos. No es lo mismo trabajar con un CSV de 2 MB que uno de 50 GB.

---

### Comando `cd` — Change Directory

Permite moverte entre carpetas.

```bash
# Ruta absoluta: ir a /var/log
$ cd /var/log

# Ruta relativa: entrar a una subcarpeta
$ cd datos_crudos

# Subir un nivel
$ cd ..

# Subir dos niveles de golpe
$ cd ../..

# Ir a tu directorio home (equivalentes)
$ cd ~
$ cd

# Volver al directorio anterior (muy útil)
$ cd -
```

> **Tip Pro:** Usa la tecla `Tab` para autocompletar nombres de archivos y carpetas. Escribe las primeras letras y presiona `Tab`; si hay una sola coincidencia, se completa automáticamente. Si hay varias, presiona `Tab` dos veces para ver todas las opciones.

---

### Comando `tree` — Ver la jerarquía

Muestra la estructura de carpetas en forma de árbol. Muy útil para documentar un proyecto.

```bash
$ tree proyectos/etl/
proyectos/etl/
├── datos/
│   ├── raw/
│   │   └── ventas_2026_01.csv
│   └── processed/
│       └── ventas_limpias.csv
├── scripts/
│   └── pipeline.sh
└── logs/
    └── pipeline_2026-03-21.log
```

---

## Gestión del Sistema de Archivos

### Comando `mkdir` — Make Directory

Crea nuevas carpetas.

```bash
# Crear una carpeta
$ mkdir datos

# Crear toda una estructura de directorios de una vez (flag -p)
$ mkdir -p proyecto/datos/raw/2026/marzo

# Estructura típica de un proyecto de Data Engineering
$ mkdir -p mi_proyecto/{datos/{raw,processed,curated},scripts,logs,notebooks}
$ tree mi_proyecto/
mi_proyecto/
├── datos/
│   ├── raw/
│   ├── processed/
│   └── curated/
├── scripts/
├── logs/
└── notebooks/
```

> **Buena práctica:** Organiza siempre tus proyectos con esta estructura de capas (raw → processed → curated). Es el patrón estándar del **Data Lakehouse**.

---

### Comando `touch` — Crear archivos vacíos

```bash
# Crear un archivo vacío (para un script que vas a escribir)
$ touch pipeline.py
$ touch ingesta.sh transformacion.sh carga.sh

# También actualiza la fecha de modificación de un archivo existente
$ touch pipeline.py  # ahora su timestamp es "ahora"
```

---

### Comando `cp` — Copy

```bash
# Copiar un archivo
$ cp ventas.csv ventas_backup.csv

# Copiar a otra carpeta
$ cp ventas.csv datos/raw/ventas.csv

# Copiar un directorio completo (flag -r de recursive)
$ cp -r datos/raw/ datos/raw_backup/

# Caso real: respaldar datos antes de un proceso destructivo
$ cp -r datos/processed/ datos/processed_backup_$(date +%Y%m%d)/
```

---

### Comando `mv` — Move / Renombrar

```bash
# Mover un archivo a otra carpeta
$ mv ventas_sucias.csv datos/raw/

# Renombrar un archivo (mismo directorio, distinto nombre)
$ mv datos.csv datos_limpios.csv

# Mover y renombrar al mismo tiempo
$ mv tmp/resultado.csv datos/processed/ventas_procesadas_2026.csv
```

---

### Comando `rm` — Remove ⚠️

```bash
# Eliminar un archivo
$ rm archivo_temporal.csv

# Eliminar una carpeta y todo su contenido (recursive)
$ rm -r carpeta_vieja/

# Pedir confirmación antes de borrar (flag -i de interactive)
$ rm -i datos_importantes.csv
rm: ¿eliminar archivo ordinario 'datos_importantes.csv'? (s/n) s

# ⚠️  PELIGRO EXTREMO: borra todo sin preguntar y sin posibilidad de recuperación
# rm -rf /    ← NUNCA ejecutes esto. Destruye el sistema completo.
# rm -rf ./   ← Borra todo en el directorio actual. Igual de peligroso.
```

> **Regla de oro:** En Linux **no existe la papelera de reciclaje** cuando usas `rm`. Lo que se borra, se borra para siempre. Ante la duda, usa `mv` para mover el archivo a `/tmp/` antes de eliminarlo definitivamente.

---

### Resumen de comandos — Sección 1

```bash
pwd                        # ¿Dónde estoy?
ls -lah                    # ¿Qué hay aquí? (detallado + ocultos + tamaño legible)
cd /ruta/absoluta          # Ir a ruta absoluta
cd carpeta_relativa        # Ir a carpeta relativa
cd ..                      # Subir un nivel
cd -                       # Volver al directorio anterior
tree proyecto/             # Ver estructura en árbol
mkdir -p a/b/c             # Crear estructura de directorios
touch archivo.py           # Crear archivo vacío
cp origen destino          # Copiar
mv origen destino          # Mover o renombrar
rm -r carpeta/             # Eliminar carpeta (con cuidado)
```

---
---

# Sección 2: Exploración de Datos y Pipelines

## Visualizando archivos de datos sin morir en el intento

En Data Engineering, frecuentemente trabajamos con archivos enormes. **Nunca debes abrir un CSV de 10 GB con Excel** (o con `cat` en la terminal). Existen herramientas diseñadas para explorar sin cargar todo en memoria.

---

### Comando `cat` — Concatenate

Imprime el contenido completo de un archivo en pantalla.

```bash
# Útil para archivos pequeños (configs, scripts cortos)
$ cat config.yml

# Útil para crear archivos rápidos
$ cat > notas.txt
Esto es una nota.
Ctrl+D para terminar.

# Concatenar archivos (su uso original)
$ cat enero.csv febrero.csv marzo.csv > trimestre_Q1.csv
```

> **⚠️ Advertencia:** `cat datos_historicos.csv` sobre un archivo de 50 GB va a bloquear tu terminal durante minutos imprimiendo millones de líneas. Usa `head` o `less` en su lugar.

---

### Comando `head` — Ver el inicio del archivo

```bash
# Ver las primeras 10 líneas (por defecto)
$ head ventas.csv
fecha,producto,region,monto,cantidad
2026-01-01,Laptop,Norte,1200000,2
2026-01-01,Mouse,Sur,15000,10
...

# Ver solo la primera línea → el encabezado del CSV
$ head -n 1 ventas.csv
fecha,producto,region,monto,cantidad

# Ver las primeras 5 líneas
$ head -n 5 ventas.csv

# Caso real: inspeccionar columnas de un dataset desconocido
$ head -n 1 dataset_desconocido.csv | tr ',' '\n'  # muestra cada columna en una línea
fecha
producto
region
monto
cantidad
```

---

### Comando `tail` — Ver el final del archivo

```bash
# Ver las últimas 10 líneas
$ tail ventas.csv

# Ver las últimas 20 líneas
$ tail -n 20 ventas.csv

# ★ El más poderoso para Data Engineering: monitorear logs en tiempo real
$ tail -f pipeline.log
# La terminal quedará "viva" mostrando nuevas líneas a medida que aparecen.
# Presiona Ctrl+C para salir.

# Ignorar la primera línea (el encabezado) — flag +2 = "desde la línea 2"
$ tail -n +2 ventas.csv
2026-01-01,Laptop,Norte,1200000,2
2026-01-01,Mouse,Sur,15000,10
...
```

> **Caso real:** Cuando lanzas un pipeline ETL en un servidor, abres otra terminal y ejecutas `tail -f pipeline.log` para monitorear el progreso y detectar errores en tiempo real.

---

### Comando `less` — Paginador interactivo

`less` es el explorador ideal para archivos grandes. **No carga todo el archivo en memoria**, solo muestra lo que necesitas ver.

```bash
$ less ventas_historicas.csv
```

**Controles dentro de `less`:**

| Tecla | Acción |
|---|---|
| `↑` / `↓` | Mover línea por línea |
| `Espacio` | Avanzar una página |
| `b` | Retroceder una página |
| `/texto` | Buscar hacia adelante |
| `?texto` | Buscar hacia atrás |
| `n` | Siguiente resultado de búsqueda |
| `G` | Ir al final del archivo |
| `1G` o `g` | Ir al inicio del archivo |
| `q` | Salir |

```bash
# Explorar un JSON grande con formato
$ cat respuesta_api.json | python3 -m json.tool | less

# Revisar logs con búsqueda
$ less /var/log/airflow/scheduler.log
# Luego dentro de less, busca errores con: /ERROR
```

---

## Flujos Estándar y Redirección (I/O)

### Concepto fundamental: stdin, stdout, stderr

Todo programa en Linux tiene tres canales de comunicación:

```
┌─────────────┐        ┌─────────────┐        ┌─────────────┐
│  stdin (0)  │ ──────▶│  PROGRAMA   │──────▶ │ stdout (1)  │
│ (entrada)   │        │             │        │  (salida)   │
└─────────────┘        └──────┬──────┘        └─────────────┘
                              │
                              ▼
                       ┌─────────────┐
                       │ stderr (2)  │
                       │  (errores)  │
                       └─────────────┘
```

### Operadores de redirección

```bash
# > Sobrescribe el archivo de salida (cuidado: borra el contenido anterior)
$ echo "ID,Nombre,Region" > clientes.csv
$ cat clientes.csv
ID,Nombre,Region

# >> Añade al final del archivo (append)
$ echo "1,Juan,Norte" >> clientes.csv
$ echo "2,María,Sur" >> clientes.csv
$ cat clientes.csv
ID,Nombre,Region
1,Juan,Norte
2,María,Sur

# 2> Redirige solo los errores a un archivo
$ python3 pipeline.py 2> errores.log

# 2>> Añade errores al log (sin borrar el histórico)
$ python3 pipeline.py >> salida.log 2>> errores.log

# &> Redirige tanto stdout como stderr al mismo archivo
$ python3 pipeline.py &> pipeline_completo.log

# /dev/null: el "agujero negro" — descarta la salida
$ python3 pipeline.py > /dev/null 2>&1   # silencia todo
```

> **Caso real de producción:**
> ```bash
> # Ejecutar pipeline guardando salida normal y errores por separado
> ./etl_ventas.sh >> /var/log/etl/ventas_$(date +%Y%m%d).log 2>> /var/log/etl/errores.log
> ```

---

## El Poder de las Tuberías (Pipes `|`)

### Filosofía Unix

> *"Escribe programas que hagan una sola cosa y la hagan bien. Escribe programas que trabajen juntos."*

### ¿Qué es un Pipe?

El símbolo `|` toma la **salida estándar** de un comando y la inyecta como **entrada estándar** del siguiente. Es el origen histórico de los pipelines de datos modernos.

```
comando_1 | comando_2 | comando_3 | comando_4
    │              ▲        │              ▲
    └──── stdout──-┘        └──── stdout──-┘
         (pipe)                  (pipe)
```

```bash
# Sin pipes: imprime todo el log, demasiado para leer
$ cat pipeline.log

# Con pipe: filtra solo las líneas de error
$ cat pipeline.log | grep "ERROR"

# Encadenando: filtra errores, cuenta cuántos hay
$ cat pipeline.log | grep "ERROR" | wc -l
42

# Caso real: top 10 procesos que más RAM consumen
$ ps aux | sort -k4 -nr | head -n 10
```

---

## Filtros y Manipulación de Texto

### Comando `grep` — Búsqueda por patrones

```bash
# Buscar líneas que contengan "ERROR" en un log
$ grep "ERROR" pipeline.log
2026-03-21 03:45:12 ERROR Conexión rechazada a la base de datos
2026-03-21 03:45:15 ERROR Timeout al insertar en tabla ventas

# Ignorar mayúsculas/minúsculas (flag -i)
$ grep -i "error" pipeline.log

# Contar cuántas ocurrencias hay (flag -c)
$ grep -c "ERROR" pipeline.log
42

# Mostrar número de línea (flag -n)
$ grep -n "ERROR" pipeline.log
145:2026-03-21 03:45:12 ERROR Conexión rechazada
148:2026-03-21 03:45:15 ERROR Timeout al insertar

# Búsqueda inversa: líneas que NO contengan el patrón (flag -v)
$ grep -v "DEBUG" pipeline.log   # Oculta las líneas de debug

# Buscar en múltiples archivos
$ grep "ERROR" /var/log/etl/*.log

# Expresiones regulares: buscar IPs
$ grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" accesos.log

# Caso real: encontrar qué tabla dio error en un log de Airflow
$ grep -i "failed" /var/log/airflow/dag_runs.log | grep "tabla_ventas"
```

---

### Comando `wc` — Word Count

```bash
# Contar líneas de un CSV (incluyendo el encabezado)
$ wc -l ventas.csv
1000001 ventas.csv   # ← 1 millón de registros + 1 encabezado

# Solo el número, sin el nombre del archivo
$ wc -l < ventas.csv
1000001

# Contar palabras (-w)
$ wc -w pipeline.sh
245 pipeline.sh

# Contar caracteres (-c)
$ wc -c pipeline.sh
1893 pipeline.sh

# Contar palabras y caracteres juntos
$ wc -w -c pipeline.sh
245 1893 pipeline.sh

# Caso real: verificar que una ingesta completó todos los registros esperados
$ ESPERADOS=50000
$ RECIBIDOS=$(wc -l < datos_ingestados.csv)
$ echo "Esperados: $ESPERADOS | Recibidos: $((RECIBIDOS - 1))"
Esperados: 50000 | Recibidos: 50000
```

---

### Comando `sort` — Ordenar

```bash
# Ordenar alfabéticamente (por defecto)
$ sort regiones.txt
Centro
Norte
Norte
Sur
Sur
Sur

# Ordenar numéricamente (flag -n)
$ sort -n numeros.txt

# Ordenar en orden inverso (flag -r)
$ sort -nr numeros.txt

# Ordenar por una columna específica en un CSV (flag -t para delimitador, -k para columna)
$ sort -t',' -k3 ventas.csv         # Por columna 3 (region), alfabético
$ sort -t',' -k4 -nr ventas.csv     # Por columna 4 (monto), numérico descendente

# Caso real: obtener los 5 productos con mayor venta
$ tail -n +2 ventas.csv | sort -t',' -k4 -nr | head -n 5
```

---

### Comando `uniq` — Colapsar duplicados

> **Importante:** `uniq` solo detecta duplicados en líneas **adyacentes**. Siempre debe ir precedido de `sort`.

```bash
# Eliminar líneas duplicadas
$ sort regiones.txt | uniq
Centro
Norte
Sur

# Contar ocurrencias de cada valor (flag -c)
$ sort regiones.txt | uniq -c
      1 Centro
      2 Norte
      3 Sur

# Ordenar por frecuencia (más frecuente primero)
$ sort regiones.txt | uniq -c | sort -nr
      3 Sur
      2 Norte
      1 Centro

# Mostrar solo los duplicados (flag -d)
$ sort regiones.txt | uniq -d
Norte
Sur

# Caso real: encontrar los tipos de error más frecuentes en un log
$ grep "ERROR" pipeline.log | awk '{print $4}' | sort | uniq -c | sort -nr
     42 ConexionRechazada
     18 Timeout
      5 PermisoDenegado
```

---

### Comando `cut` — Extraer columnas

```bash
# Extraer la columna 2 de un CSV (delimitador coma)
$ cut -d',' -f2 ventas.csv

# Extraer columnas 1 y 3
$ cut -d',' -f1,3 ventas.csv

# Extraer el encabezado para ver las columnas disponibles
$ head -n 1 ventas.csv | cut -d',' -f1-10

# Caso real: extraer solo la columna de fecha y región
$ cut -d',' -f1,3 ventas.csv | head
fecha,region
2026-01-01,Norte
2026-01-01,Sur
```

---

## Mini-ETL en Terminal: Juntando todo

Este es el concepto más poderoso de la sección. Combinando comandos simples con pipes, podemos construir un pipeline de datos completo directamente en la terminal.

### Caso práctico: Reporte de ventas por región

**Objetivo:** dado el archivo `ventas_totales.csv`, obtener cuántas ventas hay por región, ordenadas de mayor a menor, y guardar el resultado.

```bash
# El archivo de entrada:
$ head -n 4 ventas_totales.csv
fecha,producto,region,monto,cantidad
2026-01-01,Laptop,Norte,1200000,2
2026-01-02,Mouse,Sur,15000,10
2026-01-03,Teclado,Centro,35000,5
```

```bash
# El pipeline completo:
tail -n +2 ventas_totales.csv \
  | cut -d',' -f3 \
  | sort \
  | uniq -c \
  | sort -nr \
  > reporte_regiones.txt
```

**Paso a paso explicado:**

```
tail -n +2 ventas_totales.csv   # 01. Ignora la primera línea (encabezado)
         │
         ▼
   cut -d',' -f3                # 02. Extrae solo la columna 3 (region)
         │
         ▼
        sort                    # 03. Ordena las regiones alfabéticamente
         │                           (necesario para que uniq funcione)
         ▼
      uniq -c                   # 04. Cuenta ocurrencias de cada región
         │
         ▼
      sort -nr                  # 05. Ordena por conteo, de mayor a menor
         │
         ▼
  > reporte_regiones.txt        # 06. Guarda el resultado en un archivo
```

**Resultado esperado en `reporte_regiones.txt`:**
```
    152 Sur
    143 Norte
     98 Centro
     57 Oriente
```

---

### Otro ejemplo: Análisis de logs de error

```bash
# Problema: un pipeline falló. Quiero saber qué tipo de errores ocurrieron y cuántas veces.

grep "ERROR\|WARNING" /var/log/etl/pipeline.log \
  | awk '{print $3}' \
  | sort \
  | uniq -c \
  | sort -nr \
  | head -n 10 \
  > resumen_errores.txt

cat resumen_errores.txt
     42 ERROR
     18 WARNING
```

---

### Referencia rápida: Sección 2

```bash
# Visualización
cat archivo.txt             # Imprimir todo (solo archivos pequeños)
head -n 10 archivo.csv      # Ver primeras N líneas
head -n 1 archivo.csv       # Ver encabezado
tail -n 20 archivo.csv      # Ver últimas N líneas
tail -f pipeline.log        # Monitorear log en tiempo real
less archivo_grande.csv     # Explorar interactivamente

# Redirección
comando > archivo.txt       # Escribir salida (sobrescribe)
comando >> archivo.txt      # Añadir al final
comando 2> errores.log      # Redirigir errores
comando &> todo.log         # Redirigir todo

# Filtros
grep "patron" archivo       # Buscar líneas que contengan patrón
grep -c "patron" archivo    # Contar ocurrencias
grep -v "patron" archivo    # Líneas que NO contienen el patrón
wc -l archivo               # Contar líneas
cut -d',' -f3 archivo.csv   # Extraer columna 3 (delimitador coma)
sort archivo                # Ordenar alfabéticamente
sort -nr archivo            # Ordenar numérico descendente
sort | uniq                 # Eliminar duplicados
sort | uniq -c              # Contar frecuencia de cada valor
sort | uniq -c | sort -nr   # Ranking por frecuencia

# Pipeline típico de análisis
tail -n +2 datos.csv | cut -d',' -f3 | sort | uniq -c | sort -nr > reporte.txt
```

---

## Ejercicios propuestos

### Ejercicio 1 — Navegación y estructura de proyecto
Crea la siguiente estructura de directorios para un proyecto de Data Engineering:
```
proyecto_ventas/
├── datos/
│   ├── raw/
│   ├── processed/
│   └── curated/
├── scripts/
└── logs/
```
Luego crea un archivo vacío `pipeline.sh` dentro de `scripts/`.

<details>
<summary>Ver solución</summary>

```bash
mkdir -p proyecto_ventas/{datos/{raw,processed,curated},scripts,logs}
touch proyecto_ventas/scripts/pipeline.sh
tree proyecto_ventas/
```
</details>

---

### Ejercicio 2 — Exploración de datos
Dado un archivo `transacciones.csv`, responde las siguientes preguntas usando solo comandos de terminal:
1. ¿Cuántas columnas tiene?
2. ¿Cuántas filas de datos tiene (sin contar el encabezado)?
3. ¿Cuáles son los valores únicos de la columna `estado`?

<details>
<summary>Ver solución</summary>

```bash
# 1. Columnas (cuenta las comas del encabezado + 1)
head -n 1 transacciones.csv | tr ',' '\n' | wc -l

# 2. Filas de datos (total menos el encabezado)
echo "$(($(wc -l < transacciones.csv) - 1))"

# 3. Valores únicos de la columna "estado" (asumiendo que es la columna 4)
tail -n +2 transacciones.csv | cut -d',' -f4 | sort | uniq
```
</details>

---

### Ejercicio 3 — Mini-ETL
Dado el archivo `ventas_totales.csv`, construye un pipeline que:
1. Ignore el encabezado
2. Filtre solo las ventas de la región `"Norte"`
3. Extraiga el monto (columna 4)
4. Ordene de mayor a menor
5. Muestre solo el top 5
6. Guarde el resultado en `top5_ventas_norte.txt`

<details>
<summary>Ver solución</summary>

```bash
tail -n +2 ventas_totales.csv \
  | grep ",Norte," \
  | cut -d',' -f4 \
  | sort -nr \
  | head -n 5 \
  > top5_ventas_norte.txt
```
</details>

---


# Sección 3: Entorno, Permisos y Red

## Gestión de Permisos y Seguridad (`chmod`)

### ¿Por qué importan los permisos en Data Engineering?

En un entorno productivo, los permisos son la primera línea de defensa para proteger datos sensibles. Un archivo de configuración con credenciales de base de datos que cualquier usuario pueda leer es una brecha de seguridad crítica.

### El Sistema UGO

Cada archivo en Linux tiene permisos definidos para tres categorías de usuarios:

```
┌──────────────────────────────────────────────────────────┐
│                    Sistema UGO                           │
│                                                          │
│   U (User)     G (Group)    O (Others)                  │
│   ─────────    ─────────    ─────────                   │
│   Dueño del    Miembros     Todos los                   │
│   archivo      del grupo    demás                       │
└──────────────────────────────────────────────────────────┘
```

Y tres tipos de permisos posibles para cada categoría:

| Símbolo | Nombre | Valor octal | Significado para archivos | Significado para directorios |
|---|---|---|---|---|
| `r` | Read    | 4 | Leer el contenido | Listar el contenido (`ls`) |
| `w` | Write   | 2 | Modificar el archivo | Crear/borrar archivos dentro |
| `x` | Execute | 1 | Ejecutar como programa | Entrar al directorio (`cd`) |
| `-` | Ninguno | 0 | Sin permiso | Sin permiso |

### Cómo leer los permisos con `ls -l`

```bash
$ ls -l
-rwxr-xr-- 1 usuario grupo-usuarios 2048 mar 21 09:15 pipeline.sh
drwxr-x--- 2 usuario grupo-usuarios 4096 mar 20 10:30 datos_confidenciales/
```

Desglosando `-rwxr-xr--`:

```
  - rwx r-x r--
  │  │   │   │
  │  │   │   └── Others (Otros):   r-- = solo lectura (4)
  │  │   └────── Group  (Grupo):   r-x = leer y ejecutar (5)
  │  └────────── User   (Dueño):   rwx = todo (7)
  └───────────── Tipo: - archivo, d directorio, l enlace simbólico
```

### Comando `chmod` — Change Mode

**Sintaxis simbólica (fácil de leer):**

```bash
# Dar permiso de ejecución al dueño
$ chmod u+x script.sh

# Quitar permiso de escritura al grupo
$ chmod g-w datos_sensibles.csv

# Dar solo lectura a "otros"
$ chmod o=r reporte.csv

# Hacer ejecutable para todos (dueño, grupo y otros)
$ chmod +x pipeline.sh

# Quitar todos los permisos a "otros" (buena práctica con datos)
$ chmod o-rwx credenciales.env
```

**Sintaxis octal (más usada en producción):**

Cada dígito representa los permisos de U, G y O respectivamente:

```
  7  5  5
  │  │  └── Others: r-x = 4+0+1 = 5
  │  └───── Group:  r-x = 4+0+1 = 5
  └──────── User:   rwx = 4+2+1 = 7
```

```bash
# chmod 755: Dueño todo, grupo y otros pueden leer y ejecutar
# Uso típico: scripts que otros deben poder ejecutar pero no modificar
$ chmod 755 pipeline.sh

# chmod 644: Dueño lee y escribe, grupo y otros solo leen
# Uso típico: archivos de datos o configuración
$ chmod 644 config.yml

# chmod 600: Solo el dueño puede leer y escribir, nadie más ve nada
# Uso típico: archivos con credenciales (.env, claves SSH)
$ chmod 600 .env
$ chmod 600 ~/.ssh/id_rsa

# chmod 700: Solo el dueño tiene acceso total, nadie más puede ni entrar
# Uso típico: directorio con datos confidenciales
$ chmod 700 datos_confidenciales/
```

### Tabla de referencia rápida

| Octal | Simbólico | Caso de uso típico en Data Engineering |
|---|---|---|
| `777` | `rwxrwxrwx` | ⚠️ Evitar. Todos pueden hacer todo. |
| `755` | `rwxr-xr-x` | Scripts de pipeline compartidos. |
| `644` | `rw-r--r--` | Archivos de datos o configuración. |
| `640` | `rw-r-----` | Config con info semi-sensible, accesible solo por el grupo. |
| `600` | `rw-------` | `.env` con credenciales. Solo el dueño. |
| `700` | `rwx------` | Directorio privado del usuario. |

### Otros comandos relacionados

```bash
# Ver quién es el dueño de los archivos
$ ls -lh datos/

# Cambiar dueño de un archivo (requiere sudo)
$ sudo chown usuario:grupo-usuarios pipeline.sh

# Cambiar dueño recursivamente en toda una carpeta
$ sudo chown -R usuario:grupo-usuarios proyecto/

# Ver los grupos a los que pertenece tu usuario
$ groups
usuario grupo-usuarios sudo docker
```

---

## Variables de Entorno — Manejo de Credenciales

### ¿Qué son las variables de entorno?

Son variables dinámicas del sistema operativo que afectan el comportamiento de los procesos que corren en él. Almacenan configuración del entorno: rutas, usuarios, credenciales, parámetros de conexión.

```bash
# Ver TODAS las variables de entorno del sistema
$ env
# o también:
$ printenv

# Ver el valor de una variable específica
$ echo $HOME
/home/usuario

$ echo $USER
usuario

$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/home/usuario/.local/bin
```

### Variables de entorno más importantes

| Variable | Descripción | Ejemplo |
|---|---|---|
| `$HOME` | Directorio home del usuario | `/home/usuario` |
| `$USER` | Nombre del usuario actual | `usuario` |
| `$PATH` | Rutas donde el sistema busca ejecutables | `/usr/bin:/bin:...` |
| `$PWD` | Directorio de trabajo actual | `/home/usuario/proyectos` |
| `$SHELL` | Shell en uso | `/bin/bash` |
| `$EDITOR` | Editor de texto por defecto | `vim` |

### Crear y exportar variables

```bash
# Crear una variable local (solo existe en esta sesión de terminal)
$ MI_VARIABLE="hola"
$ echo $MI_VARIABLE
hola

# Exportar una variable para que los procesos hijos la hereden
$ export DB_HOST="192.168.1.100"
$ export DB_PORT="5432"
$ export DB_NAME="ventas_produccion"

# Verificar que fue creada
$ echo $DB_HOST
192.168.1.100

# Eliminar una variable
$ unset MI_VARIABLE
```

---

### ❌ La Mala Práctica: credenciales en el código

Este es uno de los errores más graves en ingeniería de datos. Las credenciales **nunca** deben estar escritas directamente en el código.

```python
# ❌ MAL: credenciales hardcodeadas en Python
import psycopg2

conn = psycopg2.connect(
    host="192.168.1.100",
    database="ventas",
    user="admin",
    password="MiContraseñaSecreta123"   # ← NUNCA hacer esto
)
```

**¿Por qué es un problema?**
- Si subes el código a GitHub, las credenciales quedan expuestas públicamente.
- Cualquier persona con acceso al código tiene las contraseñas.
- Cambiar la contraseña requiere modificar el código y redesplegar.

---

### ✅ La Buena Práctica: inyección por variables de entorno

**Paso 1:** Crear un archivo `.env` con las credenciales.

```bash
# Crear el archivo .env
$ nano .env
```

```bash
# Contenido del archivo .env
DB_HOST=192.168.1.100
DB_PORT=5432
DB_NAME=ventas_produccion
DB_USER=etl_user
DB_PASSWORD=MiContraseñaSecreta123
API_KEY=sk-abc123def456
```

**Paso 2:** Asegurar el archivo con permisos estrictos.

```bash
# Solo el dueño puede leer y escribir este archivo
$ chmod 600 .env

# Agregar .env al .gitignore para que NUNCA se suba al repositorio
$ echo ".env" >> .gitignore
```

**Paso 3:** Cargar las variables en la terminal.

```bash
# Cargar las variables del archivo .env en la sesión actual
$ export $(cat .env | xargs)

# Verificar que fueron cargadas
$ echo $DB_HOST
192.168.1.100
```

**Paso 4:** Leer las variables desde Python.

```python
# ✅ BIEN: leer credenciales desde variables de entorno
import os
import psycopg2

conn = psycopg2.connect(
    host=os.getenv('DB_HOST'),
    port=os.getenv('DB_PORT'),
    database=os.getenv('DB_NAME'),
    user=os.getenv('DB_USER'),
    password=os.getenv('DB_PASSWORD')
)

# Con valor por defecto si la variable no existe
api_key = os.getenv('API_KEY', 'valor_por_defecto')
```

### Persistencia: variables que sobreviven al cierre de terminal

Las variables definidas con `export` desaparecen al cerrar la terminal. Para hacerlas permanentes, se agregan al archivo de configuración del shell.

```bash
# Abrir el archivo de configuración del shell
$ nano ~/.bashrc       # Para Bash (más común)
# o
$ nano ~/.profile      # Para sesiones de login

# Agregar al final del archivo:
export DB_HOST="192.168.1.100"
export DB_PORT="5432"

# Aplicar los cambios sin cerrar la terminal
$ source ~/.bashrc
```

> **Tip:** Para credenciales en servidores de producción, lo ideal es usar un gestor de secretos como **AWS Secrets Manager**, **HashiCorp Vault** o **Azure Key Vault**. Las variables de entorno son un buen punto de partida, pero los secrets managers son la solución empresarial.

---

## Monitoreo de Procesos y Descargas Web

### Administración de Recursos del Sistema

#### Comando `top` — Monitor en tiempo real

```bash
$ top
```

```
top - 10:25:01 up 5 days,  2:34,  2 users,  load average: 1.23, 0.98, 0.75
Tasks: 215 total,   2 running, 213 sleeping
%Cpu(s): 45.2 us, 12.3 sy,  0.0 ni, 38.1 id
MiB Mem :  15928.4 total,   1204.5 free,  12301.7 used,   2422.2 buff/cache
MiB Swap:   2048.0 total,   1998.4 free,     49.6 used.   2100.3 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
12345 usuario    20   0   15.2g   9.8g  42312 R  87.3  63.1   2:45.21 spark-submit
 5678 usuario    20   0  512348  45231  12345 S   3.2   0.3   0:12.45 python3
```

**Columnas clave para Data Engineering:**

| Columna | Significado |
|---|---|
| `PID` | ID del proceso. Necesario para `kill`. |
| `%CPU` | Porcentaje de CPU que está usando. |
| `%MEM` | Porcentaje de RAM que está usando. |
| `RES` | Memoria RAM real utilizada. |
| `COMMAND` | Nombre del proceso. |

**Controles dentro de `top`:**

| Tecla | Acción |
|---|---|
| `M` | Ordenar por uso de memoria |
| `P` | Ordenar por uso de CPU |
| `k` | Matar un proceso (pide el PID) |
| `q` | Salir |

---

#### Comando `htop` — Monitor mejorado (recomendado)

`htop` es la versión moderna y visual de `top`. Si no está instalado:

```bash
$ sudo apt install htop   # Debian/Ubuntu
```

```bash
$ htop
```

Ventajas sobre `top`: barras de progreso para CPU y memoria, navegación con flechas, colores, y más intuitivo para matar procesos.

---

#### Comando `ps` — Snapshot de procesos

```bash
# Ver todos los procesos del sistema con detalle
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
usuario   12345 87.3 63.1 15728640 10200000 ?  R    09:40   2:45 spark-submit etl.py
usuario    5678  3.2  0.3  512348  45231 pts/0 S    10:20   0:12 python3 pipeline.py

# Buscar un proceso específico por nombre
$ ps aux | grep "python"

# Ver solo los procesos de tu usuario
$ ps -u usuario

# Ver árbol de procesos (proceso padre → hijos)
$ ps -ef --forest
```

---

#### Comando `kill` — Terminar procesos

```bash
# Terminar un proceso "amablemente" (le da tiempo a limpiar)
$ kill 12345              # Envía señal SIGTERM (15)

# Terminar un proceso de forma forzada e inmediata
$ kill -9 12345           # Envía señal SIGKILL (9)
# También se puede escribir como:
$ kill -SIGKILL 12345

# Matar todos los procesos con un nombre específico
$ killall python3

# Caso real: un script de ingesta se colgó y consume 100% de CPU
$ ps aux | grep "ingesta_datos.py"
usuario   99123 99.9  45.2  ...  python3 ingesta_datos.py

$ kill -9 99123
```

> **Diferencia clave:**
> - `kill PID` (SIGTERM): le dice al proceso "por favor, termina limpiamente". El proceso puede hacer cleanup (cerrar conexiones a BD, escribir logs).
> - `kill -9 PID` (SIGKILL): el sistema operativo mata el proceso de inmediato, sin que pueda hacer nada. Usar solo cuando SIGTERM no funciona.

---

### Descarga de Datos e Interacción Web

#### Comando `wget` — Descargar archivos

```bash
# Descarga básica: guarda el archivo en el directorio actual
$ wget https://dominio.com/dataset_ventas.csv

# Guardar con un nombre específico (flag -O)
$ wget -O ventas_2026.csv https://dominio.com/dataset_ventas.csv

# Descarga silenciosa, ideal para scripts (flag -q)
$ wget -q https://dominio.com/dataset.csv

# Continuar una descarga interrumpida (flag -c, muy útil con archivos grandes)
$ wget -c https://dominio.com/dataset_50gb.csv

# Descargar múltiples archivos desde una lista
$ cat lista_urls.txt
https://datos.gob.cl/dataset/enero_2026.csv
https://datos.gob.cl/dataset/febrero_2026.csv
https://datos.gob.cl/dataset/marzo_2026.csv

$ wget -i lista_urls.txt

# Descargar con reintentos automáticos (flag --tries)
$ wget --tries=5 https://dominio.com/dataset.csv
```

---

#### Comando `curl` — Transferir datos y llamar APIs

`curl` es más versátil que `wget` y es el estándar para interactuar con APIs REST desde la terminal.

```bash
# Llamada GET básica a una API
$ curl https://api.datos.gob.cl/v1/indicadores

# Ver la respuesta con formato legible (redirigir a python)
$ curl https://api.datos.gob.cl/v1/indicadores | python3 -m json.tool

# Guardar la respuesta en un archivo (flag -o)
$ curl -o respuesta.json https://api.datos.gob.cl/v1/indicadores

# Llamada POST con datos JSON (flag -X, -H para headers, -d para datos)
$ curl -X POST "https://api.ejemplo.com/ingresar" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $API_KEY" \
     -d '{"fecha": "2026-03-21", "region": "Norte"}'

# Llamada GET con parámetros de consulta
$ curl "https://api.datos.gob.cl/v1/ventas?region=Norte&año=2026"

# Ver los headers de respuesta HTTP (útil para debugging)
$ curl -I https://api.ejemplo.com/v1/datos

# Seguir redirecciones automáticamente (flag -L)
$ curl -L https://short.url/dataset

# Modo silencioso, solo muestra la respuesta (flag -s)
$ curl -s https://api.ejemplo.com/v1/status | python3 -m json.tool
```

**Diferencia entre `wget` y `curl`:**

| | `wget` | `curl` |
|---|---|---|
| **Uso principal** | Descargar archivos | Transferir datos / llamar APIs |
| **Protocolos** | HTTP, HTTPS, FTP | HTTP, HTTPS, FTP, SFTP y más |
| **Descarga recursiva** | ✅ Sí | ❌ No |
| **Reintentos automáticos** | ✅ Sí | Manual |
| **Llamadas REST/API** | Limitado | ✅ Ideal |
| **Uso en scripts** | Descargas simples | Integraciones con APIs |

---

#### Comando `ssh` — Conexión Remota Segura

```bash
# Conectarse a un servidor remoto
$ ssh usuario@192.168.1.100

# Conectarse a un servidor con un puerto diferente al 22
$ ssh -p 2222 usuario@servidor.empresa.com

# Conectarse usando una clave SSH (más seguro que contraseñas)
$ ssh -i ~/.ssh/mi_clave.pem ubuntu@54.123.45.67

# Ejecutar un comando remoto sin abrir sesión interactiva
$ ssh usuario@servidor "tail -f /var/log/etl/pipeline.log"

# Copiar archivos de forma segura entre servidores (scp)
$ scp datos_locales.csv usuario@servidor:/home/usuario/datos/
$ scp usuario@servidor:/var/log/etl/pipeline.log ./logs/

# Copiar directorios completos con scp
$ scp -r ./proyecto/ usuario@servidor:/home/usuario/
```

---

### Resumen de comandos — Sección 3

```bash
# Permisos
ls -l archivo               # Ver permisos actuales
chmod +x script.sh          # Hacer ejecutable
chmod 600 .env              # Solo el dueño lee/escribe (credenciales)
chmod 755 pipeline.sh       # Dueño todo, resto lee y ejecuta
chown usuario:grupo archivo # Cambiar dueño

# Variables de entorno
env                         # Ver todas las variables
echo $VARIABLE              # Ver una variable específica
export CLAVE="valor"        # Crear/exportar variable
source ~/.bashrc            # Recargar configuración
unset CLAVE                 # Eliminar variable

# Monitoreo de procesos
top                         # Monitor en tiempo real
htop                        # Monitor mejorado
ps aux                      # Snapshot de todos los procesos
ps aux | grep "nombre"      # Buscar proceso por nombre
kill PID                    # Terminar proceso (amable)
kill -9 PID                 # Forzar terminación
killall nombre              # Matar todos los procesos con ese nombre

# Red y descargas
wget URL                    # Descargar archivo
wget -c URL                 # Continuar descarga interrumpida
curl URL                    # Llamada HTTP/API
curl -o archivo.json URL    # Guardar respuesta
ssh usuario@servidor        # Conectar a servidor remoto
scp archivo usuario@srv:/ruta  # Copiar archivo a servidor remoto
```

---
---

# Sección 4: Bash Scripting y Automatización

## ¿Qué es un Script de Bash?

Un script de Bash es un **archivo de texto** que contiene una secuencia de comandos de Linux. En lugar de escribir los mismos comandos una y otra vez en la terminal, los empaquetas en un archivo y lo ejecutas cuando lo necesitas.

**Los tres pasos para crear y ejecutar cualquier script:**

```bash
# Paso 1: Escribir el script
$ nano mi_script.sh

# Paso 2: Darle permisos de ejecución
$ chmod +x mi_script.sh

# Paso 3: Ejecutarlo
$ ./mi_script.sh
```

### El Shebang `#!/bin/bash`

La **primera línea** de todo script debe indicarle al sistema qué intérprete usar. Esta línea se llama *shebang*.

```bash
#!/bin/bash
# ↑ Le dice al sistema: "ejecuta este archivo con Bash que está en /bin/bash"

# Alternativa más portable (busca bash en el PATH)
#!/usr/bin/env bash
```

### Tu primer script

```bash
#!/bin/bash
# hola_data.sh — Mi primer script de Data Engineering

echo "Iniciando proceso ETL..."
echo "Fecha y hora: $(date)"
echo "Usuario: $USER"
echo "Directorio de trabajo: $PWD"
echo "Proceso completado."
```

```bash
$ chmod +x hola_data.sh
$ ./hola_data.sh
Iniciando proceso ETL...
Fecha y hora: sab 21 mar 2026 10:30:15 -03
Usuario: usuario
Directorio de trabajo: /home/usuario/scripts
Proceso completado.
```

---

## Variables en Bash

```bash
#!/bin/bash

# Definir variables (sin espacios alrededor del =)
NOMBRE_PROYECTO="etl_ventas"
VERSION="2.1"
MAX_REINTENTOS=3

# Usar variables con el prefijo $
echo "Proyecto: $NOMBRE_PROYECTO versión $VERSION"

# Capturar la salida de un comando en una variable (command substitution)
FECHA_HOY=$(date +%Y-%m-%d)
HORA_ACTUAL=$(date +%H:%M:%S)
NUMERO_ARCHIVOS=$(ls datos/raw/*.csv | wc -l)

echo "Ejecutando el $FECHA_HOY a las $HORA_ACTUAL"
echo "Archivos a procesar: $NUMERO_ARCHIVOS"

# Variables con nombres de archivos dinámicos (muy común en ETL)
ARCHIVO_ENTRADA="ventas_${FECHA_HOY}.csv"
ARCHIVO_SALIDA="ventas_procesadas_${FECHA_HOY}.csv"
LOG_FILE="/var/log/etl/${NOMBRE_PROYECTO}_${FECHA_HOY}.log"

echo "Procesando: $ARCHIVO_ENTRADA"
echo "Guardando en: $ARCHIVO_SALIDA"
echo "Log en: $LOG_FILE"
```

---

## Argumentos de Línea de Comandos

Los scripts pueden recibir parámetros al momento de ejecutarse, haciéndolos reutilizables.

```bash
#!/bin/bash
# procesar_region.sh — Acepta argumentos externos

# Variables especiales de Bash:
# $0 → nombre del script
# $1, $2, $3 → primer, segundo, tercer argumento
# $@ → todos los argumentos como lista
# $# → cantidad de argumentos recibidos
# $? → código de salida del último comando (0 = éxito, otro = error)

REGION=$1
FECHA=$2
AMBIENTE=$3

echo "Script: $0"
echo "Región a procesar: $REGION"
echo "Fecha: $FECHA"
echo "Ambiente: $AMBIENTE"
echo "Total de argumentos recibidos: $#"
```

```bash
$ chmod +x procesar_region.sh
$ ./procesar_region.sh Norte 2026-03-21 produccion

Script: ./procesar_region.sh
Región a procesar: Norte
Fecha: 2026-03-21
Ambiente: produccion
Total de argumentos recibidos: 3
```

---

## Condicionales (`if`)

### Estructura básica

```bash
if [ condición ]; then
    # comandos si es verdadero
elif [ otra_condición ]; then
    # comandos si la segunda condición es verdadera
else
    # comandos si ninguna condición es verdadera
fi
```

### Verificaciones de archivos y directorios

```bash
#!/bin/bash
# verificar_ingesta.sh

ARCHIVO_INGESTA="datos/raw/ventas_$(date +%Y-%m-%d).csv"

# -f → ¿Es un archivo que existe?
if [ -f "$ARCHIVO_INGESTA" ]; then
    echo "✅ Archivo encontrado: $ARCHIVO_INGESTA"
    LINEAS=$(wc -l < "$ARCHIVO_INGESTA")
    echo "   Registros: $((LINEAS - 1))"
else
    echo "❌ ERROR: No se encontró el archivo $ARCHIVO_INGESTA"
    echo "   Verifica que la ingesta del día se haya completado."
    exit 1   # Salir con código de error (≠ 0)
fi

# -d → ¿Es un directorio que existe?
if [ ! -d "datos/processed" ]; then
    echo "Directorio de salida no existe. Creándolo..."
    mkdir -p datos/processed
fi

# -s → ¿El archivo existe y NO está vacío?
if [ -s "$ARCHIVO_INGESTA" ]; then
    echo "✅ El archivo tiene contenido."
else
    echo "⚠️  ADVERTENCIA: El archivo existe pero está vacío."
fi
```

### Comparaciones numéricas y de texto

```bash
#!/bin/bash

REGISTROS=$(wc -l < ventas.csv)
MINIMO_ESPERADO=1000

# Comparaciones numéricas
# -eq: igual, -ne: distinto, -gt: mayor, -lt: menor, -ge: mayor o igual, -le: menor o igual

if [ "$REGISTROS" -lt "$MINIMO_ESPERADO" ]; then
    echo "⚠️  ALERTA: Solo $REGISTROS registros. Se esperaban al menos $MINIMO_ESPERADO."
elif [ "$REGISTROS" -ge "$MINIMO_ESPERADO" ]; then
    echo "✅ Cantidad de registros OK: $REGISTROS"
fi

# Comparaciones de texto
AMBIENTE=$1

if [ "$AMBIENTE" = "produccion" ]; then
    echo "Ejecutando en PRODUCCIÓN. Máxima precaución."
elif [ "$AMBIENTE" = "staging" ]; then
    echo "Ejecutando en STAGING."
else
    echo "❌ Ambiente desconocido: $AMBIENTE"
    echo "Uso: $0 [produccion|staging|desarrollo]"
    exit 1
fi

# Verificar si una variable está vacía
if [ -z "$AMBIENTE" ]; then
    echo "❌ Debes especificar un ambiente."
    exit 1
fi

# Verificar si una variable NO está vacía
if [ -n "$AMBIENTE" ]; then
    echo "Ambiente configurado: $AMBIENTE"
fi
```

---

## Bucles (`for`)

### Iterar sobre una lista

```bash
#!/bin/bash
# procesar_regiones.sh

REGIONES=("Norte" "Sur" "Centro" "Oriente" "Poniente")

for REGION in "${REGIONES[@]}"; do
    echo "Procesando región: $REGION"
    # Aquí iría el comando real, ej: python3 etl.py --region "$REGION"
done
```

### Iterar sobre archivos

```bash
#!/bin/bash
# batch_procesamiento.sh — Procesa todos los CSV de una carpeta

CARPETA_ENTRADA="datos/raw"
CARPETA_SALIDA="datos/processed"

echo "Iniciando procesamiento batch: $(date)"
echo "---"

CONTADOR=0

for ARCHIVO in "$CARPETA_ENTRADA"/*.csv; do
    NOMBRE_ARCHIVO=$(basename "$ARCHIVO")   # Quita la ruta, deja solo el nombre
    SALIDA="$CARPETA_SALIDA/${NOMBRE_ARCHIVO%.csv}_procesado.csv"
    # "${VARIABLE%.csv}" → elimina la extensión .csv del final

    echo "Procesando: $NOMBRE_ARCHIVO"
    python3 scripts/transformar.py --input "$ARCHIVO" --output "$SALIDA"

    if [ $? -eq 0 ]; then   # $? es el código de salida del último comando
        echo "  ✅ OK"
        CONTADOR=$((CONTADOR + 1))
    else
        echo "  ❌ Error al procesar $NOMBRE_ARCHIVO"
    fi
done

echo "---"
echo "Procesamiento completado: $CONTADOR archivos OK"
```

### Bucle `while` — Esperar hasta que algo ocurra

```bash
#!/bin/bash
# esperar_archivo.sh — Espera a que un archivo de ingesta esté disponible

ARCHIVO_ESPERADO="datos/raw/ventas_$(date +%Y-%m-%d).csv"
MAX_ESPERA=60   # minutos
INTENTOS=0

while [ ! -f "$ARCHIVO_ESPERADO" ]; do
    INTENTOS=$((INTENTOS + 1))

    if [ "$INTENTOS" -gt "$MAX_ESPERA" ]; then
        echo "❌ Timeout: El archivo no llegó en $MAX_ESPERA minutos."
        exit 1
    fi

    echo "Esperando archivo... intento $INTENTOS/$MAX_ESPERA"
    sleep 60   # Esperar 1 minuto
done

echo "✅ Archivo disponible. Iniciando procesamiento."
```

---

## Script completo: Pipeline ETL orquestado

Juntando todo lo aprendido en un script real de producción:

```bash
#!/bin/bash
# =============================================================
# pipeline_etl_ventas.sh
# Orquestador del pipeline ETL de ventas diario
#
# Uso: ./pipeline_etl_ventas.sh [produccion|staging]
# Ejemplo: ./pipeline_etl_ventas.sh produccion
# =============================================================

# ---- Configuración ----
AMBIENTE=${1:-"staging"}          # Usa "staging" si no se pasa argumento
FECHA=$(date +%Y-%m-%d)
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Rutas
DIR_BASE="/home/usuario/proyecto_ventas"
DIR_RAW="$DIR_BASE/datos/raw"
DIR_PROCESSED="$DIR_BASE/datos/processed"
DIR_LOGS="$DIR_BASE/logs"
LOG_FILE="$DIR_LOGS/pipeline_${TIMESTAMP}.log"

# Función para logging
log() {
    echo "[$(date +%H:%M:%S)] $1" | tee -a "$LOG_FILE"
}

# ---- Inicio del pipeline ----
log "=============================="
log "Iniciando pipeline ETL"
log "Ambiente: $AMBIENTE | Fecha: $FECHA"
log "=============================="

# Paso 1: Verificar que existen los directorios necesarios
log "PASO 1: Verificando estructura de directorios..."
for DIR in "$DIR_RAW" "$DIR_PROCESSED" "$DIR_LOGS"; do
    if [ ! -d "$DIR" ]; then
        log "  Creando directorio: $DIR"
        mkdir -p "$DIR"
    fi
done
log "  ✅ Directorios verificados."

# Paso 2: Verificar que llegó el archivo de ingesta del día
log "PASO 2: Verificando archivo de entrada..."
ARCHIVO_ENTRADA="$DIR_RAW/ventas_${FECHA}.csv"

if [ ! -f "$ARCHIVO_ENTRADA" ]; then
    log "  ❌ ERROR CRÍTICO: No se encontró $ARCHIVO_ENTRADA"
    exit 1
fi

REGISTROS_ENTRADA=$(wc -l < "$ARCHIVO_ENTRADA")
log "  ✅ Archivo encontrado. Registros: $((REGISTROS_ENTRADA - 1))"

# Paso 3: Transformación
log "PASO 3: Ejecutando transformaciones..."
ARCHIVO_SALIDA="$DIR_PROCESSED/ventas_procesadas_${FECHA}.csv"

python3 "$DIR_BASE/scripts/transformar.py" \
    --input "$ARCHIVO_ENTRADA" \
    --output "$ARCHIVO_SALIDA" \
    --ambiente "$AMBIENTE" \
    >> "$LOG_FILE" 2>&1

if [ $? -ne 0 ]; then
    log "  ❌ ERROR: Falló la transformación. Revisa el log: $LOG_FILE"
    exit 1
fi
log "  ✅ Transformación completada."

# Paso 4: Validación del archivo de salida
log "PASO 4: Validando archivo de salida..."
if [ ! -s "$ARCHIVO_SALIDA" ]; then
    log "  ❌ ERROR: El archivo de salida está vacío."
    exit 1
fi

REGISTROS_SALIDA=$(wc -l < "$ARCHIVO_SALIDA")
log "  ✅ Registros en salida: $((REGISTROS_SALIDA - 1))"

# Paso 5: Solo en producción, hacer backup
if [ "$AMBIENTE" = "produccion" ]; then
    log "PASO 5: Haciendo backup (solo producción)..."
    BACKUP_DIR="$DIR_BASE/backups/$FECHA"
    mkdir -p "$BACKUP_DIR"
    cp "$ARCHIVO_SALIDA" "$BACKUP_DIR/"
    log "  ✅ Backup guardado en $BACKUP_DIR"
fi

# ---- Finalización ----
log "=============================="
log "✅ Pipeline completado exitosamente."
log "Duración: $SECONDS segundos."
log "=============================="

exit 0
```

---

## Cron y Automatización

### ¿Qué es Cron?

Cron es el **programador de tareas** de Linux. Permite ejecutar scripts y comandos de forma automática en intervalos de tiempo definidos: cada minuto, cada hora, a las 3 AM, los lunes, el primer día del mes, etc.

### Comandos básicos de Cron

```bash
# Editar las tareas programadas del usuario actual
$ crontab -e

# Ver las tareas programadas actuales
$ crontab -l

# Eliminar todas las tareas programadas (¡cuidado!)
$ crontab -r
```

### Sintaxis de Cron

```
┌──────── Minuto        (0-59)
│ ┌────── Hora          (0-23)
│ │ ┌──── Día del mes   (1-31)
│ │ │ ┌── Mes           (1-12)
│ │ │ │ ┌ Día semana    (0-7, donde 0 y 7 = Domingo)
│ │ │ │ │
* * * * *  comando_a_ejecutar
```

**El asterisco `*` significa "cualquier valor" (cada minuto, cada hora, etc.)**

### Ejemplos de expresiones Cron

```bash
# Ejecutar el pipeline todos los días a las 3:30 AM
30 3 * * * /home/usuario/scripts/pipeline_etl_ventas.sh produccion

# Ejecutar cada hora en punto
0 * * * * /home/usuario/scripts/actualizar_cache.sh

# Ejecutar cada 15 minutos
*/15 * * * * /home/usuario/scripts/health_check.sh

# Ejecutar solo los lunes a las 6 AM (reporte semanal)
0 6 * * 1 /home/usuario/scripts/reporte_semanal.sh

# Ejecutar el primer día de cada mes a medianoche (reporte mensual)
0 0 1 * * /home/usuario/scripts/cierre_mensual.sh

# Ejecutar de lunes a viernes a las 8 AM
0 8 * * 1-5 /home/usuario/scripts/ingesta_diaria.sh

# Ejecutar cada 5 minutos solo en horario hábil (9 AM a 6 PM, L-V)
*/5 9-18 * * 1-5 /home/usuario/scripts/monitorear_api.sh
```

> **Herramienta útil:** [crontab.guru](https://crontab.guru) — Pega tu expresión cron y te dice en palabras qué hace. Imprescindible para verificar antes de poner en producción.

### Crontab completo de un Data Engineer

```bash
$ crontab -e

# === Pipeline ETL de Ventas ===
# Ingesta diaria a las 3:00 AM (los datos del día anterior llegan a las 2 AM)
0 3 * * * /home/usuario/scripts/pipeline_etl_ventas.sh produccion >> /var/log/etl/cron.log 2>&1

# Verificación a las 4 AM de que la ingesta fue exitosa
0 4 * * * /home/usuario/scripts/verificar_ingesta.sh >> /var/log/etl/cron.log 2>&1

# === Reportes ===
# Reporte semanal los lunes a las 7 AM
0 7 * * 1 /home/usuario/scripts/reporte_semanal.sh >> /var/log/reportes/cron.log 2>&1

# Cierre mensual el día 1 a las 1 AM
0 1 1 * * /home/usuario/scripts/cierre_mensual.sh produccion >> /var/log/etl/cierre.log 2>&1

# === Mantenimiento ===
# Limpiar archivos temporales mayores a 7 días, todos los domingos a las 2 AM
0 2 * * 0 find /tmp/etl/ -type f -mtime +7 -delete

# Rotar logs mayores a 100MB, cada día a medianoche
0 0 * * * find /var/log/etl/ -size +100M -exec gzip {} \;
```

### Buenas prácticas con Cron

```bash
# ✅ 1. Usa SIEMPRE rutas absolutas en los comandos
# ❌ MAL:
0 3 * * * pipeline.sh
# ✅ BIEN:
0 3 * * * /home/usuario/scripts/pipeline.sh

# ✅ 2. Redirige la salida para no perder los logs de Cron
0 3 * * * /home/usuario/scripts/pipeline.sh >> /var/log/etl/pipeline.log 2>&1

# ✅ 3. Prueba el comando manualmente ANTES de agregarlo a crontab
$ /home/usuario/scripts/pipeline.sh produccion

# ✅ 4. Documenta cada tarea con un comentario (#)
# === Pipeline de ventas diario ===
0 3 * * * /home/usuario/scripts/pipeline_etl_ventas.sh produccion

# ✅ 5. En scripts, usa set -e para que el script falle ante el primer error
#!/bin/bash
set -e          # Salir inmediatamente si algún comando falla
set -u          # Tratar variables no definidas como error
set -o pipefail # El pipe falla si cualquier comando del pipeline falla
```

---

## Buenas Prácticas Generales de Bash Scripting

### 1. Encabezado estándar de un script

```bash
#!/bin/bash
# =============================================================
# nombre_script.sh
# Descripción: Qué hace este script y para qué sirve.
# Autor: Tu nombre
# Fecha: 2026-03-21
# Versión: 1.0
#
# Uso: ./nombre_script.sh [argumento1] [argumento2]
# Ejemplo: ./nombre_script.sh produccion 2026-03-21
# =============================================================
set -euo pipefail   # Activa modo estricto: falla rápido ante errores
```

### 2. Validar los argumentos de entrada

```bash
#!/bin/bash

# Verificar que se recibió el número correcto de argumentos
if [ "$#" -ne 2 ]; then
    echo "Error: Se requieren exactamente 2 argumentos."
    echo "Uso: $0 <ambiente> <fecha>"
    echo "Ejemplo: $0 produccion 2026-03-21"
    exit 1
fi

AMBIENTE=$1
FECHA=$2

# Validar que el ambiente sea uno de los esperados
if [[ "$AMBIENTE" != "produccion" && "$AMBIENTE" != "staging" && "$AMBIENTE" != "desarrollo" ]]; then
    echo "Error: Ambiente inválido '$AMBIENTE'"
    echo "Valores permitidos: produccion, staging, desarrollo"
    exit 1
fi
```

### 3. Funciones para reutilizar código

```bash
#!/bin/bash

# Definir funciones
log_info() {
    echo "[INFO]  $(date +%H:%M:%S) - $1"
}

log_error() {
    echo "[ERROR] $(date +%H:%M:%S) - $1" >&2
}

verificar_archivo() {
    local ARCHIVO=$1
    if [ ! -f "$ARCHIVO" ]; then
        log_error "Archivo no encontrado: $ARCHIVO"
        return 1
    fi
    log_info "Archivo verificado: $ARCHIVO"
    return 0
}

# Usar las funciones
log_info "Iniciando proceso..."
verificar_archivo "datos/raw/ventas.csv" || exit 1
log_info "Todo OK."
```

---

### Resumen final: Sección 4

```bash
# Estructura básica de un script
#!/bin/bash
set -euo pipefail

# Variables
VARIABLE="valor"
FECHA=$(date +%Y-%m-%d)      # Command substitution

# Argumentos
$0  # Nombre del script
$1  # Primer argumento
$#  # Cantidad de argumentos
$?  # Código de salida del último comando

# Condicionales
if [ -f "$ARCHIVO" ]; then ...        # ¿Existe el archivo?
if [ -d "$DIR" ]; then ...            # ¿Existe el directorio?
if [ -z "$VAR" ]; then ...            # ¿La variable está vacía?
if [ "$NUM" -gt 100 ]; then ...       # ¿Es mayor que 100?
if [ "$STR" = "valor" ]; then ...     # ¿Son iguales los strings?

# Bucles
for ITEM in "${LISTA[@]}"; do ...     # Iterar sobre lista
for ARCHIVO in carpeta/*.csv; do ...  # Iterar sobre archivos
while [ condicion ]; do ...           # Repetir mientras condición sea verdadera

# Cron
crontab -e                            # Editar tareas
crontab -l                            # Listar tareas
# Sintaxis: Minuto Hora DíaMes Mes DíaSemana Comando
0 3 * * * /ruta/absoluta/script.sh >> /ruta/log.log 2>&1
```

---

## Ejercicios propuestos

### Ejercicio 1 — Permisos y seguridad
1. Crea un archivo `.env` con variables ficticias de conexión a base de datos.
2. Aplica permisos `600` al archivo.
3. Verifica con `ls -l` que los permisos son correctos.
4. Carga las variables con `export $(cat .env | xargs)` y verifica con `echo`.

<details>
<summary>Ver solución</summary>

```bash
# 1. Crear el archivo
cat > .env << EOF
DB_HOST=localhost
DB_PORT=5432
DB_NAME=mi_base
DB_USER=admin
DB_PASSWORD=secreto123
EOF

# 2. Asegurar permisos
chmod 600 .env

# 3. Verificar
ls -l .env
# -rw------- 1 matias matias 95 mar 21 10:30 .env

# 4. Cargar y verificar
export $(cat .env | xargs)
echo $DB_HOST
# localhost
```
</details>

---

### Ejercicio 2 — Script con validaciones
Crea un script `verificar_datos.sh` que:
1. Reciba como argumento el nombre de un archivo CSV.
2. Verifique que el archivo existe.
3. Cuente cuántas líneas tiene (sin el encabezado).
4. Avise si tiene menos de 100 registros (posible error de ingesta).

<details>
<summary>Ver solución</summary>

```bash
#!/bin/bash
# verificar_datos.sh

if [ "$#" -ne 1 ]; then
    echo "Uso: $0 <archivo.csv>"
    exit 1
fi

ARCHIVO=$1

if [ ! -f "$ARCHIVO" ]; then
    echo "❌ Error: No se encontró el archivo '$ARCHIVO'"
    exit 1
fi

REGISTROS=$(( $(wc -l < "$ARCHIVO") - 1 ))
echo "Archivo: $ARCHIVO"
echo "Registros: $REGISTROS"

if [ "$REGISTROS" -lt 100 ]; then
    echo "⚠️  ADVERTENCIA: Menos de 100 registros. Verificar ingesta."
else
    echo "✅ Cantidad de registros OK."
fi
```
</details>

---

### Ejercicio 3 — Automatización con Cron
Escribe la expresión de crontab para los siguientes requisitos:
1. Ejecutar `ingesta.sh` de lunes a viernes a las 2:30 AM.
2. Ejecutar `reporte_semanal.sh` los viernes a las 5 PM.
3. Ejecutar `limpieza_tmp.sh` el último minuto de cada hora.

<details>
<summary>Ver solución</summary>

```bash
# 1. Lunes a viernes a las 2:30 AM
30 2 * * 1-5 /home/matias/scripts/ingesta.sh >> /var/log/ingesta.log 2>&1

# 2. Los viernes a las 5:00 PM
0 17 * * 5 /home/matias/scripts/reporte_semanal.sh >> /var/log/reportes.log 2>&1

# 3. El último minuto de cada hora (minuto 59)
59 * * * * /home/matias/scripts/limpieza_tmp.sh
```
</details>

---

[← Clase anterior: Entorno de Desarrollo e IA](../clase-01-entorno-de-desarrollo-e-ia/README.md) | [Volver al módulo →](../README.md)

# Clase 03: Creando un proyecto con uv

## Objetivos de aprendizaje

- Crear un proyecto nuevo de Python con uv
- Entender la estructura de archivos que genera uv
- Ejecutar un script de Python dentro del proyecto

---

## Crear un proyecto nuevo

Para crear un proyecto de Python, abre una terminal, navega a la carpeta donde quieras trabajar y ejecuta:

```bash
uv init mi-primer-proyecto
```

Esto crea una carpeta llamada `mi-primer-proyecto` con todo lo necesario para empezar a trabajar.

Ahora entra en la carpeta del proyecto:

```bash
cd mi-primer-proyecto
```

## Estructura del proyecto

Veamos qué archivos creó uv:

```
mi-primer-proyecto/
├── .python-version      ← Versión de Python que usa el proyecto
├── main.py              ← Archivo principal de Python
├── pyproject.toml       ← Configuración del proyecto (el archivo más importante)
└── README.md            ← Descripción del proyecto
```

> Al ejecutar tu primer comando `uv run`, uv también creará automáticamente la carpeta `.venv/` (el ambiente virtual) y el archivo `uv.lock`.

Veamos cada archivo en detalle.

### `pyproject.toml` — La ficha técnica del proyecto

Este es el archivo más importante. Contiene toda la información sobre tu proyecto:

```toml
[project]
name = "mi-primer-proyecto"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = []
```

¿Qué significa cada parte?

| Campo | Descripción |
|-------|-------------|
| `name` | Nombre del proyecto |
| `version` | Versión actual |
| `description` | Descripción breve |
| `requires-python` | Versión mínima de Python necesaria |
| `dependencies` | Lista de paquetes que el proyecto necesita (por ahora vacía) |

### Analogía

Piensa en `pyproject.toml` como la **etiqueta de un equipo de laboratorio**: indica el nombre del equipo, su modelo, para qué sirve y qué accesorios necesita para funcionar. Si alguien más necesita replicar tu configuración, solo tiene que leer esta etiqueta.

### `.python-version` — La versión de Python

Este archivo contiene simplemente el número de versión de Python que usa tu proyecto, por ejemplo:

```
3.12
```

Lo mejor es que **no necesitas instalar Python manualmente**. Cuando uv detecta que no tienes la versión indicada, la descarga e instala automáticamente.

### `main.py` — Tu primer archivo de código

uv genera un archivo de ejemplo:

```python
def main():
    print("Hello from mi-primer-proyecto!")


if __name__ == "__main__":
    main()
```

## Ejecutar un script

Para ejecutar código dentro del proyecto, usamos `uv run`:

```bash
uv run main.py
```

La primera vez que ejecutes este comando, uv:

1. Descarga la versión de Python indicada en `.python-version` (si no la tienes)
2. Crea el ambiente virtual en la carpeta `.venv/`
3. Instala las dependencias listadas en `pyproject.toml`
4. Ejecuta el script

Deberías ver en la terminal:

```
Hello from mi-primer-proyecto!
```

> **Importante**: Siempre usa `uv run` para ejecutar tus scripts en lugar de `python script.py`. De esta forma, uv se asegura de que el código se ejecute dentro del ambiente virtual correcto con todas las dependencias instaladas.

Las siguientes veces que ejecutes `uv run`, será casi instantáneo porque el ambiente ya existe.

## Abrir el proyecto en VS Code

Para abrir tu proyecto en VS Code desde la terminal:

```bash
code .
```

### Seleccionar el intérprete de Python

VS Code necesita saber cuál Python usar. Para que use el del ambiente virtual:

1. Abre la paleta de comandos con `Ctrl + Shift + P`
2. Escribe **"Python: Select Interpreter"**
3. Selecciona la opción que muestra la ruta `.venv` de tu proyecto, por ejemplo:
   ```
   Python 3.12.x ('.venv')
   ```

> Si no ves la opción de `.venv`, asegúrate de haber ejecutado `uv run` al menos una vez (esto crea el ambiente virtual).

De esta manera, VS Code usará el mismo ambiente virtual que uv, y las funciones como autocompletado e inspección de código funcionarán correctamente.

---

## Ejercicios prácticos

1. **Crea un proyecto nuevo** llamado `prueba-uv` usando `uv init prueba-uv`
2. **Explora los archivos**: Abre cada archivo generado y lee su contenido
3. **Ejecuta el script**: Usa `uv run main.py` y verifica que funcione
4. **Modifica el código**: Cambia el mensaje en `main.py` por algo personalizado y vuelve a ejecutar
5. **Abre en VS Code**: Ejecuta `code .` y selecciona el intérprete del `.venv`

---

[← Clase anterior: Introducción a uv](../clase-02-introduccion-a-uv/README.md) | [Siguiente clase: Gestión de dependencias →](../clase-04-gestion-de-dependencias/README.md)

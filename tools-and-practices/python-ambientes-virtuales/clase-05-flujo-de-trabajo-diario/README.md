# Clase 05: Flujo de trabajo diario

## Objetivos de aprendizaje

- Ejecutar scripts y notebooks con uv en el día a día
- Saber qué archivos incluir en Git y cuáles ignorar
- Conocer el flujo completo de trabajo con uv

---

## Ejecutar scripts con `uv run`

Como vimos en clases anteriores, `uv run` es el comando que usarás constantemente. Ejecuta cualquier comando dentro del ambiente virtual de tu proyecto:

```bash
uv run python mi_script.py
```

También puedes ejecutar cualquier herramienta instalada como dependencia. Por ejemplo:

```bash
uv run pytest                  # ejecutar pruebas automatizadas
uv run ruff check .            # revisar el estilo del código
```

> **Nota**: Herramientas como `pytest` y `ruff` son comunes en el mundo del desarrollo de software, pero no necesitas conocerlas para trabajar con uv. Lo importante aquí es el patrón: cualquier comando que dependa de los paquetes de tu proyecto se ejecuta con `uv run` adelante. En nuestro caso, lo más habitual será `uv run python mi_script.py` o `uv run jupyter lab`.

## Trabajar con Jupyter Notebooks

Para usar notebooks en tu proyecto, primero agrega Jupyter como dependencia de desarrollo:

```bash
uv add --dev jupyterlab
```

Luego, inicia Jupyter Lab:

```bash
uv run jupyter lab
```

Esto abrirá Jupyter Lab en tu navegador. Todos los paquetes de tu proyecto (pandas, numpy, etc.) estarán disponibles dentro de los notebooks.

> **Tip**: Agregar Jupyter como dependencia de desarrollo (`--dev`) en lugar de dependencia normal indica que es una herramienta de trabajo, no un requisito del proyecto en sí.

## Integración con Git

### Qué archivos incluir en Git

Cuando trabajas con un equipo, es importante que ciertos archivos se compartan a través de Git y otros no.

| Archivo | ¿Incluir en Git? | ¿Por qué? |
|---------|:-----------------:|------------|
| `pyproject.toml` | **Sí** | Define las dependencias y configuración del proyecto |
| `uv.lock` | **Sí** | Garantiza que todos usen las mismas versiones exactas |
| `.python-version` | **Sí** | Indica qué versión de Python usar |
| `main.py` y otros `.py` | **Sí** | Es el código fuente del proyecto |
| `.venv/` | **No** | Es una carpeta local que se puede recrear con `uv sync` |

### Verificar el `.gitignore`

Tu proyecto debe tener un archivo `.gitignore` que excluya la carpeta del ambiente virtual. Verifica que contenga al menos estas líneas:

```
# Ambiente virtual
.venv/
```

> Si usaste `uv init` para crear el proyecto, uv ya incluye `.venv` en el `.gitignore` automáticamente.

## Flujo completo de trabajo

### Empezar un proyecto nuevo

```
uv init mi-proyecto          # 1. Crear el proyecto
cd mi-proyecto               # 2. Entrar al proyecto
uv add pandas numpy          # 3. Agregar dependencias
code .                       # 4. Abrir en VS Code
uv run main.py               # 5. Ejecutar código
```

### Unirse a un proyecto existente del equipo

```
git clone <url-del-repo>     # 1. Clonar el repositorio
cd nombre-del-proyecto       # 2. Entrar al proyecto
uv sync                      # 3. Instalar dependencias (crea .venv automáticamente)
code .                       # 4. Abrir en VS Code
uv run main.py               # 5. Ejecutar código
```

### Diagrama del flujo diario

```
┌─────────────────────────────────────────────────┐
│                 Flujo diario                    │
│                                                 │
│   git pull                                      │
│     │        (traer cambios del equipo)         │
│     ▼                                           │
│   uv sync                                       │
│     │        (actualizar dependencias si hubo   │
│     │         cambios en pyproject.toml)        │
│     ▼                                           │
│   Escribir código / trabajar en notebooks       │
│     │                                           │
│     ▼                                           │
│   uv run script.py                              │
│     │        (ejecutar y probar)                │
│     ▼                                           │
│   git add, commit, push                         │
│              (compartir tu trabajo)             │
└─────────────────────────────────────────────────┘
```

## Buenas prácticas del equipo

1. **Siempre usa `uv run`** para ejecutar código. No uses `python script.py` directamente, ya que podría usar un Python diferente al del proyecto
2. **Haz `uv sync` después de `git pull`** si ves cambios en `pyproject.toml` o `uv.lock`. Esto asegura que tu ambiente local esté actualizado
3. **Incluye `uv.lock` en los commits**. Esto permite que todos en el equipo trabajen con las mismas versiones exactas
4. **No incluyas `.venv/` en Git**. Cada persona recrea su ambiente local con `uv sync`
5. **Usa dependencias de desarrollo** (`--dev`) para herramientas como Jupyter, pytest o ruff que no son parte del producto final

## Resumen de comandos

| Comando | Uso |
|---------|-----|
| `uv init nombre` | Crear un proyecto nuevo |
| `uv add paquete` | Agregar una dependencia |
| `uv add --dev paquete` | Agregar dependencia de desarrollo |
| `uv remove paquete` | Quitar una dependencia |
| `uv run script.py` | Ejecutar un script |
| `uv run jupyter lab` | Abrir Jupyter Lab |
| `uv sync` | Instalar/actualizar dependencias del proyecto |
| `uv python list` | Ver versiones de Python disponibles |

---

## Ejercicios prácticos

1. **Crea un proyecto completo**: Usa `uv init analisis-datos`, agrega pandas y matplotlib, y crea un script que lea un archivo CSV y genere un gráfico simple
2. **Trabaja con notebooks**: Agrega jupyterlab como dependencia de desarrollo, inicia Jupyter Lab y crea un notebook que importe pandas
3. **Verifica el `.gitignore`**: Abre el archivo `.gitignore` de tu proyecto y confirma que `.venv/` está excluido
4. **Simula el flujo de equipo**: Inicializa Git en tu proyecto (`git init`), haz un commit inicial con todos los archivos (excepto `.venv/`), borra la carpeta `.venv/`, y recrea el ambiente con `uv sync`
5. **Practica el flujo diario**: Agrega una nueva dependencia con `uv add`, escribe un script que la use, ejecútalo con `uv run` y haz un commit con los cambios

---

[← Clase anterior: Gestión de dependencias](../clase-04-gestion-de-dependencias/README.md) | [Volver al módulo →](../README.md)

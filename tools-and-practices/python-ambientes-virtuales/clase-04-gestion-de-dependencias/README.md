# Clase 04: Gestión de dependencias

## Objetivos de aprendizaje

- Agregar y quitar paquetes de un proyecto
- Entender la diferencia entre `pyproject.toml` y `uv.lock`
- Sincronizar un ambiente virtual a partir de los archivos del proyecto

---

## Agregar una dependencia

Para agregar un paquete a tu proyecto, usa `uv add` seguido del nombre del paquete:

```bash
uv add pandas
```

Este comando hace tres cosas:

1. Descarga e instala **pandas** (y sus propias dependencias) en el ambiente virtual
2. Agrega `pandas` a la lista de `dependencies` en `pyproject.toml`
3. Actualiza el archivo `uv.lock` con las versiones exactas instaladas

Puedes verificar que se instaló correctamente:

```bash
uv run python -c "import pandas; print(pandas.__version__)"
```

### Agregar varios paquetes a la vez

```bash
uv add numpy matplotlib
```

## ¿Qué cambió en el proyecto?

### Cambios en `pyproject.toml`

Después de agregar pandas, numpy y matplotlib, la sección de dependencias se ve así:

```toml
dependencies = [
    "matplotlib>=3.9",
    "numpy>=2.1",
    "pandas>=2.2",
]
```

Nota que las versiones usan `>=` (mayor o igual). Esto indica la versión **mínima** compatible, dando flexibilidad para actualizaciones menores.

### El archivo `uv.lock`

Este archivo se genera automáticamente y contiene las **versiones exactas** de cada paquete y todas sus dependencias internas. No necesitas editarlo nunca.

### Analogía

Piensa en estos dos archivos como una receta de cocina:

- **`pyproject.toml`** es la **receta**: dice "necesitas harina, huevos y azúcar"
- **`uv.lock`** es la **lista de compras exacta**: dice "harina marca X de 1kg, 6 huevos tamaño L, azúcar refinada de 500g"

La receta te dice qué necesitas. La lista de compras garantiza que cualquier persona que siga la receta obtenga exactamente los mismos ingredientes.

> **¿Por qué importa?** Cuando un compañero de equipo clona tu proyecto, `uv.lock` asegura que instale exactamente las mismas versiones que tú usas. Esto evita el clásico problema de "en mi computador funciona pero en el tuyo no".

## Quitar una dependencia

Si ya no necesitas un paquete:

```bash
uv remove matplotlib
```

Esto lo elimina de `pyproject.toml`, de `uv.lock` y del ambiente virtual.

## Sincronizar un ambiente virtual

El comando `uv sync` lee los archivos del proyecto (`pyproject.toml` y `uv.lock`) y se asegura de que el ambiente virtual tenga exactamente los paquetes indicados:

```bash
uv sync
```

### ¿Cuándo usar `uv sync`?

El caso más común es cuando **clonas un proyecto existente** del equipo:

```bash
git clone https://github.com/tu-equipo/proyecto-analisis.git
cd proyecto-analisis
uv sync
```

Con estos tres comandos ya tienes el proyecto listo para trabajar: uv crea el ambiente virtual, descarga la versión correcta de Python e instala todas las dependencias.

> **Recuerda**: No necesitas ejecutar `uv sync` después de `uv add` o `uv remove`, ya que esos comandos actualizan el ambiente automáticamente. `uv sync` es para cuando el ambiente no existe o quedó desactualizado (por ejemplo, después de un `git pull` que trajo cambios en las dependencias).

## Dependencias de desarrollo

Algunos paquetes solo los necesitas mientras desarrollas (herramientas de pruebas, formateadores de código, etc.), pero no son necesarios para que tu proyecto funcione. Estos se llaman **dependencias de desarrollo** (*dev dependencies*).

Para agregarlas, usa la opción `--dev`:

```bash
uv add --dev pytest
uv add --dev ruff
```

En `pyproject.toml` aparecen en una sección separada:

```toml
[dependency-groups]
dev = [
    "pytest>=8.3",
    "ruff>=0.8",
]
```

Esto ayuda a distinguir entre lo que el proyecto necesita para funcionar y lo que solo necesitas tú como desarrollador.

---

## Ejercicios prácticos

1. **Agrega dependencias**: En tu proyecto `prueba-uv`, ejecuta `uv add pandas numpy`
2. **Verifica los cambios**: Abre `pyproject.toml` y observa cómo cambió la sección `dependencies`
3. **Prueba un paquete**: Ejecuta `uv run python -c "import pandas; print(pandas.__version__)"` para verificar que pandas funciona
4. **Quita una dependencia**: Ejecuta `uv remove numpy` y verifica que se eliminó de `pyproject.toml`
5. **Agrega una dependencia de desarrollo**: Ejecuta `uv add --dev pytest` y observa dónde aparece en `pyproject.toml`
6. **Simula un compañero de equipo**: Borra la carpeta `.venv` de tu proyecto, luego ejecuta `uv sync` y verifica que todo se reconstruye correctamente

---

[← Clase anterior: Creando un proyecto con uv](../clase-03-creando-un-proyecto-con-uv/README.md) | [Siguiente clase: Flujo de trabajo diario →](../clase-05-flujo-de-trabajo-diario/README.md)

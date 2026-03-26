# Clase 02: Introducción a uv

## Objetivos de aprendizaje

- Conocer qué es uv y qué problemas resuelve
- Entender por qué elegimos uv sobre otras alternativas
- Instalar uv en tu computador

---

## ¿Qué es uv?

**uv** es un gestor de paquetes y proyectos para Python (*package and project manager*). Es una herramienta que se encarga de:

- **Crear ambientes virtuales** automáticamente
- **Instalar paquetes** de forma rápida
- **Gestionar las versiones de Python** sin que tengas que instalarlas manualmente
- **Organizar tu proyecto** con archivos de configuración estándar

En pocas palabras, uv reemplaza varias herramientas que antes necesitabas por separado (`pip`, `venv`, `pyenv`, `pip-tools`) con **una sola herramienta** que hace todo.

### Analogía

Imagina que antes necesitabas una herramienta diferente para cada tarea del laboratorio: una para medir, otra para mezclar, otra para calentar, otra para enfriar. **uv** es como un equipo multifuncional que hace todas esas tareas por ti, de forma más rápida y sencilla.

## ¿Por qué elegimos uv?

Existen varias herramientas para gestionar proyectos de Python. Esta tabla resume las diferencias principales:

| Característica | pip + venv | conda | poetry | uv |
|---------------|-----------|-------|--------|-----|
| Velocidad de instalación | Lenta | Lenta | Media | Muy rápida |
| Crea ambientes virtuales | Manual (venv aparte) | Sí | Sí | Sí (automático) |
| Gestiona versiones de Python | No | Sí | No | Sí |
| Archivo de configuración estándar | No (requirements.txt) | No (environment.yml) | Sí (pyproject.toml) | Sí (pyproject.toml) |
| Facilidad de uso | Requiere varios pasos | Media | Media | Simple |
| Todo en una herramienta | No | Parcial | Parcial | Sí |

### ¿Por qué uv para nuestro equipo?

1. **Simplicidad**: Un solo comando para crear un proyecto, agregar paquetes y ejecutar código
2. **Velocidad**: Instala paquetes hasta 100 veces más rápido que pip. No más esperas largas
3. **Todo en uno**: No necesitas instalar ni aprender múltiples herramientas
4. **Gestión automática de Python**: uv descarga e instala la versión de Python que tu proyecto necesita. No tienes que buscarla ni instalarla manualmente
5. **Estándar**: Usa `pyproject.toml`, el formato de configuración moderno y oficial de Python

> **Para el equipo**: Elegimos uv porque reduce la complejidad. En lugar de aprender 3 o 4 herramientas diferentes, aprendemos una sola que cubre todo lo que necesitamos.

## Instalación de uv

### Windows

Abre **PowerShell** y ejecuta el siguiente comando:

```bash
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

> Si estás usando Git Bash o la terminal de VS Code con bash, puedes usar en su lugar:
> ```bash
> curl -LsSf https://astral.sh/uv/install.sh | sh
> ```

### Verificar la instalación

Cierra y vuelve a abrir tu terminal, luego ejecuta:

```bash
uv --version
```

Deberías ver algo como: `uv 0.6.x`

> Si el comando no es reconocido, cierra completamente VS Code (o tu terminal) y ábrelo de nuevo. La instalación agrega uv al PATH del sistema, pero las terminales abiertas antes de la instalación no lo detectan.

## Comandos principales de uv (vista general)

No necesitas memorizarlos ahora. Los iremos aprendiendo uno por uno en las siguientes clases.

| Comando | ¿Qué hace? |
|---------|-------------|
| `uv init` | Crea un proyecto nuevo de Python |
| `uv add` | Agrega un paquete al proyecto |
| `uv remove` | Quita un paquete del proyecto |
| `uv run` | Ejecuta un script dentro del ambiente virtual |
| `uv sync` | Instala todas las dependencias del proyecto |
| `uv python list` | Muestra las versiones de Python disponibles |

> **Tip**: Puedes ver la ayuda de cualquier comando ejecutando `uv help <comando>`, por ejemplo: `uv help init`.

---

## Ejercicios prácticos

1. **Instala uv** en tu computador siguiendo las instrucciones de arriba
2. **Verifica la instalación** ejecutando `uv --version` en tu terminal
3. **Explora la ayuda** ejecutando `uv help` para ver todos los comandos disponibles
4. **Consulta un comando**: Ejecuta `uv help init` y lee la descripción de lo que hace
5. **Verifica las versiones de Python**: Ejecuta `uv python list` para ver qué versiones de Python están disponibles en tu sistema

---

[← Clase anterior: ¿Qué son los ambientes virtuales?](../clase-01-que-son-los-ambientes-virtuales/README.md) | [Siguiente clase: Creando un proyecto con uv →](../clase-03-creando-un-proyecto-con-uv/README.md)

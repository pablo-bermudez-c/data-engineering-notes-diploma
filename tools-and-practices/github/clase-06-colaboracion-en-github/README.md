# Clase 06: Colaboración en GitHub

## Objetivos de aprendizaje

- Usar Issues para organizar el trabajo del equipo
- Aplicar buenas prácticas de *code review*
- Conocer GitHub Projects para gestión de tareas
- Definir convenciones del equipo

---

## Issues: organizar el trabajo

Un **Issue** es un registro de una tarea, error (*bug*), o solicitud de funcionalidad (*feature request*). Es la forma principal de organizar el trabajo en GitHub.

### Crear un Issue

1. Ve al repositorio en GitHub
2. Haz clic en **"Issues"** → **"New issue"**
3. Escribe un título claro y una descripción detallada
4. (Opcional) Asigna el issue a una persona, agrega etiquetas (*labels*) y vincúlalo a un proyecto

### Estructura de un buen Issue

```markdown
## Descripción
Explicación clara del problema o tarea.

## Contexto
¿Por qué es necesario? ¿Qué impacto tiene?

## Pasos para reproducir (si es un bug)
1. Paso 1
2. Paso 2
3. Resultado inesperado

## Resultado esperado
Qué debería pasar cuando se resuelva.
```

### Etiquetas útiles (*labels*)

| Etiqueta | Uso |
|----------|-----|
| `bug` | Error que necesita corrección |
| `enhancement` | Mejora o nueva funcionalidad |
| `documentation` | Cambios en documentación |
| `question` | Pregunta o solicitud de ayuda |
| `help wanted` | Se necesita ayuda de otros |

### Vincular Issues con Pull Requests

Cuando creas un PR que resuelve un Issue, puedes vincularlo escribiendo en la descripción del PR:

```
Closes #12
```

Cuando el PR se fusione, el Issue se cerrará automáticamente.

## Code Review: buenas prácticas

El *code review* (revisión de código) es el proceso donde un compañero revisa tus cambios antes de fusionarlos. Esto ayuda a:

- Encontrar errores antes de que lleguen a producción
- Compartir conocimiento dentro del equipo
- Mantener la calidad y consistencia del código

### Para quien crea el PR

- **Mantén los cambios pequeños**: PRs pequeños son más fáciles de revisar
- **Describe tus cambios**: Explica el "qué" y el "por qué"
- **Revisa tu propio código** antes de solicitar revisión
- **Responde a los comentarios** de forma constructiva

### Para quien revisa

- **Sé respetuoso y constructivo**: Ofrece sugerencias, no solo críticas
- **Enfócate en lo importante**: Lógica, errores, claridad
- **Haz preguntas**: Si algo no está claro, pregunta en lugar de asumir
- **Aprueba cuando esté listo**: No busques la perfección, busca que funcione correctamente

### Ejemplo de un buen comentario de revisión

```
❌ "Esto está mal"
✅ "¿Podríamos usar pandas.read_excel() aquí en lugar de xlrd directamente?
    Simplificaría el manejo de errores y es más consistente con el resto del código."
```

## GitHub Projects: gestión visual

**GitHub Projects** es una herramienta de gestión de tareas integrada en GitHub, similar a un tablero Kanban.

### Configurar un tablero básico

1. Ve al repositorio → pestaña **"Projects"** → **"New project"**
2. Elige la vista de **"Board"** (tablero)
3. Crea columnas como:

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Por hacer   │  │  En progreso │  │  En revisión │  │  Completado  │
│  (To Do)     │  │ (In Progress)│  │  (In Review) │  │   (Done)     │
│              │  │              │  │              │  │              │
│  Issue #3    │  │  Issue #5    │  │  PR #7       │  │  Issue #1    │
│  Issue #4    │  │              │  │              │  │  Issue #2    │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
```

4. Arrastra los Issues entre columnas a medida que avanza el trabajo

## Convenciones del equipo

Establecer convenciones ayuda a mantener el orden y la consistencia en el repositorio.

### Mensajes de commit

Usa el formato:

```
Verbo en imperativo + descripción breve

Ejemplos:
✅ "Agrega análisis del ciclo 5"
✅ "Corrige error en lectura de archivos Excel"
✅ "Actualiza README con instrucciones de instalación"
❌ "cambios varios"
❌ "wip"
❌ "asdfgh"
```

### Nombres de ramas

```
tipo/descripcion-breve

Tipos comunes:
- analisis/  → para nuevos análisis de datos
- fix/       → para corrección de errores
- docs/      → para documentación
- feature/   → para nuevas funcionalidades
```

### Nombres de archivos

- Usa **minúsculas** y **guiones bajos** (`_`) o **guiones** (`-`)
- Sé descriptivo: `analisis_ciclo_5.py` en lugar de `a5.py`
- Evita espacios y caracteres especiales en nombres de archivos

---

## Ejercicios prácticos

1. Crea un **Issue** en el repositorio describiendo una tarea pendiente del equipo
2. Asigna el Issue a ti mismo y agrégale una etiqueta (*label*)
3. Crea una rama, haz cambios que resuelvan el Issue, y abre un **Pull Request** que lo vincule con `Closes #N`
4. Revisa el Pull Request de un compañero dejando al menos un comentario constructivo
5. Explora la pestaña **Projects** y crea un tablero básico para el equipo

---

[← Clase anterior](../clase-05-flujo-de-trabajo-en-equipo/README.md) | [Volver al índice del módulo →](../README.md)

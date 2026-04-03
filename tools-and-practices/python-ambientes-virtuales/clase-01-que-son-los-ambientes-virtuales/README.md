# Clase 01: ¿Qué son los ambientes virtuales?

## Objetivos de aprendizaje

- Entender qué es una dependencia y por qué los proyectos las necesitan
- Comprender el problema de los conflictos entre proyectos
- Conocer qué es un ambiente virtual y cómo resuelve estos conflictos

---

## ¿Qué es una dependencia?

Cuando escribimos código en Python, rara vez hacemos todo desde cero. Usamos **paquetes** (*packages*) que otras personas han creado: herramientas para leer datos, hacer gráficos, calcular estadísticas, etc.

Estos paquetes que tu proyecto necesita para funcionar se llaman **dependencias** (*dependencies*).

### Ejemplo

Imagina un proyecto de análisis de datos. Podría necesitar:

- **pandas** — para leer y manipular tablas de datos
- **matplotlib** — para crear gráficos
- **numpy** — para cálculos numéricos

Sin estas dependencias instaladas, tu código no funciona. Son como los ingredientes de una receta: sin ellos, no puedes preparar el plato.

## El problema: conflictos entre proyectos

Python instala los paquetes en una ubicación compartida por defecto. Esto significa que todos tus proyectos usan los mismos paquetes y las mismas versiones. ¿Cuál es el problema?

### Ejemplo concreto

Supón que tienes dos proyectos:

```
Proyecto A (análisis de señales EMG)
  └── Necesita pandas versión 1.5

Proyecto B (dashboard de resultados)
  └── Necesita pandas versión 2.0
```

Si instalas pandas 2.0 para el Proyecto B, el Proyecto A podría dejar de funcionar porque el código fue escrito para una versión anterior. Y viceversa.

### Analogía

Piensa en cada proyecto como un **experimento de laboratorio**. No mezclarías los reactivos del experimento A con los del experimento B; cada experimento tiene su propia mesa de trabajo con sus propios materiales, organizados y etiquetados. Si compartieras todo en una sola mesa, corres el riesgo de contaminar un experimento con los materiales del otro.

### Sin ambientes virtuales vs. con ambientes virtuales

```
SIN ambientes virtuales:              CON ambientes virtuales:

┌─────────────────────────┐           ┌──────────────┐  ┌──────────────┐
│   Python del sistema    │           │ Proyecto A   │  │ Proyecto B   │
│                         │           │              │  │              │
│  pandas 2.0  (¿o 1.5?)  │           │ pandas 1.5   │  │ pandas 2.0   │
│  numpy 1.24             │           │ numpy 1.24   │  │ numpy 1.26   │
│  matplotlib 3.7         │           │ matplotlib   │  │ plotly       │
│                         │           │              │  │              │
│  Proyecto A ← conflicto │           │ ✓ Aislado    │  │ ✓ Aislado    │
│  Proyecto B ← conflicto │           └──────────────┘  └──────────────┘
└─────────────────────────┘
```

## ¿Qué es un ambiente virtual?

Un **ambiente virtual** (*virtual environment*) es una carpeta aislada que contiene su propia copia de Python y sus propios paquetes, independiente de cualquier otro proyecto.

Cuando creas un ambiente virtual para un proyecto:

- Los paquetes que instales solo existen dentro de ese ambiente
- Otros proyectos no se ven afectados
- Puedes tener diferentes versiones del mismo paquete en distintos proyectos
- Si algo sale mal, puedes borrar el ambiente y recrearlo sin afectar nada más

> **Resumen**: Un ambiente virtual es como darle a cada proyecto su propia mesa de laboratorio, con sus propios materiales y herramientas. Nada se mezcla entre proyectos.

## Herramientas para crear ambientes virtuales

Existen varias herramientas para gestionar ambientes virtuales en Python:

| Herramienta | Descripción |
|-------------|-------------|
| **venv** | Viene incluida con Python. Básica, solo crea el ambiente |
| **conda** | Popular en ciencia de datos. Gestiona paquetes y ambientes, pero puede ser lenta |
| **poetry** | Gestiona dependencias y ambientes. Más completa que venv |
| **uv** | Moderna, extremadamente rápida y todo-en-uno. La que usaremos en este módulo |

> En las siguientes clases aprenderemos a usar **uv**, que combina lo mejor de todas estas herramientas en una sola, de forma rápida y sencilla.

---

## Ejercicios prácticos

1. **Piensa en un proyecto**: Elige un proyecto de análisis de datos (real o imaginario) y haz una lista de 3 paquetes de Python que necesitaría
2. **Identifica un conflicto**: Imagina que tienes dos proyectos que usan versiones diferentes del mismo paquete. Describe qué problema podrías tener si no usaras ambientes virtuales
3. **Reflexiona**: ¿Alguna vez has tenido un error al ejecutar código de Python que funcionaba antes? ¿Podría haber sido un problema de versiones?

---

[Siguiente clase: Introducción a uv →](../clase-02-introduccion-a-uv/README.md)

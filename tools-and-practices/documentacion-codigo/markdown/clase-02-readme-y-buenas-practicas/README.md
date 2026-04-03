# Clase 02: README y buenas prácticas

## Objetivos de aprendizaje

- Entender la función de un archivo README en un proyecto.
- Conocer la estructura recomendada para un README profesional.
- Aplicar buenas prácticas de documentación para que otros (y tu yo futuro) puedan entender tu proyecto.

---

## ¿Qué es un README?

Un archivo `README.md` es la "portada" de un proyecto. Es lo primero que ve alguien cuando llega a un repositorio y responde la pregunta más importante: **¿de qué se trata esto y cómo lo uso?**

En GitHub y GitLab, el archivo `README.md` en la raíz del repositorio se muestra automáticamente debajo de los archivos, renderizado con formato.

### Analogía

Un README es como el manual de instrucciones de un electrodoméstico: si está bien escrito, cualquier persona puede entender qué hace el producto y cómo usarlo sin necesidad de llamar a soporte técnico. Si está mal escrito o no existe, la persona lo deja en la caja y busca otra cosa.

---

## ¿Para quién escribes el README?

Antes de escribir, ten en mente a tu audiencia. Un README puede ser leído por:

- **Tu yo del futuro**: dentro de seis meses, habrás olvidado el 80% de los detalles del proyecto.
- **Colaboradores**: personas que quieren contribuir y necesitan entender el contexto.
- **Usuarios**: personas que solo quieren *usar* lo que construiste.
- **Reclutadores o revisores**: personas que evalúan tu trabajo.

Un buen README sirve a todos estos públicos.

---

## Estructura recomendada

No existe una estructura universal obligatoria, pero la siguiente funciona bien para la mayoría de proyectos técnicos:

```
# Nombre del proyecto

Descripción breve (1-3 oraciones)

## Tabla de contenidos (opcional, para READMEs largos)

## Requisitos previos

## Instalación

## Uso

## Estructura del proyecto (opcional)

## Contribuir (opcional)

## Licencia
```

---

## Cada sección explicada

### 1. Título y descripción

El título debe ser el nombre del proyecto. La descripción debe responder en pocas oraciones:
- ¿Qué hace?
- ¿Para quién es?
- ¿Qué problema resuelve?

```markdown
# analizador-ventas

Herramienta para analizar datos de ventas desde archivos CSV y generar
reportes automáticos en Excel. Diseñada para equipos de negocio sin
experiencia en programación.
```

> **Evita**: descripciones vagas como "este proyecto es un sistema para hacer cosas". Sé específico desde el primer momento.

---

### 2. Requisitos previos (*prerequisites*)

Lista todo lo que la persona necesita tener instalado o configurado antes de poder usar el proyecto.

```markdown
## Requisitos previos

- Python 3.11 o superior
- [uv](https://docs.astral.sh/uv/) para manejo de dependencias
- Acceso a los archivos CSV de ventas (formato explicado en `/docs`)
```

---

### 3. Instalación

Instrucciones paso a paso, numeradas, con comandos exactos en bloques de código.

```markdown
## Instalación

1. Clona el repositorio:

   ```bash
   git clone https://github.com/usuario/analizador-ventas.git
   cd analizador-ventas
   ```

2. Crea el ambiente virtual e instala dependencias:

   ```bash
   uv sync
   ```

3. Activa el ambiente virtual:

   ```bash
   source .venv/bin/activate  # Linux / macOS
   .venv\Scripts\activate     # Windows
   ```
```

---

### 4. Uso (*usage*)

Muestra cómo se usa el proyecto con ejemplos concretos. Si hay múltiples formas de uso, cubra las más comunes.

```markdown
## Uso

Ejecuta el analizador pasando la ruta al archivo CSV:

```bash
python main.py --input datos/ventas_enero.csv
```

El reporte se genera en la carpeta `output/` con la fecha actual en el nombre.
```

---

### 5. Estructura del proyecto (opcional pero recomendado)

Ayuda a cualquier persona a orientarse rápidamente en el repositorio.

```markdown
## Estructura del proyecto

```
analizador-ventas/
├── main.py              # Punto de entrada principal
├── src/
│   ├── lector.py        # Lee y valida archivos CSV
│   └── reportes.py      # Genera los reportes Excel
├── datos/               # Archivos CSV de ejemplo
├── output/              # Reportes generados (no se incluye en git)
└── README.md
```
```

---

### 6. Insignias (*badges*) — elemento visual opcional

Las insignias son pequeñas imágenes que muestran información del estado del proyecto: versión de Python, licencia, estado de los tests, etc. Se ven así:

```markdown
![Python](https://img.shields.io/badge/python-3.11+-blue)
![Licencia](https://img.shields.io/badge/licencia-MIT-green)
```

Se ubican justo después del título, antes de la descripción.

---

## Buenas prácticas de escritura

### Sé directo

Escribe en segunda persona y con oraciones cortas.

| En vez de... | Escribe... |
|---|---|
| "El sistema permite al usuario..." | "Usa este comando para..." |
| "Es posible configurar..." | "Para configurar X, edita..." |
| "Este proyecto fue creado con el objetivo de..." | "Esta herramienta analiza..." |

### Mantén el README actualizado

Un README desactualizado es peor que no tener README. Actualízalo cada vez que cambies algo importante (comandos de instalación, nuevas dependencias, cambios en la API).

### Usa ejemplos reales

Los comandos y ejemplos de código deben funcionar. Pruébalos antes de publicarlos.

### No asumas conocimiento

Explica siglas y términos técnicos cuando los uses por primera vez. No todos tienen el mismo contexto que tú.

### El README no es el único documento

Para proyectos grandes, el README debe ser una puerta de entrada, no un manual completo. Usa carpetas como `/docs` para documentación más detallada y enlaza desde el README.

---

## Errores comunes

| Error | Por qué es un problema | Solución |
|-------|------------------------|----------|
| README vacío o con solo el nombre | No da ninguna información útil | Agrega descripción e instrucciones básicas |
| Instrucciones de instalación rotas | La persona no puede usar el proyecto | Prueba las instrucciones en una máquina limpia |
| Sin sección de requisitos | La persona instala y falla sin saber por qué | Lista todos los requisitos explícitamente |
| Comandos sin bloques de código | Difícil de copiar y pegar | Siempre usa ` ```bash ... ``` ` para comandos |
| README en inglés cuando el equipo es hispanohablante | Barrera de entrada innecesaria | Escribe en el idioma de tu audiencia principal |

---

## Ejercicios prácticos

1. **Evalúa un README existente**: Busca en GitHub un proyecto que uses o conozcas. Lee su README y responde: ¿Entiendes qué hace el proyecto? ¿Puedes instalarlo y usarlo solo con esa información? ¿Qué agregarías o cambiarías?

2. **Escribe el README de un proyecto tuyo**: Toma cualquier proyecto o script que hayas creado (aunque sea pequeño) y escríbele un README siguiendo la estructura de esta clase. Mínimo: título, descripción, requisitos e instrucciones de uso.

3. **README para un proyecto imaginario**: Si no tienes proyectos propios aún, crea el README de un proyecto ficticio. Por ejemplo, una herramienta que descarga el menú del casino universitario cada día. Sé específico y concreto.

4. **Revisión entre pares**: Si trabajas con alguien más, intercambien READMEs e intenten seguir las instrucciones del otro. Anoten qué información faltó o estaba confusa.

5. **Agrega un README al repositorio de este curso**: Crea un `README.md` personal en una carpeta de práctica que describa qué estás aprendiendo, qué herramientas usas y cómo está organizado tu espacio de trabajo.

---

[Siguiente clase: Mermaid — Introducción y diagramas de flujo →](../../mermaid/clase-01-introduccion-y-diagramas-de-flujo/README.md)

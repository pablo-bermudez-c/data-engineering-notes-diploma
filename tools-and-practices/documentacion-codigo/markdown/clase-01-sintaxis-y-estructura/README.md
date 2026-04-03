# Clase 01: Sintaxis y estructura de Markdown

## Objetivos de aprendizaje

- Entender qué es Markdown y por qué se usa en proyectos técnicos.
- Conocer y aplicar los elementos fundamentales: encabezados, listas, énfasis, código, links e imágenes.
- Ser capaz de leer y escribir un documento Markdown básico.

---

## ¿Qué es Markdown?

Markdown es una forma de escribir texto con formato usando solo caracteres del teclado. En lugar de hacer clic en botones como en Word, escribes símbolos que indican cómo debe verse el texto.

### Analogía

Piensa en Markdown como una receta escrita a mano: no tiene colores ni tipografías especiales, pero tiene una estructura clara que cualquiera puede leer. Cuando se "cocina" (renderiza), el resultado es un documento con formato visual limpio.

Por ejemplo, este texto en Markdown:

```
## Ingredientes
- 2 tazas de harina
- **1 cucharada** de sal
```

Se ve así cuando se renderiza:

> ## Ingredientes
> - 2 tazas de harina
> - **1 cucharada** de sal

---

## ¿Dónde se usa Markdown?

| Plataforma | Uso |
|------------|-----|
| GitHub / GitLab | READMEs, issues, pull requests, wikis |
| Jupyter Notebooks | Celdas de texto y explicaciones |
| Notion | Documentos y bases de conocimiento |
| VS Code | Archivos `.md` con vista previa |
| Slack / Discord | Formato en mensajes |

---

## Elementos fundamentales

### 1. Encabezados (*headings*)

Se crean con el símbolo `#`. La cantidad de `#` define el nivel del encabezado.

```markdown
# Título principal (H1)
## Sección (H2)
### Subsección (H3)
#### Sub-subsección (H4)
```

> **Regla práctica**: usa H1 solo una vez por documento (el título). Usa H2 para secciones principales y H3 para subsecciones.

---

### 2. Párrafos y saltos de línea

Un párrafo es simplemente texto seguido. Para crear un nuevo párrafo, deja una línea en blanco entre ellos.

```markdown
Este es el primer párrafo. Puede tener varias oraciones.

Este es el segundo párrafo. Está separado por una línea vacía.
```

---

### 3. Énfasis (*emphasis*)

```markdown
**texto en negrita**
*texto en cursiva*
~~texto tachado~~
```

Se ve así:
- **texto en negrita**
- *texto en cursiva*
- ~~texto tachado~~

---

### 4. Listas

**Lista no ordenada** (con viñetas):

```markdown
- Elemento uno
- Elemento dos
  - Sub-elemento (indentado con 2 espacios)
- Elemento tres
```

**Lista ordenada** (numerada):

```markdown
1. Primer paso
2. Segundo paso
3. Tercer paso
```

> Las listas ordenadas siempre se numeran correctamente aunque escribas `1.` en todos los elementos — Markdown se encarga de la numeración.

---

### 5. Código (*code*)

Para texto en línea (*inline code*), usa una comilla invertida `` ` ``:

```markdown
El comando `git status` muestra el estado del repositorio.
```

Se ve así: El comando `git status` muestra el estado del repositorio.

Para bloques de código (*code blocks*), usa tres comillas invertidas ` ``` ` seguidas del lenguaje:

````markdown
```python
nombre = "mundo"
print(f"Hola, {nombre}!")
```
````

Se ve así:

```python
nombre = "mundo"
print(f"Hola, {nombre}!")
```

> Especificar el lenguaje después de las comillas activa el **resaltado de sintaxis** (*syntax highlighting*).

---

### 6. Links (hipervínculos)

```markdown
[texto visible](URL)

# Ejemplos:
[GitHub](https://github.com)
[Volver al inicio](../README.md)
```

El texto entre corchetes `[]` es lo que ve el usuario. La URL entre paréntesis `()` puede ser una dirección web o una ruta relativa a otro archivo.

---

### 7. Imágenes

La sintaxis es casi igual a los links, pero con un `!` al inicio:

```markdown
![texto alternativo](ruta/a/la/imagen.png)
```

El texto alternativo (*alt text*) aparece si la imagen no carga, y es importante para accesibilidad.

---

### 8. Citas (*blockquotes*)

```markdown
> Esta es una cita o nota destacada.
> Puede tener múltiples líneas.
```

Se ve así:

> Esta es una cita o nota destacada.
> Puede tener múltiples líneas.

---

### 9. Líneas horizontales (*horizontal rules*)

Se crean con tres guiones `---`:

```markdown
Sección uno

---

Sección dos
```

---

### 10. Tablas

```markdown
| Columna A | Columna B | Columna C |
|-----------|-----------|-----------|
| Valor 1   | Valor 2   | Valor 3   |
| Valor 4   | Valor 5   | Valor 6   |
```

Se ve así:

| Columna A | Columna B | Columna C |
|-----------|-----------|-----------|
| Valor 1   | Valor 2   | Valor 3   |
| Valor 4   | Valor 5   | Valor 6   |

> La segunda fila de guiones `---` es obligatoria y define que la primera fila es el encabezado.

---

## Referencia rápida

| Elemento | Sintaxis |
|----------|----------|
| Encabezado H2 | `## Título` |
| Negrita | `**texto**` |
| Cursiva | `*texto*` |
| Código en línea | `` `código` `` |
| Bloque de código | ` ```lenguaje ... ``` ` |
| Lista viñetas | `- elemento` |
| Lista numerada | `1. elemento` |
| Link | `[texto](url)` |
| Imagen | `![alt](ruta)` |
| Cita | `> texto` |
| Línea horizontal | `---` |

---

## Ejercicios prácticos

1. **Crea tu primer documento**: Abre VS Code, crea un archivo llamado `prueba.md` y escribe un texto con al menos un encabezado H1, un H2, un párrafo, una lista y una palabra en negrita. Usa la vista previa de VS Code (`Ctrl+Shift+V`) para ver el resultado.

2. **Documenta una receta o proceso**: Escribe en Markdown los pasos para preparar tu comida favorita o para hacer alguna tarea cotidiana. Usa lista numerada para los pasos y negrita para resaltar ingredientes o elementos clave.

3. **Practica con código**: Escribe un bloque de código en cualquier lenguaje que conozcas (puede ser pseudocódigo). Asegúrate de especificar el lenguaje para activar el resaltado de sintaxis.

4. **Crea una tabla de comparación**: Elige dos herramientas, lenguajes o conceptos que conozcas y crea una tabla Markdown comparándolos en al menos 3 criterios.

5. **Explora Markdown en GitHub**: Abre cualquier repositorio público en GitHub y haz clic en el archivo `README.md`. Luego haz clic en el botón "Raw" para ver el Markdown sin renderizar. Compara el texto original con el resultado visual.

---

[Siguiente clase: README y buenas prácticas →](../clase-02-readme-y-buenas-practicas/README.md)

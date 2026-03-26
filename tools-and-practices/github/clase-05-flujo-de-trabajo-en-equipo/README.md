# Clase 05: Flujo de trabajo en equipo

## Objetivos de aprendizaje

- Entender cómo sincronizar tu trabajo con el repositorio remoto
- Aprender el modelo de ramas (*branching model*) para el equipo
- Crear y gestionar Pull Requests (*PR*)
- Revisar y aprobar cambios de otros

---

## Sincronizar con el repositorio remoto

### `git push` — Enviar cambios

Después de hacer *commits* locales, necesitas enviarlos a GitHub para que el equipo pueda verlos:

```bash
# Enviar cambios de la rama actual
git push

# Si es la primera vez que envías una rama nueva
git push -u origin nombre-de-la-rama
```

### `git pull` — Recibir cambios

Para descargar los cambios que otros miembros del equipo han subido:

```bash
git pull
```

> **Buena práctica**: Siempre haz `git pull` antes de empezar a trabajar para asegurarte de tener la versión más reciente.

### `git fetch` — Ver cambios sin aplicarlos

Si quieres ver qué cambios hay en el remoto sin aplicarlos todavía:

```bash
git fetch
```

## Modelo de ramas para el equipo

Usaremos un flujo simple basado en **ramas por tarea** (*feature branches*):

```
main ●────────●────────────●────────●──── (rama estable)
      \      /    \       /
       ●────●      ●────●
     analisis    fix-datos
     ciclo-5
```

### Reglas del flujo

1. **`main`** es la rama estable. No se trabaja directamente en ella.
2. Para cada tarea, **crea una rama nueva** desde `main`.
3. Trabaja en tu rama, haciendo *commits* frecuentes.
4. Cuando termines, **crea un Pull Request** para fusionar tu rama en `main`.
5. Un compañero **revisa y aprueba** el Pull Request.
6. Se fusiona (*merge*) el Pull Request en `main`.

### Convención para nombres de ramas

```
tipo/descripcion-breve
```

Ejemplos:
- `analisis/ciclo-5-datos`
- `fix/error-lectura-excel`
- `docs/actualizar-readme`

## Pull Requests (*PR*)

Un **Pull Request** es una solicitud para fusionar los cambios de tu rama en otra rama (generalmente `main`). Es el mecanismo principal para revisar código en equipo.

### Crear un Pull Request

#### Desde GitHub (navegador)

1. Ve al repositorio en GitHub
2. Haz clic en **"Pull requests"** → **"New pull request"**
3. Selecciona tu rama como *compare* y `main` como *base*
4. Escribe un título descriptivo y una descripción de los cambios
5. Haz clic en **"Create pull request"**

#### Desde VS Code (con la extensión GitHub Pull Requests)

1. Abre la paleta de comandos (`Ctrl+Shift+P`)
2. Busca **"GitHub Pull Requests: Create Pull Request"**
3. Completa el título y la descripción
4. Confirma la creación

### Estructura de un buen Pull Request

```markdown
## Descripción
Breve explicación de qué cambia y por qué.

## Cambios realizados
- Cambio 1
- Cambio 2

## Cómo probar
Pasos para verificar que los cambios funcionan correctamente.
```

## Revisar un Pull Request

Cuando un compañero crea un PR, puedes revisarlo:

1. Ve al PR en GitHub
2. Haz clic en la pestaña **"Files changed"**
3. Revisa los cambios línea por línea
4. Si tienes comentarios, haz clic en la línea y escribe tu comentario
5. Cuando termines, haz clic en **"Review changes"**:
   - **Comment**: solo dejar comentarios
   - **Approve**: aprobar los cambios
   - **Request changes**: solicitar modificaciones

### Flujo completo: ejemplo paso a paso

```bash
# 1. Actualizar main
git switch main
git pull

# 2. Crear rama de trabajo
git switch -c analisis/ciclo-5-datos

# 3. Trabajar y hacer commits
# ... editar archivos ...
git add .
git commit -m "Agrega análisis del ciclo 5"

# 4. Enviar la rama a GitHub
git push -u origin analisis/ciclo-5-datos

# 5. Crear Pull Request en GitHub (navegador o VS Code)

# 6. Esperar revisión y aprobación

# 7. Fusionar el PR en GitHub

# 8. Actualizar main local
git switch main
git pull
```

---

## Ejercicios prácticos

1. Crea una rama nueva con el formato `docs/tu-nombre`
2. Agrega un archivo con tu nombre en la carpeta del equipo
3. Haz *commit* y *push* de tu rama
4. Crea un **Pull Request** en GitHub
5. Pide a un compañero que revise y apruebe tu PR
6. Fusiona el PR y actualiza tu `main` local

---

[← Clase anterior](../clase-04-github-en-vscode/README.md) | [Siguiente clase: Colaboración en GitHub →](../clase-06-colaboracion-en-github/README.md)

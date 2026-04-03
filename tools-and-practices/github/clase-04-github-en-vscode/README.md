# Clase 04: GitHub en VS Code

## Objetivos de aprendizaje

- Usar la integración de Git en VS Code para gestionar cambios
- Hacer *stage*, *commit*, *push* y *pull* desde la interfaz gráfica
- Ver diferencias (*diffs*) visualmente
- Conocer extensiones útiles para trabajar con GitHub

---

## La barra lateral de control de código fuente

VS Code tiene Git integrado. El ícono de **Control de código fuente** (*Source Control*) en la barra lateral izquierda es tu centro de operaciones:

```
┌──────────────────────────────────┐
│  SOURCE CONTROL                  │
│                                  │
│  Changes (Cambios)               │
│    M  notebook.ipynb             │
│    U  nuevo_archivo.py           │
│                                  │
│  Staged Changes (Preparados)     │
│    (vacío)                       │
│                                  │
│  [Message input box]             │
│  [✓ Commit button]               │
└──────────────────────────────────┘
```

### Indicadores de estado

| Letra | Significado | Descripción |
|-------|------------|-------------|
| **M** | *Modified* | Archivo modificado |
| **U** | *Untracked* | Archivo nuevo, no rastreado |
| **D** | *Deleted* | Archivo eliminado |
| **A** | *Added* | Archivo agregado al *staging* |

## Flujo de trabajo en VS Code

### 1. Ver cambios (*diffs*)

Haz clic en cualquier archivo modificado en la vista de *Source Control* para ver las diferencias lado a lado:

- **Rojo**: líneas eliminadas
- **Verde**: líneas agregadas

Esto es mucho más visual que usar `git diff` en la terminal.

### 2. Preparar cambios (*stage*)

Para mover archivos al área de preparación (*staging area*):

- **Un archivo**: haz clic en el ícono `+` junto al nombre del archivo
- **Todos los archivos**: haz clic en el ícono `+` junto a "Changes"

Para quitar un archivo del *staging*:

- Haz clic en el ícono `-` junto al archivo en "Staged Changes"

### 3. Confirmar cambios (*commit*)

1. Escribe un mensaje descriptivo en el campo de texto
2. Haz clic en el botón **"✓ Commit"** o presiona `Ctrl+Enter`

### 4. Sincronizar con GitHub

Después del *commit*, usa los botones de sincronización:

- **Push** (↑): Enviar tus cambios al repositorio remoto en GitHub
- **Pull** (↓): Descargar cambios del repositorio remoto
- **Sync** (↑↓): Hace *pull* y luego *push* en un solo paso

También puedes usar la barra de estado inferior de VS Code, donde aparece el nombre de la rama actual y los botones de sincronización.

## Cambiar de rama en VS Code

Haz clic en el **nombre de la rama** en la esquina inferior izquierda de VS Code:

```
┌──────────────────────────────────────────────────┐
│                                          main ↺  │
└──────────────────────────────────────────────────┘
                                            ↑
                                    Clic aquí para
                                    cambiar de rama
```

Se abrirá un menú donde puedes:

- Seleccionar una rama existente
- Crear una nueva rama (*Create new branch*)

## Resolver conflictos visualmente

Cuando ocurre un conflicto de *merge*, VS Code muestra opciones interactivas directamente en el archivo:

```
<<<<<<< HEAD (Current Change)
contenido actual
=======
contenido entrante
>>>>>>> otra-rama (Incoming Change)
```

VS Code ofrece botones sobre cada conflicto:

- **Accept Current Change**: mantener tu versión
- **Accept Incoming Change**: mantener la versión entrante
- **Accept Both Changes**: mantener ambas versiones
- **Compare Changes**: ver las diferencias lado a lado

## Extensiones recomendadas

### GitHub Pull Requests and Issues

Esta extensión permite gestionar *Pull Requests* e *Issues* directamente desde VS Code:

- Ver y revisar Pull Requests
- Crear nuevos Issues
- Hacer *checkout* de ramas de un PR

Para instalarla: busca "GitHub Pull Requests" en la pestaña de extensiones (`Ctrl+Shift+X`).

### GitLens

Extensión que agrega información adicional de Git:

- Ver quién modificó cada línea (*blame*)
- Navegar el historial de un archivo
- Comparar versiones

---

## Ejercicios prácticos

1. Abre el repositorio del equipo en VS Code
2. Modifica cualquier archivo de texto
3. Ve a la vista de *Source Control* y observa los cambios
4. Haz clic en el archivo modificado para ver el *diff*
5. Prepara (*stage*) el archivo con el botón `+`
6. Escribe un mensaje de *commit* y confirma
7. Cambia de rama usando el selector en la esquina inferior izquierda

---

[← Clase anterior](../clase-03-ramas-y-merge/README.md) | [Siguiente clase: Flujo de trabajo en equipo →](../clase-05-flujo-de-trabajo-en-equipo/README.md)

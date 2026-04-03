# Clase 08: Referencia de Comandos y Flujos de Trabajo

Esta clase es una consolidación de todo lo visto en el módulo. No introduce conceptos nuevos: su propósito es que tengas un lugar único al que volver cuando necesites recordar un comando o el orden correcto de los pasos para una tarea típica.

---

## Comandos de referencia rápida

### Configuración inicial

| Comando | Descripción |
| ------- | ----------- |
| `git config --global user.name "Tu Nombre"` | Define tu nombre de usuario global |
| `git config --global user.email "tu@email.com"` | Define tu correo electrónico global |
| `git config --list` | Muestra la configuración actual |

### Creación y clonación

| Comando | Descripción |
| ------- | ----------- |
| `git init` | Crea un nuevo repositorio Git en el directorio actual |
| `git clone <url>` | Clona un repositorio remoto |

### Cambios en archivos

| Comando | Descripción |
| ------- | ----------- |
| `git status` | Muestra el estado actual del repositorio |
| `git add <archivo>` | Agrega un archivo específico al área de *staging* |
| `git add .` | Agrega todos los cambios al *staging* |
| `git commit -m "mensaje"` | Guarda los cambios con un mensaje descriptivo |
| `git commit -am "mensaje"` | Agrega y confirma todos los archivos modificados en un solo paso |

### Historial y diferencias

| Comando | Descripción |
| ------- | ----------- |
| `git log` | Muestra el historial de *commits* |
| `git log --oneline` | Muestra un historial compacto |
| `git diff` | Muestra los cambios entre el *working directory* y el *staging area* |

### Ramas

| Comando | Descripción |
| ------- | ----------- |
| `git branch` | Lista todas las ramas |
| `git branch <nombre>` | Crea una nueva rama |
| `git checkout <nombre>` | Cambia a otra rama |
| `git checkout -b <nombre>` | Crea y cambia a una nueva rama en un solo paso |
| `git merge <rama>` | Fusiona una rama con la rama actual |
| `git branch -d <nombre>` | Elimina una rama local |

### Repositorio remoto

| Comando | Descripción |
| ------- | ----------- |
| `git remote -v` | Muestra los repositorios remotos configurados |
| `git remote add origin <url>` | Conecta el repositorio local con uno remoto |
| `git push -u origin main` | Envía los commits locales a la rama principal por primera vez |
| `git push` | Envía los cambios al repositorio remoto |
| `git pull` | Descarga y fusiona los cambios del remoto |
| `git fetch` | Descarga los cambios sin fusionarlos automáticamente |

### Deshacer cambios

| Comando | Descripción |
| ------- | ----------- |
| `git restore <archivo>` | Deshace cambios en un archivo antes de agregarlo al *staging* |
| `git restore --staged <archivo>` | Quita un archivo del *staging area* sin perder los cambios |
| `git reset --soft HEAD~1` | Deshace el último commit manteniendo los cambios en *staging* |
| `git reset --hard <commit>` | Vuelve el repositorio a un commit anterior (elimina cambios permanentemente) |
| `git revert <commit>` | Crea un nuevo commit que revierte uno anterior (seguro para ramas publicadas) |

### Otros útiles

| Comando | Descripción |
| ------- | ----------- |
| `git stash` | Guarda temporalmente los cambios sin hacer commit |
| `git stash push -m "mensaje"` | Guarda el stash con un nombre descriptivo |
| `git stash list` | Lista todos los stashes guardados |
| `git stash pop` | Recupera y elimina el último stash |
| `git tag <nombre>` | Crea una etiqueta (por ejemplo, para marcar una versión) |
| `git fetch --prune` | Elimina del local las referencias a ramas remotas que ya no existen |

---

## Flujos de trabajo típicos

### Clonar un repositorio y comenzar a trabajar

Descarga un repositorio existente y crea tu rama de trabajo.

```bash
git clone https://github.com/usuario/repositorio.git
cd repositorio
git checkout -b nueva-rama
```

---

### Flujo completo de desarrollo de una feature

Flujo estándar desde cero: crear una rama, trabajar, hacer commits y publicar los cambios.

```bash
git checkout main
git pull                                      # asegurarse de tener main actualizado

git checkout -b mi-feature                   # crear rama de trabajo

# ... editar archivos ...

git add .
git commit -m "Agrega nueva funcionalidad"

git push -u origin mi-feature                # publicar la rama en GitHub
```

---

### Actualizar tu rama con cambios de `main`

Cuando otros han integrado cambios en `main` y quieres incorporarlos en tu rama activa.

**Con merge** (crea un commit de fusión):

```bash
git checkout main
git pull
git checkout mi-rama
git merge main
```

**Con rebase** (reescribe tu historial sobre `main`, más lineal):

```bash
git checkout mi-rama
git fetch origin
git rebase origin/main
```

> Usa **merge** si la rama es compartida con otros. Usa **rebase** solo en ramas personales, ya que reescribe el historial.

---

### Resolver conflictos después de un merge

Cuando Git no puede fusionar automáticamente dos ramas.

```bash
git merge main
# Git informa los archivos con conflicto

# Abre cada archivo con conflicto, edítalo y elimina los marcadores:
# <<<<<<< HEAD
# tu versión
# =======
# versión entrante
# >>>>>>> main

git add .
git commit -m "Resuelve conflictos con main"
```

---

### Guardar cambios temporales con stash

Útil cuando necesitas cambiar de rama con trabajo a medias que aún no está listo para un commit.

```bash
git stash push -m "trabajo en progreso"      # guardar cambios
git checkout otra-rama                       # cambiar de rama
# ... hacer lo que necesitabas ...
git checkout mi-rama-original
git stash pop                                # recuperar los cambios
```

---

### Deshacer cambios antes de hacer commit

Descarta modificaciones en archivos que aún no fueron confirmados.

```bash
# Descartar cambios en un archivo específico
git restore archivo.txt

# Sacar un archivo del staging (sin perder los cambios)
git restore --staged archivo.txt
```

---

### Deshacer el último commit (manteniendo los cambios)

Revierte el último commit pero conserva los cambios en el área de trabajo para editarlos o reconfirmarlos.

```bash
git reset --soft HEAD~1
# los cambios siguen en staging, listos para un nuevo commit
```

---

### Eliminar una rama después de hacer merge

Limpia las ramas locales y remotas una vez que el Pull Request fue fusionado.

```bash
git checkout main
git pull                                     # actualizar main con el merge recién hecho

git branch -d mi-rama                        # eliminar rama local
git push origin --delete mi-rama            # eliminar rama remota
git fetch --prune                            # limpiar referencias locales a ramas remotas eliminadas
```

---

## Diagrama: el flujo de un archivo en Git

```
 Directorio de trabajo      Área de staging           Repositorio local         GitHub (remoto)
 (Working Directory)        (Staging Area)           (Local Repository)       (Remote Repository)
┌──────────────────┐       ┌──────────────┐         ┌──────────────────┐       ┌──────────────┐
│                  │       │              │         │                  │       │              │
│  Editas archivos │─add──>│  Preparados  │─commit─>│  Historial de    │─push─>│  Repositorio │
│                  │       │  para commit │         │  commits         │       │  en GitHub   │
│                  │<──────│              │         │                  │<─pull─│              │
└──────────────────┘restore└──────────────┘         └──────────────────┘       └──────────────┘
```

---

[← Clase anterior](../clase-07-gitignore-stash-deshacer/README.md) | [Volver al índice del módulo →](../README.md)

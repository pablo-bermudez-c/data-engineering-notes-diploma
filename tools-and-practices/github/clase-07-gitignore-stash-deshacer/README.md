# Clase 07: .gitignore, Stash y Deshacer Cambios

## Objetivos de aprendizaje

- Excluir archivos del control de versiones con `.gitignore`
- Guardar trabajo temporal con `git stash`
- Deshacer cambios de distintas formas según la situación

---

## `.gitignore`: qué no debe entrar al repositorio

El archivo `.gitignore` le indica a Git qué archivos o carpetas debe ignorar por completo. Git actuará como si esos archivos no existieran.

### ¿Qué archivos conviene ignorar?

| Tipo | Ejemplos |
|------|----------|
| Archivos generados automáticamente | `__pycache__/`, `*.pyc`, `.ipynb_checkpoints/` |
| Dependencias instaladas | `node_modules/`, `venv/`, `.env/` |
| Archivos de configuración local | `.env`, `config.local.json` |
| Archivos del sistema operativo | `.DS_Store` (Mac), `Thumbs.db` (Windows) |
| Archivos de tu editor | `.vscode/settings.json`, `.idea/` |
| Archivos de datos grandes o sensibles | `*.csv`, `*.xlsx`, `datos_privados/` |

### Crear un `.gitignore`

Crea un archivo llamado `.gitignore` en la raíz de tu repositorio:

```bash
# En la terminal
touch .gitignore
```

Luego edítalo con las reglas que necesites. Ejemplo para un proyecto de Python con Jupyter:

```gitignore
# Python
__pycache__/
*.py[cod]
*.pyo
*.pyd

# Jupyter Notebook
.ipynb_checkpoints/
*.ipynb_checkpoints

# Entornos virtuales
venv/
.venv/
env/

# Variables de entorno (¡nunca subir credenciales!)
.env
.env.local

# Sistema operativo
.DS_Store
Thumbs.db

# VS Code
.vscode/
```

### Sintaxis básica de `.gitignore`

```gitignore
# Esto es un comentario

datos.csv          # ignora un archivo específico
*.log              # ignora todos los archivos .log
logs/              # ignora toda una carpeta
!logs/importante.log  # excepción: no ignores este archivo en particular
datos/**/*.tmp     # ignora archivos .tmp dentro de cualquier subcarpeta de datos/
```

### ¿Y si ya subí un archivo que debería ignorar?

Si el archivo ya está rastreado por Git, agregarlo al `.gitignore` no es suficiente. Primero hay que quitarlo del seguimiento:

```bash
# Dejar de rastrear el archivo (sin borrarlo de tu computador)
git rm --cached nombre_del_archivo.csv

# Para una carpeta completa
git rm --cached -r carpeta/

# Luego agrega la regla al .gitignore, y confirma los cambios
git add .gitignore
git commit -m "Deja de rastrear archivos ignorados"
```

> **Importante**: nunca subas archivos con contraseñas, tokens de API o datos sensibles. Una vez en el historial de Git, son difíciles de eliminar por completo.

### Usar una plantilla de `.gitignore`

GitHub ofrece plantillas listas para usar según el lenguaje o herramienta. Puedes encontrarlas en [github.com/github/gitignore](https://github.com/github/gitignore) o generarlas en [gitignore.io](https://www.toptal.com/developers/gitignore).

---

## `git stash`: guardar trabajo sin hacer commit

Imagina que estás trabajando a mitad de un análisis y de repente necesitas cambiar de rama para revisar algo urgente. No quieres hacer un *commit* porque tu trabajo está incompleto. Para eso existe `git stash`.

**`git stash`** guarda temporalmente tus cambios sin confirmarlos, dejando tu directorio de trabajo limpio.

```
 Directorio de trabajo        Stash (pila temporal)
┌────────────────────┐       ┌───────────────────────┐
│                    │ stash │                       │
│  Cambios sin       │──────>│  stash@{0}: trabajo   │
│  confirmar         │       │  en progreso...       │
│                    │<──────│                       │
└────────────────────┘  pop  └───────────────────────┘
```

### Comandos principales

```bash
# Guardar los cambios actuales en el stash
git stash

# Guardar con un nombre descriptivo (recomendado)
git stash push -m "análisis ciclo 5 a mitad"

# Ver la lista de stashes guardados
git stash list
# Salida:
# stash@{0}: On main: análisis ciclo 5 a mitad
# stash@{1}: On main: cambios en notebook

# Recuperar el último stash (y eliminarlo de la lista)
git stash pop

# Recuperar un stash específico sin eliminarlo de la lista
git stash apply stash@{1}

# Eliminar un stash que ya no necesitas
git stash drop stash@{0}

# Limpiar todos los stashes
git stash clear
```

### Flujo típico con stash

```bash
# 1. Estás trabajando en tu rama y tienes cambios a medias
git status
# → Changes not staged for commit: notebook.ipynb

# 2. Surge algo urgente, necesitas cambiar de rama
git stash push -m "análisis incompleto ciclo 5"

# 3. Ahora el directorio está limpio, puedes cambiar de rama
git switch fix/error-datos
# ... resuelves el problema urgente, haces commit ...

# 4. Vuelves a tu rama original
git switch analisis/ciclo-5

# 5. Recuperas tu trabajo guardado
git stash pop
# → tus cambios están de vuelta
```

---

## Deshacer cambios

Git ofrece varias formas de deshacer cambios. La clave es entender *en qué estado* están los cambios que quieres revertir.

```
¿Qué quiero deshacer?
│
├── Cambio en mi directorio de trabajo (no hice git add)
│   └── git restore <archivo>
│
├── Cambio ya en staging (hice git add, no hice commit)
│   └── git restore --staged <archivo>
│
├── Un commit reciente (aún no hice push)
│   └── git reset
│
└── Un commit ya publicado (ya hice push)
    └── git revert
```

### `git restore` — deshacer cambios locales

```bash
# Descartar cambios en un archivo (volver a cómo estaba en el último commit)
git restore notebook.ipynb

# Descartar todos los cambios no confirmados
git restore .

# Sacar un archivo del staging (sin perder los cambios)
git restore --staged notebook.ipynb
```

> ⚠️ `git restore` sobre archivos no en staging es irreversible. Los cambios se pierden permanentemente.

### `git reset` — mover el historial hacia atrás

Úsalo para deshacer commits que **aún no enviaste a GitHub**.

```bash
# Ver el historial para saber a dónde quieres volver
git log --oneline
# a1b2c3d Commit equivocado
# e4f5g6h Commit correcto  ← quiero volver aquí

# --soft: deshace el commit pero conserva los cambios en staging
git reset --soft HEAD~1

# --mixed (opción por defecto): deshace el commit y saca los cambios del staging
git reset HEAD~1

# --hard: deshace el commit y descarta los cambios completamente
git reset --hard HEAD~1
```

| Opción | ¿Deshace el commit? | ¿Qué pasa con los cambios? |
|--------|--------------------|-----------------------------|
| `--soft` | ✅ | Quedan en *staging*, listos para un nuevo commit |
| `--mixed` | ✅ | Quedan en el directorio de trabajo, sin hacer *stage* |
| `--hard` | ✅ | Se pierden permanentemente |

> ⚠️ Nunca uses `git reset` en commits que ya hayas enviado con `git push`. Altera el historial y puede causar problemas a tu equipo.

### `git revert` — deshacer cambios de forma segura

Si ya hiciste `push` de un commit con un error, la solución segura es `git revert`. En lugar de borrar el commit del historial, **crea un nuevo commit** que deshace los cambios.

```bash
# Ver el historial para identificar el commit a revertir
git log --oneline
# a1b2c3d Subió archivo sensible  ← quiero revertir este
# e4f5g6h Commit anterior

# Revertir el commit (se crea un nuevo commit de reversión)
git revert a1b2c3d

# Enviar el cambio al repositorio remoto
git push
```

```
Antes:   ... ●──────●──────●  (commit problemático)
                           ↑
Después: ... ●──────●──────●──────●  (nuevo commit que revierte el anterior)
```

---

## Resumen: ¿qué herramienta usar?

| Situación | Solución |
|-----------|----------|
| No quiero que Git rastree ciertos archivos | `.gitignore` |
| Ya subí un archivo que debería ignorar | `git rm --cached` + `.gitignore` |
| Necesito cambiar de rama con trabajo a medias | `git stash` |
| Deshago cambios que no hice *stage* | `git restore <archivo>` |
| Saco un archivo del *staging* | `git restore --staged <archivo>` |
| Deshago un commit local (sin push) | `git reset` |
| Deshago un commit ya publicado | `git revert` |

---

## Ejercicios prácticos

1. Crea un archivo `.gitignore` en el repositorio del equipo que incluya al menos: carpetas de entornos virtuales, archivos `.pyc` y `.DS_Store`
2. Crea un archivo `prueba.log`, verifica con `git status` que Git lo detecta, luego agrégalo al `.gitignore` y verifica que desaparece del estado
3. Modifica un archivo cualquiera sin hacer *commit*, guárdalo con `git stash push -m "prueba de stash"`, verifica que el directorio está limpio y luego recupéralo con `git stash pop`
4. Haz un *commit* con un mensaje equivocado y corrígelo con `git reset --soft HEAD~1` seguido de un nuevo `git commit -m "mensaje correcto"`

---

[← Clase anterior](../clase-06-colaboracion-en-github/README.md) | [Volver al índice del módulo →](../README.md)

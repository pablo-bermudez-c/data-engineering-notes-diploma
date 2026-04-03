# Clase 02: Repositorios y Commits

## Objetivos de aprendizaje

- Entender qué es un repositorio (*repository*)
- Aprender a clonar (*clone*) un repositorio existente
- Comprender el flujo: modificar → preparar → confirmar
- Usar los comandos `git status`, `git add` y `git commit`

---

## ¿Qué es un repositorio?

Un **repositorio** (*repository* o *repo*) es una carpeta de tu computador que está siendo rastreada por Git. Git guarda todo el historial de cambios dentro de una carpeta oculta llamada `.git`.

Existen dos formas de obtener un repositorio:

1. **Crear uno nuevo** con `git init`
2. **Clonar uno existente** con `git clone`

## Clonar un repositorio existente

Para trabajar con un repositorio que ya existe en GitHub, usamos el comando `git clone`:

```bash
git clone https://github.com/usuario/nombre-del-repo.git
```

Esto crea una copia completa del repositorio en tu computador, incluyendo todo su historial.

### Ejemplo con nuestro repositorio

```bash
git clone https://github.com/tu-organizacion/equipo-biomedico.git
```

Después de clonar, entra a la carpeta del repositorio:

```bash
cd equipo-biomedico
```

## Los tres estados de Git

Git maneja tus archivos en tres áreas:

```
 Directorio de trabajo     Área de preparación         Repositorio
 (Working Directory)          (Staging Area)           (Repository)
┌──────────────────┐     ┌──────────────────┐      ┌──────────────────┐
│                  │     │                  │      │                  │
│  Archivos que    │────>│  Archivos listos │─────>│  Cambios         │
│  modificas       │ add │  para confirmar  │commit│  guardados       │
│                  │     │                  │      │  permanentemente │
└──────────────────┘     └──────────────────┘      └──────────────────┘
```

1. **Directorio de trabajo** (*working directory*): donde editas tus archivos normalmente
2. **Área de preparación** (*staging area*): donde colocas los cambios que quieres guardar
3. **Repositorio** (*repository*): donde se almacenan los cambios confirmados

## Comandos esenciales

### `git status` — Ver el estado actual

Muestra qué archivos han sido modificados, cuáles están preparados (*staged*) y cuáles no están siendo rastreados:

```bash
git status
```

Ejemplo de salida:

```
On branch main
Changes not staged for commit:
  modified:   notebook.ipynb

Untracked files:
  nuevo_archivo.py
```

### `git add` — Preparar cambios

Mueve archivos al área de preparación (*staging area*):

```bash
# Preparar un archivo específico
git add notebook.ipynb

# Preparar varios archivos
git add archivo1.py archivo2.py

# Preparar todos los cambios
git add .
```

### `git commit` — Confirmar cambios

Guarda los cambios preparados en el historial del repositorio con un mensaje descriptivo:

```bash
git commit -m "Agrega análisis de datos del ciclo 5"
```

> **Buena práctica**: Escribe mensajes de commit claros y descriptivos. Usa verbos en presente o imperativo: "Agrega...", "Corrige...", "Actualiza...".

### `git log` — Ver el historial

Muestra la lista de cambios confirmados (*commits*):

```bash
git log
```

Para una vista más compacta:

```bash
git log --oneline
```

Ejemplo de salida:

```
b5e3b57 Uploading notebook and patch to load data
9564eb6 agregue texto inicial del readme
eb7b6ca Firt
```

## Flujo típico de trabajo

```bash
# 1. Verificar el estado actual
git status

# 2. Hacer cambios en tus archivos (editar, crear, etc.)

# 3. Preparar los cambios
git add nombre_del_archivo.py

# 4. Confirmar los cambios
git commit -m "Descripción clara del cambio"

# 5. Verificar que todo está limpio
git status
```

---

## Ejercicios prácticos

1. **Clona** el repositorio del equipo en tu computador
2. Crea un archivo de texto con tu nombre (ejemplo: `mi_nombre.txt`)
3. Usa `git status` para ver que Git detectó el nuevo archivo
4. Usa `git add mi_nombre.txt` para preparar el archivo
5. Usa `git commit -m "Agrega archivo de prueba"` para confirmar el cambio
6. Usa `git log --oneline` para ver tu cambio en el historial

---

[← Clase anterior](../clase-01-introduccion-a-git/README.md) | [Siguiente clase: Ramas y Merge →](../clase-03-ramas-y-merge/README.md)

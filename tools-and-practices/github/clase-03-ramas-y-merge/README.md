# Clase 03: Ramas y Merge

## Objetivos de aprendizaje

- Entender qué es una rama (*branch*) y para qué sirve
- Crear y cambiar entre ramas
- Fusionar (*merge*) ramas
- Resolver conflictos (*merge conflicts*)

---

## ¿Qué es una rama?

Una **rama** (*branch*) es una línea de desarrollo independiente. Imagina que tu proyecto es un árbol: la rama principal (*main*) es el tronco, y cada rama nueva es una bifurcación donde puedes trabajar sin afectar el tronco.

```
        clase-analisis
       /              \
main ●───●───●─────────●───● main (con los cambios fusionados)
              \       /
               fix-bug
```

### ¿Por qué usar ramas?

- **Trabajar en paralelo**: Cada persona puede trabajar en su propia rama sin interferir con otros
- **Experimentar sin riesgo**: Si algo sale mal, la rama principal no se ve afectada
- **Organizar el trabajo**: Cada rama corresponde a una tarea específica

## Ver las ramas existentes

```bash
# Ver ramas locales
git branch

# Ver todas las ramas (locales y remotas)
git branch -a
```

La rama actual aparece marcada con un asterisco `*`:

```
* main
  rehacom-analysis-python
```

## Crear y cambiar de rama

### Crear una nueva rama

```bash
# Crear una rama nueva
git branch nombre-de-la-rama

# Crear y cambiar a la nueva rama en un solo paso
git switch -c nombre-de-la-rama
```

### Cambiar de rama

```bash
git switch nombre-de-la-rama
```

> **Convención de nombres**: Usa nombres descriptivos en minúsculas separados por guiones. Ejemplos: `analisis-ciclo-5`, `fix-error-datos`, `agregar-graficos`.

### Ejemplo práctico

```bash
# Crear una rama para trabajar en un nuevo análisis
git switch -c analisis-ciclo-5

# Hacer cambios en tus archivos...
git add .
git commit -m "Agrega análisis del ciclo 5"

# Volver a la rama principal
git switch main
```

## Fusionar ramas (*merge*)

Cuando terminas tu trabajo en una rama, puedes **fusionarla** (*merge*) de vuelta a la rama principal:

```bash
# 1. Cambiar a la rama que recibirá los cambios (generalmente main)
git switch main

# 2. Fusionar la rama de trabajo
git merge analisis-ciclo-5
```

### Tipos de merge

| Tipo | Cuándo ocurre | Resultado |
|------|---------------|-----------|
| **Fast-forward** | No hubo cambios en *main* mientras trabajabas | Git simplemente mueve el puntero hacia adelante |
| **Merge commit** | Hubo cambios en ambas ramas | Git crea un nuevo *commit* que combina ambas ramas |

## Resolver conflictos (*merge conflicts*)

Un **conflicto** ocurre cuando dos personas modifican la misma parte de un archivo en diferentes ramas. Git no puede decidir automáticamente cuál versión mantener.

### ¿Cómo se ve un conflicto?

Git marca el archivo con indicadores especiales:

```
<<<<<<< HEAD
Este es el contenido de la rama actual (main)
=======
Este es el contenido de la rama que estás fusionando
>>>>>>> analisis-ciclo-5
```

### ¿Cómo resolverlo?

1. **Abre el archivo** con el conflicto
2. **Decide qué contenido mantener**: puedes quedarte con una versión, la otra, o combinar ambas
3. **Elimina los marcadores** (`<<<<<<<`, `=======`, `>>>>>>>`)
4. **Guarda el archivo**
5. **Confirma la resolución**:

```bash
git add archivo-con-conflicto.py
git commit -m "Resuelve conflicto en archivo-con-conflicto.py"
```

> En la siguiente clase veremos cómo VS Code hace este proceso mucho más visual y fácil.

---

## Ejercicios prácticos

1. Crea una nueva rama llamada `practica-ramas`
2. En esa rama, crea un archivo `mi_practica.txt` con algún texto
3. Haz un *commit* con el cambio
4. Vuelve a la rama `main`
5. Fusiona (*merge*) la rama `practica-ramas` en `main`
6. Verifica con `git log --oneline` que el cambio aparece en `main`

---

[← Clase anterior](../clase-02-repositorios-y-commits/README.md) | [Siguiente clase: GitHub en VS Code →](../clase-04-github-en-vscode/README.md)

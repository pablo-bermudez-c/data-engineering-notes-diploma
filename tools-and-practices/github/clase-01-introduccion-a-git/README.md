# Clase 01: Introducción a Git

## Objetivos de aprendizaje

- Entender qué es Git y por qué es útil
- Conocer la diferencia entre Git y GitHub
- Instalar y configurar Git en tu computador

---

## ¿Qué es Git?

**Git** es un sistema de control de versiones (*version control system*). Piensa en él como un "historial de cambios" para tus archivos, similar a la función "Historial de versiones" de Google Docs, pero mucho más potente.

Con Git puedes:

- **Guardar versiones** de tu trabajo en diferentes momentos
- **Volver atrás** si algo sale mal
- **Trabajar en equipo** sin sobrescribir el trabajo de otros
- **Ver quién hizo qué cambio** y cuándo

### Analogía

Imagina que estás escribiendo un informe. Sin Git, podrías terminar con archivos como:

```
informe_v1.docx
informe_v2.docx
informe_v2_final.docx
informe_v2_final_FINAL.docx
```

Con Git, solo tienes **un archivo** y Git se encarga de recordar todas las versiones anteriores.

## ¿Qué es GitHub?

**GitHub** es una plataforma en la nube (*cloud*) que almacena tus repositorios Git de forma remota. Es como un "Google Drive para código" que además facilita la colaboración en equipo.

### Diferencia clave

| Git | GitHub |
|-----|--------|
| Software que se instala en tu computador | Sitio web / servicio en la nube |
| Funciona de forma local (*local*) | Funciona de forma remota (*remote*) |
| Gestiona el historial de cambios | Permite compartir y colaborar |
| Gratuito y de código abierto | Gratuito para repositorios públicos y privados |

> **Resumen**: Git es la herramienta, GitHub es el servicio donde compartimos nuestro trabajo.

## Instalación de Git

### Windows

1. Descarga Git desde [git-scm.com/downloads](https://git-scm.com/downloads)
2. Ejecuta el instalador
3. Acepta las opciones por defecto (son adecuadas para la mayoría de usuarios)
4. Verifica la instalación abriendo una terminal y escribiendo:

```bash
git --version
```

Deberías ver algo como: `git version 2.xx.x`

## Configuración inicial

Después de instalar Git, necesitas configurar tu nombre y correo electrónico. Estos datos aparecerán en cada cambio (*commit*) que hagas.

Abre una terminal (o Git Bash en Windows) y ejecuta:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu.correo@ejemplo.com"
```

> Usa el mismo correo que registraste en GitHub.

Para verificar tu configuración:

```bash
git config --list
```

---

## Conectar tu cuenta de GitHub al computador

Antes de poder enviar (*push*) o recibir (*pull*) cambios desde GitHub, necesitas autenticarte. La forma más sencilla es usando **GitHub CLI**.

### Instalar GitHub CLI

Descarga el instalador desde [cli.github.com](https://cli.github.com) y sigue las instrucciones para tu sistema operativo.

Verifica la instalación:

```bash
gh --version
```

### Iniciar sesión con tu cuenta de GitHub

```bash
gh auth login
```

Este comando te guiará por un proceso interactivo. Elige las siguientes opciones cuando se te pregunte:

```
? Where do you use GitHub?         GitHub.com
? What is your preferred protocol? HTTPS
? Authenticate Git with your GitHub credentials? Yes
? How would you like to authenticate? Login with a web browser
```

Se abrirá el navegador para que inicies sesión con tu cuenta de GitHub. Una vez que apruebes el acceso, la autenticación quedará guardada en tu computador.

### Verificar que la autenticación fue exitosa

```bash
gh auth status
```

Deberías ver algo como:

```
github.com
  ✓ Logged in to github.com as tu-usuario
  ✓ Git operations for github.com configured to use https protocol.
```

> A partir de este punto, podrás hacer `git push` y `git pull` sin necesidad de ingresar tu contraseña cada vez.

---

## Ejercicios prácticos

1. **Instala Git** en tu computador siguiendo las instrucciones de arriba
2. **Configura tu nombre y correo** con `git config`
3. **Verifica la instalación** ejecutando `git --version` en tu terminal
4. **Crea una cuenta en GitHub** en [github.com](https://github.com) si aún no tienes una
5. **Instala GitHub CLI** y ejecuta `gh auth login` para conectar tu cuenta

---

[Siguiente clase: Repositorios y Commits →](../clase-02-repositorios-y-commits/README.md)

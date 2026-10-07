# Notas de desarrollo — Incremento 1: animales, adoptantes y usuarios

> Una entrada por funcionalidad, escrita al terminarla. Sirve para estudiar lo hecho, como borrador del Capítulo 7
> del informe y como respaldo para la defensa. Alcance del incremento (decidido el 07-10): usuarios con inicio de
> sesión y perfiles, animales, adoptantes y la base estética del sitio público y del panel. La postulación pasa al
> incremento 2.

---

## 0. Preparación del entorno (07-10-2026)

**Qué se hizo.** Se instaló el entorno de desarrollo y se creó el proyecto Laravel con el kit oficial de React.

**Requisito que cumple.** Ninguno directamente; es la base de todos. Concreta la decisión de tecnologías (informe
2.2.3 y 2.5) y da los datos para la sección 2.6.

**Cómo quedó.**

| Pieza | Versión real |
|---|---|
| Windows 11 Pro + WSL2 | Ubuntu 26.04.1 LTS (usuario `carlo`) |
| Docker Desktop | 4.94.0 (Docker 29.8.2, Compose 5.5.1) |
| Laravel | 13.35.0 |
| Inertia (Laravel) | 3.5.1 |
| Fortify | 1.41.0 |
| Pest | 4.7 |
| Imagen de PHP (`sail-8.3/app`) | Ubuntu 24.04, PHP 8.3.35, Node 24.21.0, npm 12.2.0, Composer 2.10.3 |
| Base de datos | MySQL 8.4.11 (imagen `mysql:8.4`), base `rescate` |

**Pasos, en orden.**

1. Se activó en Windows la función *Virtual Machine Platform* y se instaló WSL2 con Ubuntu: Docker la necesita en
   Windows, porque los contenedores usan el núcleo de Linux.
2. Se instaló Docker Desktop y se activó su integración con Ubuntu.
3. El proyecto se creó con un **contenedor temporal** de PHP 8.3 (`laravelsail/php83-composer`), sin instalar PHP en
   Windows: `laravel new codigo --react --database=mysql --pest`. La imagen temporal no trae Node, así que se usó la
   variable `LARAVEL_INSTALLER_NO_NODE=1`; las librerías de JavaScript se instalaron después dentro del contenedor
   de Sail. El primer intento, sin esa variable, se cortó a la mitad y se rehízo desde cero.
4. Se instaló **Sail** con MySQL y PHP 8.3 (`sail:install --with=mysql --php=8.3`) y se **publicó el Dockerfile**
   (`sail:publish`) para que la receta del entorno quede en el repositorio. Se borraron las recetas de las versiones
   que no se usan.
5. La primera construcción de la imagen se quedaba detenida. Diagnóstico: desde los contenedores, los servidores
   principales de Ubuntu (`archive.ubuntu.com`, `security.ubuntu.com`) no respondían por http, mientras el resto de
   internet sí. Solución: una línea en el Dockerfile que usa el **espejo oficial chileno** (`cl.archive.ubuntu.com`).
6. Con el código en `C:\`, `npm install` fallaba con *permission denied* y la página daba error 500. Diagnóstico: el
   contenedor ve las carpetas de `C:\` con dueño `root`, y Sail trabaja con un usuario normal (`sail`) que no puede
   escribir ahí. Solución: el código se movió a Ubuntu (`~/proyecto-de-titulo/codigo`), donde el dueño es `carlo`,
   con el mismo número de usuario (1000) que `sail`.
7. Se instalaron las librerías de JavaScript, se ejecutaron las migraciones iniciales y se comprobó que
   http://localhost responde.

**Archivos que importan.**

| Archivo | Qué es |
|---|---|
| `compose.yaml` | El plano de los contenedores: `laravel.test` (PHP + Node + el código) y `mysql` |
| `docker/8.3/Dockerfile` | La receta de la imagen de PHP, con las versiones fijas |
| `.env` | La configuración de este equipo (base de datos, contraseñas). No va a git |
| `.gitignore` | Viene de Laravel; excluye `.env`, `vendor/`, `node_modules/` y también `CLAUDE.md` (archivo de trabajo, no se versiona) |

**Por qué así.**

- **Docker con Sail y no PHP instalado en Windows**: las versiones quedan fijas y escritas en archivos, el entorno se
  reproduce en otro equipo con un comando y Windows queda limpio. Costo: en Windows exige WSL2 y que el código esté
  dentro de Ubuntu.
- **Docker sólo en desarrollo**: el hosting compartido (Hosty) no corre contenedores. Allá se sube el código y las
  pantallas compiladas; Docker sirve para que el entorno local tenga las mismas versiones de PHP y MySQL que el
  servidor.
- **Kit oficial de React**: trae resuelto el inicio de sesión, el registro y la recuperación de contraseña, la parte
  más sensible en seguridad.
- **PHP 8.3**: es el mínimo de Laravel 13; lo que funciona en 8.3 funciona en 8.4 y 8.5, así que sirve para cualquier
  versión que tenga Hosty desde la 8.3.

**Preguntas de la comisión.**

- *¿Por qué Docker si el servidor no lo usa?* Para reproducir en desarrollo las versiones del servidor; al servidor
  se despliega el código, no los contenedores.
- *¿Por qué modificó el Dockerfile oficial?* Un solo cambio, el espejo de Ubuntu, por un problema de red
  diagnosticado; está comentado en el archivo.
- *¿Y las 5 vulnerabilidades críticas de `npm audit`?* Están en herramientas de desarrollo (`concurrently`,
  `oxfmt`, `vite-plus`), que no se despliegan. No se forzó su corrección porque rompe las versiones del kit.

**Cómo se probó.** `sail artisan migrate:status` muestra las 5 migraciones del kit ejecutadas; http://localhost
responde con la página de bienvenida; VS Code dejó de marcar errores al instalarse las librerías.

**Diferencias con el diseño, para el Apéndice E.** La tabla de usuarios se llama `users` y su columna `name` (no
`usuarios` y `nombre`), y el kit agrega `remember_token`, `email_verified_at`, columnas de doble factor y la tabla
`passkeys`.

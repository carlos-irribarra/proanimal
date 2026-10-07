# Código del sistema — reglas técnicas

> Reglas para trabajar en el código. El contexto del proyecto (objetivos, reglas legales, decisiones, calendario,
> pendientes) está en el CLAUDE.md del informe: `C:\Proyectos\Proyecto de Titulo-2026\CLAUDE.md`
> (desde Ubuntu: `/mnt/c/Proyectos/Proyecto de Titulo-2026/CLAUDE.md`). Leerlo primero.

## Dónde está y cómo se trabaja

- Carpeta: `~/proyecto-de-titulo/codigo` en Ubuntu (WSL). Desde Windows:
  `\\wsl.localhost\Ubuntu\home\carlo\proyecto-de-titulo\codigo`. No está en `C:\` porque desde ahí el contenedor
  no puede escribir (permisos) y es lento.
- Se trabaja desde la ventana de VS Code del informe: Claude lee y edita por la ruta de Windows y corre comandos con
  `wsl.exe`. Carlos corre sus comandos en la terminal de Ubuntu.
- **Quién hace qué.** Claude escribe el código y explica cada paso **antes** de hacerlo, como a alguien que parte de
  cero. Carlos corre los comandos de uso (encender, migrar, compilar, probar) y hace todo lo de git. Claude le da el
  comando exacto y explica cada parte.

## Stack y versiones (fijadas en el entorno)

| Pieza | Versión | Dónde se fija |
|---|---|---|
| Laravel | 13.x (13.35.0 al crear) | `composer.json` |
| Inertia (Laravel) | 3.x | `composer.json` |
| Fortify (inicio de sesión) | 1.x | `composer.json` |
| React + TypeScript | kit oficial de Laravel | `package.json` |
| Pest (pruebas) | 4.x | `composer.json` |
| PHP | 8.3 | `compose.yaml` → `docker/8.3/Dockerfile` |
| Node | 24 | `docker/8.3/Dockerfile` (`NODE_VERSION`) |
| MySQL | 8.4 | `compose.yaml` (`mysql:8.4`) |
| Ubuntu (imagen) | 24.04 | `docker/8.3/Dockerfile` |

Cambio propio al Dockerfile de Sail: el espejo chileno de Ubuntu (`cl.archive.ubuntu.com`), porque desde Docker los
servidores principales no respondían por http. Cuando Hosty informe sus versiones de PHP y MySQL, se igualan aquí.

## Comandos (desde `~/proyecto-de-titulo/codigo`)

| Comando | Qué hace |
|---|---|
| `./vendor/bin/sail up -d` | Enciende los contenedores (PHP y MySQL) |
| `./vendor/bin/sail down` | Los apaga (sin `-v`, para no borrar la base de datos) |
| `./vendor/bin/sail npm run dev` | Compila las pantallas en vivo (queda corriendo) |
| `./vendor/bin/sail artisan migrate` | Ejecuta las migraciones pendientes |
| `./vendor/bin/sail artisan test` | Corre las pruebas de Pest |
| `./vendor/bin/sail npm install` | Instala las librerías de JavaScript |

El sistema queda en http://localhost.

## Convenciones

- **Nombres en español** para lo propio del sistema: tablas (`animales`, `adoptantes`), columnas, modelos (`Animal`,
  `Adoptante`), controladores, rutas y pantallas. **Lo que trae Laravel queda en inglés**: `users`, `name`,
  `remember_token`, `password_reset_tokens`, `sessions`, `cache`, `jobs`. Decisión del 07-10.
- Las tablas siguen el **Apéndice E** del informe (diccionario de datos). Toda diferencia entre lo diseñado y lo
  construido se anota en `notas/` para actualizar el Apéndice E y la Figura 6.4 al cerrar el incremento.
- Zona horaria: los plazos legales se cuentan en días hábiles de Chile; la aplicación debe usar
  `America/Santiago` (el contenedor está en UTC).
- **Datos ficticios siempre** en seeders, pruebas y capturas. Nunca datos reales de adoptantes.
- Comentarios en español, sólo donde explican un porqué que el código no muestra.

## Ciclo por funcionalidad

1. **Concepto**: qué se construye, qué RF cumple, qué piezas toca (migración, modelo, controlador, ruta, pantalla).
   Carlos lo aprueba.
2. **Código**: por partes, explicando cada archivo.
3. **Prueba automática** con Pest de las reglas importantes (base del Apéndice C).
4. **Carlos lo prueba** en el navegador.
5. **Nota** en `notas/incremento-N.md`: qué se hizo, RF, cómo funciona, archivos, por qué, preguntas de la comisión,
   cómo se probó. Es la fuente del Capítulo 7 del informe.
6. **Commit** del código (lo hace Carlos), separado del commit del informe.

## Cuidados conocidos

- No correr `npm audit fix --force`: las alertas son de herramientas de desarrollo y el `--force` rompe las versiones
  del kit.
- El kit trae verificación de correo, doble factor y passkeys. El proyecto decidió no verificar el correo (04-10);
  qué se apaga se decide al construir usuarios.
- Si un archivo se guarda desde Windows, revisar que quede con finales de línea de Linux (LF), sobre todo `.env`.

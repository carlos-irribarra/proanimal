# Sistema de gestión para organizaciones de rescate animal

Sistema web para la gestión de donaciones y adopciones de animales rescatados, con trazabilidad del cumplimiento de
la Ley N° 21.020 y seguimiento post-adopción asistido. Proyecto de título, Ingeniería de Ejecución en Computación e
Informática, Universidad del Bío-Bío. Organización: Agrupación Proanimal Brisa del Sol (Talcahuano).

Construido con Laravel 13, React con TypeScript (mediante Inertia) y MySQL 8.4. En desarrollo corre en contenedores
Docker con Laravel Sail.

## Requisitos

- Windows 10/11 con **WSL2** y una distribución Ubuntu, o un equipo Linux o macOS.
- **Docker Desktop** con la integración WSL activada para Ubuntu.
- Git.

No hace falta instalar PHP, Composer, Node ni MySQL: vienen en los contenedores.

## Puesta en marcha (primera vez)

En Windows, el proyecto debe estar **dentro de Ubuntu** (por ejemplo `~/proyecto-de-titulo/codigo`), no en `C:\`:
desde `C:\` el contenedor no puede escribir en la carpeta y todo es más lento.

```bash
git clone <url-del-repositorio> ~/proyecto-de-titulo/codigo
cd ~/proyecto-de-titulo/codigo

# Instalar las librerías de PHP con un contenedor temporal (no requiere PHP en el equipo)
docker run --rm -u "$(id -u):$(id -g)" -v "$(pwd):/var/www/html" -w /var/www/html \
    laravelsail/php83-composer:latest composer install --ignore-platform-reqs

cp .env.example .env            # luego ajustar DB_DATABASE, DB_HOST=mysql, DB_USERNAME=sail, DB_PASSWORD=password

./vendor/bin/sail up -d         # enciende PHP y MySQL (la primera vez construye la imagen: varios minutos)
./vendor/bin/sail artisan key:generate
./vendor/bin/sail npm install
./vendor/bin/sail artisan migrate
./vendor/bin/sail npm run dev   # queda corriendo
```

El sistema queda en http://localhost.

## Uso diario

```bash
cd ~/proyecto-de-titulo/codigo
./vendor/bin/sail up -d
./vendor/bin/sail npm run dev
```

Para apagar: `Ctrl + C` en la terminal de `npm run dev` y luego `./vendor/bin/sail down`.

## Pruebas

```bash
./vendor/bin/sail artisan test
```

## Versiones del entorno

PHP 8.3 (imagen sobre Ubuntu 24.04), Node 24, MySQL 8.4. Están fijadas en `compose.yaml` y `docker/8.3/Dockerfile`.

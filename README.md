# Laravel Docker Starter

A reusable Laravel development starter powered by Docker Compose.

## Tech Stack

| Technology | Version | Purpose |
|---|---|---|
| Laravel | 13 | PHP framework |
| PHP | 8.4 | Backend runtime |
| MySQL | 8.4 | Database |
| phpMyAdmin | 5.2 | Database management |
| Node.js | 24 LTS | Frontend tooling |
| Vite | Project lockfile | Frontend asset bundling |
| Tailwind CSS | Project lockfile | CSS framework |
| Docker Compose | V2 | Container orchestration |

## Requirements

- Docker Desktop with Docker Compose
- WSL 2 with Ubuntu (recommended on Windows)
- VS Code with WSL extension (recommended)
- Git

PHP, Composer, MySQL and Node.js do not need to be installed directly on the host.

## Getting Started

### 1. Create a new project

Create a new repository using this GitHub template, then clone it:

```bash
git clone https://github.com/farid-larav/laravel-docker-starter.git
cd YOUR_PROJECT
```

### 2. Configure environment variables

```bash
cp .env.example .env
```

Open `.env` and configure:

```dotenv
APP_NAME="My Laravel App"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel
DB_PASSWORD=your_local_db_password

MYSQL_ROOT_PASSWORD=your_local_root_password

HOST_UID=1000
HOST_GID=1000
```

Replace the example passwords. Preserve the additional Laravel settings already present in `.env`.

Set `HOST_UID` and `HOST_GID` to the output of:

```bash
id -u
id -g
```

Docker Compose reads the environment values from `.env`.

**Important:** MySQL initialization credentials only take effect when the database volume is first initialized.

### 3. Build the PHP image

```bash
docker compose build app
```

### 4. Install PHP dependencies

```bash
docker compose run --rm --no-deps \
  -e COMPOSER_HOME=/tmp/composer \
  app composer install
```

### 5. Install frontend dependencies

```bash
docker compose run --rm --no-deps node npm ci
```

### 6. Start the containers

```bash
docker compose up -d
```

Check the status:

```bash
docker compose ps
```

MySQL should become healthy, and the application, phpMyAdmin and Node.js services should be running.

### 7. Generate the application key

```bash
docker compose exec app php artisan key:generate
```

### 8. Run database migrations

```bash
docker compose exec app php artisan migrate
```

### 9. Open the application

| Service | Address |
|---|---|
| Laravel | http://localhost:8000 |
| phpMyAdmin | http://localhost:8080 |
| Vite | http://localhost:5173 |
| MySQL (host access) | 127.0.0.1:3307 |

To log in to phpMyAdmin, use the `DB_USERNAME` and `DB_PASSWORD` defined in your `.env` file.

## Common Docker Commands

| Task | Command |
|---|---|
| Start services | `docker compose up -d` |
| Stop services | `docker compose down` |
| Check status | `docker compose ps` |
| View logs | `docker compose logs -f` |
| Run Artisan | `docker compose exec app php artisan` |
| Run migrations | `docker compose exec app php artisan migrate` |
| Run tests | `docker compose exec app php artisan test` |
| Install PHP package | `docker compose exec app composer require vendor/package` |
| Install NPM package | `docker compose exec node npm install package` |
| Build frontend assets | `docker compose run --rm --no-deps node npm run build` |

## Development

Laravel uses the PHP development server on port 8000.

The Node.js container runs Vite on port 5173, supporting frontend hot module replacement.

Update PHP, Blade, CSS and JavaScript files using VS Code connected to WSL Ubuntu.

## Database Persistence

MySQL data is stored in a Docker named volume.

Running:

```bash
docker compose down
```

stops and removes the containers but preserves the database volume.

**Warning:** Running `docker compose down -v` can permanently remove the database volume and its contents.

## Repository Structure

```text
app/
bootstrap/
config/
database/
docker/
  php/
    Dockerfile
public/
resources/
routes/
storage/
tests/
.env.example
.gitignore
compose.yaml
composer.json
composer.lock
package.json
package-lock.json
vite.config.js
```

## Security Notes

- Never commit `.env` or real database passwords.
- Generate a new `APP_KEY` for each project.
- Use unique database credentials for each project.
- Keep `APP_DEBUG=true` restricted to local development.
- Do not expose local development services publicly.
- The built-in PHP development server and Vite server are not intended for production deployments.

## License

This starter is available under the MIT license.

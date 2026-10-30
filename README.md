# Laravel Docker Starter

A reusable **Laravel 13 development starter** powered by Docker Compose, PHP 8.4, MySQL 8.4, Node.js 24, Vite, and Tailwind CSS.

Designed to simplify Laravel development by providing a consistent, isolated, and configurable Docker environment.

Create a new project from this repository using **GitHub's "Use this template"** feature.

## Tech Stack

| Technology | Version | Purpose |
|---|---|---|
| Laravel | 13 | PHP framework |
| PHP | 8.4 | Backend runtime |
| MySQL | 8.4 | Database |
| phpMyAdmin | 5.2 | Database management |
| Node.js | 24 | Frontend tooling |
| Vite | Project lockfile | Frontend asset bundling and HMR |
| Tailwind CSS | Project lockfile | CSS framework |
| Docker Compose | V2 | Container orchestration |

## Requirements

- Docker Desktop with Docker Compose
- WSL 2 with Ubuntu (recommended on Windows)
- VS Code with WSL extension (recommended)
- Git

**PHP, Composer, MySQL and Node.js do not need to be installed directly on the host.**

## Getting Started

### 1. Create a new project

Open the [Laravel Docker Starter](https://github.com/farid-larav/laravel-docker-starter) repository.

Click **Use this template → Create a new repository**.

Choose a repository name, then clone your new project:

```bash
git clone https://github.com/farid-larav/laravel-docker-starter.git
cd YOUR_PROJECT
```

Each generated repository has its own Git history and can be developed independently.

### 2. Configure environment variables

Create your local environment file:

```bash
cp .env.example .env
```

Open `.env` and configure the following values:

```dotenv
# Laravel
APP_NAME="My Laravel App"
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:8000

# Docker Compose
COMPOSE_PROJECT_NAME=my_laravel_project

# Docker host ports
APP_PORT=8000
MYSQL_FORWARD_PORT=3307
PHPMYADMIN_PORT=8080
VITE_PORT=5173

# Host permissions
HOST_UID=1000
HOST_GID=1000

# MySQL
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel
DB_PASSWORD=your_local_db_password

MYSQL_ROOT_PASSWORD=your_local_root_password
```

Replace the example passwords with your own credentials.

Preserve the additional Laravel settings already present in `.env`.

**Important configuration notes:**

- `COMPOSE_PROJECT_NAME` must be unique for each Laravel project.
- `APP_URL` should match the configured `APP_PORT`.
- `APP_KEY` must remain empty until a new application key is generated.
- `DB_HOST=mysql` refers to the MySQL service inside Docker.
- `DB_PORT=3306` is the internal MySQL port, regardless of the published host port.
- `VITE_PORT` controls the Vite development server and browser HMR connection.

#### Configure Linux file permissions

For Ubuntu WSL users, run:

```bash
id -u
id -g
```

Set `HOST_UID` and `HOST_GID` in `.env` to match the returned values.

Docker Compose reads these variables from the project's `.env` file.

### 3. Build the PHP image

```bash
docker compose build app
```

This builds the PHP 8.4 image with the required Laravel extensions and Composer.

### 4. Install PHP dependencies

```bash
docker compose run --rm --no-deps \
  -e COMPOSER_HOME=/tmp/composer \
  app composer install
```

Dependencies are installed using `composer.lock`.

### 5. Install frontend dependencies

```bash
docker compose run --rm --no-deps node npm ci
```

Dependencies are installed using `package-lock.json`.

### 6. Generate the application key

```bash
docker compose run --rm --no-deps \
  app php artisan key:generate
```

This generates a unique Laravel application key for your new project.

### 7. Start Docker containers

```bash
docker compose up -d
```

Check the status:

```bash
docker compose ps
```

The following services should be running:

- `app` — Laravel development server
- `mysql` — MySQL database
- `phpmyadmin` — Database management interface
- `node` — Vite development server

MySQL should become healthy after initialization.

### 8. Run database migrations

```bash
docker compose exec app php artisan migrate
```

**Note:** MySQL initialization credentials only take effect when the database volume is first created.

### 9. Open the application

With the default environment configuration:

| Service | Address |
|---|---|
| Laravel | http://localhost:8000 |
| phpMyAdmin | http://localhost:8080 |
| Vite | http://localhost:5173 |
| MySQL (host access) | `127.0.0.1:3307` |

To log in to phpMyAdmin, use the `DB_USERNAME` and `DB_PASSWORD` defined in your `.env` file.

## Docker Configuration

Docker Compose ports and project identifiers are configurable through `.env`.

The default values are:

| Variable | Default | Description |
|---|---|---|
| `COMPOSE_PROJECT_NAME` | `my_laravel_project` | Unique Docker project identifier |
| `APP_PORT` | `8000` | Laravel host HTTP port |
| `MYSQL_FORWARD_PORT` | `3307` | MySQL host port |
| `PHPMYADMIN_PORT` | `8080` | phpMyAdmin host HTTP port |
| `VITE_PORT` | `5173` | Vite server and HMR port |
| `HOST_UID` | `1000` | Host Linux user ID (example) |
| `HOST_GID` | `1000` | Host Linux group ID (example) |

The default UID and GID are examples. Adjust them to match your development environment.

### Running multiple Laravel projects

Multiple Laravel projects can run simultaneously using different Docker Compose project names and host ports.

**Example: Project A**

```dotenv
COMPOSE_PROJECT_NAME=laravel_project_a

APP_URL=http://localhost:8000

APP_PORT=8000
MYSQL_FORWARD_PORT=3307
PHPMYADMIN_PORT=8080
VITE_PORT=5173
```

**Example: Project B**

```dotenv
COMPOSE_PROJECT_NAME=laravel_project_b

APP_URL=http://localhost:8001

APP_PORT=8001
MYSQL_FORWARD_PORT=3308
PHPMYADMIN_PORT=8081
VITE_PORT=5174
```

| Service | Project A | Project B |
|---|---|---|
| Laravel | localhost:8000 | localhost:8001 |
| MySQL | localhost:3307 | localhost:3308 |
| phpMyAdmin | localhost:8080 | localhost:8081 |
| Vite | localhost:5173 | localhost:5174 |

Each project has independent:

- Docker containers
- Docker networks
- Compose-managed named volumes
- MySQL database storage
- Laravel environment configuration

**No modifications to `compose.yaml` or `vite.config.js` are required.**

Only the project's `.env` configuration needs to change.

### Internal versus external ports

Docker containers communicate through their internal network.

Laravel connects to MySQL using:

```dotenv
DB_HOST=mysql
DB_PORT=3306
```

An external database client running on your host uses:

```text
Host: 127.0.0.1
Port: 3307
```

The host port comes from `MYSQL_FORWARD_PORT`.

Changing the published MySQL port does not require changing `DB_PORT` inside Laravel.

## Frontend Development

The Node.js container runs the Vite development server.

By default:

```dotenv
VITE_PORT=5173
```

Vite automatically reads the configured port.

Its development server and Hot Module Replacement (HMR) connection use the same configured port.

For another project, you can use:

```dotenv
VITE_PORT=5174
```

The Docker port mapping and Vite server configuration will adjust automatically.

### Hot Module Replacement

Vite supports automatic browser updates when frontend files change.

You can edit:

- Blade templates
- Tailwind CSS styles
- JavaScript modules
- Vue or other frontend components, if installed

Vite handles supported hot updates, while Laravel's Vite plugin can trigger browser refreshes for configured files.

You do not need to rebuild the frontend assets for every development change.

### Build production frontend assets

```bash
docker compose run --rm --no-deps node npm run build
```

This generates optimized frontend assets using Vite.

## Common Docker Commands

| Task | Command |
|---|---|
| Start services | `docker compose up -d` |
| Stop services | `docker compose down` |
| Check status | `docker compose ps` |
| View logs | `docker compose logs -f` |
| View Laravel logs | `docker compose logs -f app` |
| View Vite logs | `docker compose logs -f node` |
| Run Artisan | `docker compose exec app php artisan` |
| Run migrations | `docker compose exec app php artisan migrate` |
| Run tests | `docker compose exec app php artisan test` |
| Install PHP package | `docker compose exec app composer require vendor/package` |
| Install NPM package | `docker compose exec node npm install package` |
| Build frontend assets | `docker compose run --rm --no-deps node npm run build` |
| Validate Docker Compose | `docker compose config --quiet` |

## Development Workflow

Laravel uses PHP's built-in development server inside the `app` container.

By default, the server listens on internal port `8000` and is published through `APP_PORT`.

Node.js runs Vite using the configured `VITE_PORT`.

MySQL and phpMyAdmin run in separate containers.

For Windows development, keep the project inside the Ubuntu WSL filesystem and open it using VS Code with the WSL extension.

This provides a convenient development environment without requiring a local PHP, Composer, Node.js or MySQL installation.

## Database Persistence

MySQL data is stored in a Docker named volume.

Stopping the environment:

```bash
docker compose down
```

removes the containers and Compose network but preserves the database volume.

Restart with:

```bash
docker compose up -d
```

The existing database data remains available.

**Warning:** Running the following command removes Compose-managed named volumes, including the MySQL data volume:

```bash
docker compose down -v
```

This can permanently delete development database data.

### Changing MySQL credentials

The MySQL Docker image uses initialization variables when it creates a new database volume.

Changing `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` or `MYSQL_ROOT_PASSWORD` in `.env` does not automatically update an existing initialized database.

For an existing database, update credentials through MySQL administration or deliberately recreate the test database volume when its data is no longer needed.

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
README.md
```

Additional standard Laravel files and directories may be present.

## Security Notes

- Never commit `.env`, credentials or API keys.
- Use `.env.example` with placeholder values only.
- Generate a new `APP_KEY` for each project.
- Use unique database credentials for each application.
- Keep `APP_DEBUG=true` restricted to local development.
- Use a unique `COMPOSE_PROJECT_NAME` for every local project.
- Keep development ports bound to localhost unless remote access is explicitly required.
- Do not expose MySQL, phpMyAdmin or development servers publicly.
- The built-in PHP development server and Vite server are not intended for production deployment.

## License

<<<<<<< HEAD
This project is intended to be distributed under the MIT License. See the `LICENSE` file when included.
=======
This starter is available under the MIT license.
>>>>>>> 2cda0060b96eb8b08818c91b46a8f55077df8126

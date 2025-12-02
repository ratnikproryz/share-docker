
## Laravel multi-project Docker environment

This repository provides a reusable Docker setup for running **multiple Laravel projects** on a single stack using:

- **PHP 8.2 (FPM, Alpine)**
- **Apache 2.4**
- **MySQL 8**
- **Redis 6**
- **MinIO (S3 compatible)**

All Laravel apps live under the `src` directory and are served via separate virtual hosts (e.g. `project01.local`, `project02.local`) through the same Docker Compose stack.

---

## 1. Directory structure

- `src/`
  - `project01/` (Laravel project)
  - `project02/` (Laravel project)
- `docker/`
  - `php/` – PHP 8.2 FPM Dockerfile + config
  - `apache/` – Apache Dockerfile + vhost configs
  - `mysql/` – MySQL config (`my.cnf`)
  - `redis/` – Redis config
  - `minio/` – MinIO data volume (created at runtime)
- `docker-compose.yml` – Definition for all services

Each Laravel project must follow the standard structure (`public/` directory, etc.).

---

## 2. First-time setup

1. **Clone this repo** (or copy into your project root).

2. **Create or copy Laravel projects** into `src/`:

   ```bash
   cd /Users/phuc.pham/workspace/share-docker
   mkdir -p src
   # Example: create a new Laravel project
   # composer create-project laravel/laravel src/project01
   # composer create-project laravel/laravel src/project02
   ```

3. **Configure local hostnames**  
   Edit `/etc/hosts` (macOS/Linux) and add:

   ```text
   127.0.0.1   project01.local project02.local
   ```

   You can add more hostnames if you add more projects (e.g. `project03.local`).

---

## 3. Running the stack

From the project root:

```bash
docker compose up -d --build
```

This starts:

- `database` – MySQL 8
- `app` – PHP 8.2 FPM
- `web` – Apache 2.4
- `redis` – Redis
- `mailhog` – Mail testing UI
- `minio` – S3-compatible storage

Check containers:

```bash
docker compose ps
```

Stop everything:

```bash
docker compose down
```

---

## 4. Accessing your Laravel projects

By default, Apache (`web` service) is published on host port **8000**:

- `http://project01.local:8000` → serves `src/project01/public`
- `http://project02.local:8000` → serves `src/project02/public`

You can configure additional projects by:

1. Adding a new Laravel app under `src/projectXX`.
2. Adding a matching `VirtualHost` block in `docker/apache/config/All/laravel-multi.conf`.
3. Adding the hostname to `/etc/hosts`.

Example mapping in Apache config (already provided for `project01` and `project02`):

- `ServerName project01.local` → `/var/www/project01/public`
- `ServerName project02.local` → `/var/www/project02/public`

The `src` directory on the host is mounted into the containers as `/var/www`.

---

## 5. Database (MySQL 8)

The `database` service is configured in `docker-compose.yml`:

- **Host (from app container)**: `database`
- **Default port**: `3306`
- **Default env (overridable via `.env`)**:
  - `DB_NAME` (default: `project_database`)
  - `DB_USER` (default: `project`)
  - `DB_PASS` (default: `project123!db`)

Example `.env` for `project01` / `project02`:

```env
DB_CONNECTION=mysql
DB_HOST=database
DB_PORT=3306
DB_DATABASE=project_database
DB_USERNAME=project
DB_PASSWORD=project123!db
```

On the host, MySQL is exposed at `${DB_PORT:-3306}` (default `3306`), so you can connect from a GUI:

- Host: `127.0.0.1`
- Port: `3306`

---

## 6. Redis

The `redis` service is available to Laravel apps as:

- **Host**: `redis`
- **Port**: `6379`

Example `.env`:

```env
REDIS_HOST=redis
REDIS_PORT=6379
```

On the host, Redis is mapped to `${REDIS_PORT:-6379}` (default `6379`).

---

## 7. MinIO (S3-compatible)

The `minio` service runs with:

- **API (S3)**: `${MINIO_SERVER_PORT:-9000}` → default `9000`
- **Console UI**: `${MINIO_PORT:-9001}` → default `9001`

Default credentials (overridable via env):

- `MINIO_ROOT_USER=user`
- `MINIO_ROOT_PASSWORD=password`

You can access:

- MinIO console: `http://localhost:9001`

Example Laravel `.env` S3 section (aligned with `env` keys from `config/filesystems.php` / `.env.example`), using MinIO:

```env
FILESYSTEM_DISK=s3

AWS_ACCESS_KEY_ID=user
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=your-bucket-name
AWS_URL=
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

Remember to create the bucket in the MinIO console (using the same `your-bucket-name`) and update `AWS_BUCKET`.

---

## 8. Common troubleshooting

- **cURL works but browser fails with `ERR_TUNNEL_CONNECTION_FAILED`**  
  - This is usually a **proxy/VPN/DNS issue** on your machine, not Docker.
  - Verify `curl http://project01.local:8000` works.
  - Check your system/browser proxy settings and VPN.
  - Try `http://localhost:8000` or change domains to `.test` (e.g. `project01.test`).

- **Permissions on `storage` / `bootstrap/cache`**  
  - Inside `app` container:

    ```bash
    docker exec -it docker-multi-app sh
    cd /var/www/project01
    php artisan storage:link
    chown -R www-data:www-data storage bootstrap/cache
    ```

---

## 9. Extending this setup

- Add more Laravel apps by placing them under `src/` and adding a new `VirtualHost` in `laravel-multi.conf`.
- Customize PHP extensions or settings by editing `docker/php/Dockerfile` and `docker/php/php.ini`.
- Add more services (e.g. Elasticsearch, RabbitMQ) by extending `docker-compose.yml`.

This environment is intended as a **shared local dev stack** for multiple Laravel projects; you can adapt it further for staging or production as needed.

---

## 10. Mail (Mailhog)

The `mailhog` service is included for local email testing.

- **SMTP host (inside Docker network)**: `mailhog`
- **SMTP port (inside Docker network)**: `1025`
- **MailHog web UI (from host)**: `http://localhost:8026` (by default)

Example Laravel `.env` mail section (per project):

```env
MAIL_MAILER=smtp
MAIL_HOST=mailhog
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```



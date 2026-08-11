# Server Storage

Storage stack with **PostgreSQL**, **Redis**, and **MinIO** — all run via Docker Compose.

| Service    | Default port | Purpose                          |
|------------|--------------|----------------------------------|
| PostgreSQL | 5432         | Relational database              |
| Redis      | 6379         | Cache / broker (password auth)   |
| MinIO API  | 9010         | S3-compatible object storage     |
| MinIO Console | 9011      | MinIO web UI                     |

MinIO host ports default to `9010` / `9011` so they do not clash with Portainer (`9000`) on a shared host with [server-monitor](https://github.com/arubaya/server-monitor).

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/) (usually included with Docker Desktop)

## Usage

### 1. Clone & enter the folder

```bash
git clone <repo-url>
cd server-storage
```

### 2. Configure environment

```bash
cp .env.example .env
```

Edit `.env` (users, passwords, ports). Compose reads `.env` automatically. Empty values fall back to the defaults in `docker-compose.yml`.

### 3. Start the stack

```bash
docker compose up -d
```

Check status:

```bash
docker compose ps
```

### 4. Connect

- **PostgreSQL:** `postgresql://<POSTGRES_USER>:<POSTGRES_PASSWORD>@localhost:<POSTGRES_PORT>/<POSTGRES_DB>`
  - Defaults: `postgres` / `postgres` / `5432` / `postgres`
- **Redis:** `redis://:<REDIS_PASSWORD>@localhost:<REDIS_PORT>/0`
  - Defaults: password `changeme`, port `6379`
- **MinIO API:** http://localhost:9010 (or `MINIO_API_PORT`)
- **MinIO Console:** http://localhost:9011 (or `MINIO_CONSOLE_PORT`)
  - User / password: from `.env` (`MINIO_ROOT_*`); defaults `minioadmin` / `minioadmin`

### 5. Stop / remove

```bash
# stop without deleting data
docker compose down

# stop and remove all named volumes (deletes DB, Redis AOF, MinIO objects)
docker compose down -v
```

## Structure

```
server-storage/
├── .env.example         # copy → .env
├── docker-compose.yml
└── README.md
```

## Notes

- Default credentials are fine for local use only — change them in `.env` before deploying to a public server. `.env` is gitignored.
- PostgreSQL and Redis have no TLS in this stack. Do not expose ports `5432` / `6379` to the internet without a firewall or SSH tunnel. Prefer binding to loopback if the host is internet-facing, e.g. `POSTGRES_PORT=127.0.0.1:5432`.
- MinIO root password must be at least **8** characters; root user at least **3**.
- Data lives in Docker named volumes: `postgres-data`, `redis-data`, `minio-data`.
- Redis runs with AOF persistence (`appendonly yes`) and `requirepass`.

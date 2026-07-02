# Redis Setup

This setup runs Redis locally using Docker Compose for development.

## Folder Structure

```text
redis/
├── README.md
├── docker-compose.yml
└── .env.example
```

## docker-compose.yml

```yaml
services:
  redis:
    image: redis:8.6
    container_name: redis
    restart: unless-stopped

    ports:
      - "${REDIS_PORT}:6379"

    command:
      - redis-server
      - --appendonly
      - "yes"

    volumes:
      - redis_data:/data

    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 10

volumes:
  redis_data:
```

## .env.example

```env
REDIS_PORT=6379
```

Create your local `.env`:

```bash
cp .env.example .env
```

## Start Redis

```bash
docker compose up -d
```

## Stop Redis

```bash
docker compose down
```

## View Logs

```bash
docker compose logs -f redis
```

## Connect from Local Application

```env
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_URL=redis://localhost:6379
```

## Test Redis

Open Redis CLI:

```bash
docker exec -it redis redis-cli
```

Test connection:

```text
PING
```

Expected output:

```text
PONG
```

Example commands:

```text
SET username "john"
GET username
DEL username
KEYS *
```

## Reset Redis

Remove all data:

```bash
docker compose down -v
```

Start again:

```bash
docker compose up -d
```

## Git Ignore

Add:

```gitignore
.env
```

Push:

* docker-compose.yml
* .env.example
* README.md

Do not push:

* .env

## Notes

* Redis listens on port **6379**.
* Data is persisted in the `redis_data` Docker volume.
* `--appendonly yes` enables AOF persistence so data survives container restarts.

# PostgreSQL + pgAdmin Setup

This setup runs PostgreSQL and pgAdmin locally using Docker Compose.

## Folder Structure

```txt
postgres/
├── README.md
├── docker-compose.yml
└── .env.example
```

## docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:18
    container_name: postgres18
    restart: unless-stopped

    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}

    ports:
      - "${POSTGRES_PORT}:5432"

    volumes:
      - postgres_data:/var/lib/postgresql/data

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5

  pgadmin:
    image: dpage/pgadmin4:9.15
    container_name: pgadmin4
    restart: unless-stopped

    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD}

    ports:
      - "${PGADMIN_PORT}:80"

    volumes:
      - pgadmin_data:/var/lib/pgadmin

    depends_on:
      postgres:
        condition: service_healthy

volumes:
  postgres_data:
  pgadmin_data:
```

## .env.example

```env
POSTGRES_USER=admin
POSTGRES_PASSWORD=admin123
POSTGRES_DB=mydb
POSTGRES_PORT=5432

PGADMIN_DEFAULT_EMAIL=admin@example.com
PGADMIN_DEFAULT_PASSWORD=admin123
PGADMIN_PORT=5050
```

Create your local `.env` file:

```bash
cp .env.example .env
```

Do not push `.env` to Git.

## Start Services

```bash
docker compose up -d
```

## Stop Services

```bash
docker compose down
```

## View Logs

```bash
docker compose logs -f postgres
```

```bash
docker compose logs -f pgadmin
```

## Open pgAdmin

Open:

```txt
http://localhost:5050
```

Login using:

```txt
Email: admin@example.com
Password: admin123
```

## Add PostgreSQL Server in pgAdmin

After logging into pgAdmin, add a new server.

### General Tab

```txt
Name: Local PostgreSQL
```

### Connection Tab

```txt
Host: postgres
Port: 5432
Maintenance database: mydb
Username: admin
Password: admin123
```

Important: use `postgres` as the host inside pgAdmin because both containers are running in the same Docker Compose network.

## Connect from Local Application

When connecting from your Node.js app running on your machine, use:

```env
DATABASE_URL=postgresql://admin:admin123@localhost:5432/mydb
```

When connecting from another Docker container in the same Compose network, use:

```env
DATABASE_URL=postgresql://admin:admin123@postgres:5432/mydb
```

## Access PostgreSQL CLI

```bash
docker exec -it postgres18 psql -U admin -d mydb
```

Test query:

```sql
SELECT version();
```

Exit psql:

```bash
\q
```

## Reset Database Data

This removes PostgreSQL and pgAdmin data.

```bash
docker compose down -v
```

Then start again:

```bash
docker compose up -d
```

## Git Ignore

Add this to `.gitignore`:

```gitignore
.env
```

You can push:

```txt
docker-compose.yml
.env.example
README.md
```

Do not push:

```txt
.env
```

## Notes

* PostgreSQL data is persisted in the `postgres_data` Docker volume.
* pgAdmin data is persisted in the `pgadmin_data` Docker volume.
* `depends_on` with `service_healthy` makes pgAdmin wait until PostgreSQL is ready.
* For local development, default credentials are acceptable.
* For production, use strong passwords and a managed database service when possible.

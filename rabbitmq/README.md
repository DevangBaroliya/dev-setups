# RabbitMQ Setup

This setup runs RabbitMQ with the Management UI using Docker Compose.

## Folder Structure

```text
rabbitmq/
├── README.md
├── docker-compose.yml
└── .env.example
```

## docker-compose.yml

```yaml
services:
  rabbitmq:
    image: rabbitmq:4-management
    container_name: rabbitmq
    restart: unless-stopped

    ports:
      - "${RABBITMQ_AMQP_PORT}:5672"
      - "${RABBITMQ_MANAGEMENT_PORT}:15672"

    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_DEFAULT_USER}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_DEFAULT_PASS}

    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 5s
      timeout: 5s
      retries: 10

volumes:
  rabbitmq_data:
```

## .env.example

```env
RABBITMQ_AMQP_PORT=5672
RABBITMQ_MANAGEMENT_PORT=15672

RABBITMQ_DEFAULT_USER=admin
RABBITMQ_DEFAULT_PASS=admin123
```

Create your local `.env`:

```bash
cp .env.example .env
```

## Start RabbitMQ

```bash
docker compose up -d
```

## Stop RabbitMQ

```bash
docker compose down
```

## View Logs

```bash
docker compose logs -f rabbitmq
```

## RabbitMQ Management UI

Open:

```text
http://localhost:15672
```

Login using:

```text
Username: admin
Password: admin123
```

## Connect from Local Application

```env
RABBITMQ_URL=amqp://admin:admin123@localhost:5672
```

## Connect from Docker

```env
RABBITMQ_URL=amqp://admin:admin123@rabbitmq:5672
```

## Test RabbitMQ

Check health:

```bash
docker exec -it rabbitmq rabbitmq-diagnostics ping
```

List users:

```bash
docker exec -it rabbitmq rabbitmqctl list_users
```

List queues:

```bash
docker exec -it rabbitmq rabbitmqctl list_queues
```

List exchanges:

```bash
docker exec -it rabbitmq rabbitmqctl list_exchanges
```

## Reset RabbitMQ

Remove all queues, exchanges, and messages:

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

* AMQP runs on port **5672**.
* Management UI runs on port **15672**.
* Data is persisted in the `rabbitmq_data` Docker volume.
* The Management UI is intended for development and administration.
* Use the `amqp://` connection string in your applications.

# MongoDB Setup

This setup runs MongoDB locally using Docker Compose and connects to it using MongoDB Compass.

## Folder Structure

```txt
mongodb/
├── README.md
├── docker-compose.yml
└── .env.example
```

## docker-compose.yml

```yaml
services:
  mongodb:
    image: mongo:8
    container_name: mongodb
    restart: unless-stopped

    ports:
      - "${MONGO_PORT}:27017"

    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}
      MONGO_INITDB_DATABASE: ${MONGO_INITDB_DATABASE}

    volumes:
      - mongodb_data:/data/db

    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mongodb_data:
```

## .env.example

```env
MONGO_PORT=27017

MONGO_INITDB_ROOT_USERNAME=admin
MONGO_INITDB_ROOT_PASSWORD=admin123
MONGO_INITDB_DATABASE=mydb
```

Create your local `.env` file:

```bash
cp .env.example .env
```

## Start MongoDB

```bash
docker compose up -d
```

## Stop MongoDB

```bash
docker compose down
```

## View Logs

```bash
docker compose logs -f mongodb
```

## MongoDB Compass Connection

Open MongoDB Compass and use this connection string:

```txt
mongodb://admin:admin123@localhost:27017/mydb?authSource=admin
```

Then click **Connect**.

## MongoDB Compass Manual Setup

Use these values if you want to fill the connection form manually:

```txt
Hostname: localhost
Port: 27017
Authentication: Username / Password
Username: admin
Password: admin123
Authentication Database: admin
Default Database: mydb
```

Important:

```txt
authSource=admin
```

is required because the root user is created in the `admin` database.

## Connect from Local Application

Use this when your Node.js app runs directly on your machine:

```env
MONGODB_URI=mongodb://admin:admin123@localhost:27017/mydb?authSource=admin
```

## Connect from Docker Container

Use this when your app runs inside the same Docker Compose network:

```env
MONGODB_URI=mongodb://admin:admin123@mongodb:27017/mydb?authSource=admin
```

## Access MongoDB Shell

```bash
docker exec -it mongodb mongosh -u admin -p admin123 --authenticationDatabase admin
```

Switch to your database:

```js
use mydb
```

Insert test data:

```js
db.users.insertOne({
  name: "John",
  email: "john@example.com"
})
```

Read data:

```js
db.users.find()
```

Exit shell:

```bash
exit
```

## Reset MongoDB Data

This removes all MongoDB data:

```bash
docker compose down -v
```

Start again:

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

* MongoDB runs on port `27017`.
* Data is persisted in the `mongodb_data` Docker volume.
* Use `localhost` when your app runs directly on your machine.
* Use `mongodb` as the host when your app runs inside Docker.
* Use MongoDB Compass to visually browse databases, collections, and documents.
* `authSource=admin` is required for the root user created by Docker.

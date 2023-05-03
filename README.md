# metabase-docker-container

Easily deploy Metabase using Docker container with this simple setup. It targets port 80, allowing you to access the Metabase instance directly through your IP address in a browser.

## Prerequisites
- Docker and Docker Compose installed on your system.

## Set Up

1. Create a directory called Metabase

```shell
mkdir metabase
cd metabase
docker-compose pull
docker-compose up
```

2. Create a ´docker-compose.yml´ file with the following content:

```yaml
version: '3'
services:
  metabase:
    image: metabase/metabase
    ports:
      - 80:3000
    environment:
      MB_DB_TYPE: postgres
      MB_DB_DBNAME: metabase
      MB_DB_PORT: 5432
      MB_DB_USER: metabase
      MB_DB_PASS: metabase
      MB_DB_HOST: postgres
  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: metabase
      POSTGRES_DB: metabase
      POSTGRES_PASSWORD: metabase
    volumes:
      - ./pg:/var/lib/postgresql/data
```

3. Pull the required Docker image

```shell
docker-compose pull
```

4. Start the Metabase and PostgreSQL containers:

```shell
docker-compose up
```

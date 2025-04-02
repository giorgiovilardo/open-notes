## Start

```shell
docker run --name yourName -e POSTGRES_PASSWORD=yourPw --rm -d -p 5432:5432 postgres:latest
```

## Stop

```shell
docker kill yourName
```

## Shell

```shell
docker exec -u postgres -it yourName bash

docker exec -u postgres -it yourName psql
```

## Seed from sql

```shell
cat yourFile.sql | docker exec -u postgres -i yourName pg_restore --dbname=yourDb --single-transaction --no-owner --disable-triggers
```
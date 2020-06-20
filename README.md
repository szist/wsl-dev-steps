# wsl-dev-steps
Some notes to get up and running with development inside wsl

Assumes wsl2 setup after the 2020 may update. [wsl2](https://docs.microsoft.com/en-us/windows/wsl/wsl2-index)

**Basic setup Ubuntu**
```
$ sudo apt-get install build-essential
```

## File system access

From windows navigate to `\\wsl$\` You will see your distributions.

## docker

Is useful to set up databases, other services that are project-related.

The new docker for windows actually uses wsl 2 as backend. Make sure to enable wsl integration for your distributions.
[link](https://code.visualstudio.com/blogs/2020/03/02/docker-in-wsl2)

## NodeJS

The same approach works as with any linux distribution.
[nvm-setup](https://github.com/nvm-sh/nvm#install--update-script)

## Elixir

Follow steps on [elixir website](https://elixir-lang.org/install.html#unix-and-unix-like)

## Postgresql

**Dependencies**: docker for windows with wsl integration

### Imperative approach

```
>$ docker create -p 127.0.0.1:5432:5432 -e POSTGRES_PASSWORD=pass --name <project name> postgres
>$ docker start <project name>

# optional if there is no migration library
>$ docker exec -it <project name> bash
># psql -U postgres
postgres=# CREATE DATABASE <project DB>;
postgres=# \q

# manipulate with it
>$ docker start <project name>
>$ docker stop <project name>
```

### Declarative approach (docker-compose)

Create `docker-compose.dev.yml`
```
version: '3.1'

services:
  db:
    image: postgres
    restart: always
    volumes:
        - './data/db/postgresql/data:/var/lib/postgresql/data'
        - './init-db-scripts:/docker-entrypoint-initdb.d:ro'
    ports:
        - "5432"
    environment:
        POSTGRES_PASSWORD=pass
```
The `init-db-scripts` volume is optional. It can be used for init scripts. From the previous example `init-db-scripts/01-init-db.sh`:
```
#!/bin/bash
set -e

psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
    CREATE DATABASE <project DB>;
EOSQL
```

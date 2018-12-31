# keycloak-tools

This project contains tools used to automate keycloak's deployment process.

## Submodules

| folder | description |
|--------|-------------|
| config-cli | tool to configure keycloak via json files |
| encrypt-password-cli | tool to encrypt a password     |

### Config-CLI

Tool to configure keycloak via json files

#### Config files

The config files are based on the keycloak export files. You can use them to re-import your settings.
But keep your files as small as possible. Remove all UUIDs and all stuff which is default set by keycloak.

[moped.json](./example-config/moped.json) is a full working example file you can consider.
 Other examples are located in the [test resources](./config-cli/src/test/resources/import-files).

## Build this project

```bash
$ mvn package
```

## Run this project

### via Maven

Start a local keycloak on port 8080:

```bash
$ docker-compose down --remove-orphans && docker-compose up keycloak
``` 

before performing following command:

```bash
$ java -jar ./config-cli/target/config-cli.jar --keycloak.url=http://localhost:8080 --keycloak.password=admin123 --import.file=./example-config/moped.json
```

### Docker

#### Docker run

```
$ docker run -e KEYCLOAK_URL=http://<your keycloak host>:8080 \
             -e KEYCLOAK_ADMIN=<keycloak admin username> \
             -e KEYCLOAK_ADMIN_PASSWORD=<keycloak admin password> \
             -e KEYCLOAK_ENCRYPT_ADMIN_PASSWORD=<true/false> \
             -e JWKS_CONNECT_TIMEOUT=250 \
             -e JWKS_READ_TIMEOUT=250 \
             -e JWKS_SIZE_LIMIT=51200 \
             -v <your config path>:/opt/keycloak-config-cli/configs \
             adorsys/keycloak-config-cli:latest \
             config-cli
```

#### Docker-compose

```
version: '3.1'
services:
  keycloak:
    image: jboss/keycloak:4.4.0.Final
    environment:
      KEYCLOAK_USER: <keycloak admin username>
      KEYCLOAK_PASSWORD: <keycloak admin password>
    ports:
    - "8080:8080"
    networks:
    - my_network
    command:
    - "-b"
    - "0.0.0.0"
    - "--debug"
  keycloak_config:
    image: adorsys/keycloak-config-cli:latest
    depends_on:
    - keycloak
    links:
    - keycloak
    volumes:
    - <your config path>:/tmp/keycloak-config-cli/configs
    environment:
    - KEYCLOAK_URL=http://<your keycloak host>:8080/auth
    - KEYCLOAK_ADMIN=<keycloak admin username>
    - KEYCLOAK_ADMIN_PASSWORD=<keycloak admin password>
    - KEYCLOAK_ENCRYPT_ADMIN_PASSWORD=<true/false>
    - JWKS_CONNECT_TIMEOUT=250
    - JWKS_READ_TIMEOUT=250
    - JWKS_SIZE_LIMIT=51200
    depends_on:
    - keycloak
    networks:
    - my_network
    command: config-cli

networks:
  my_network:

```

## Integration tests

```bash
$ mvn verify
```

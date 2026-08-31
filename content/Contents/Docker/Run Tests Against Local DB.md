
1. Set up Docker-Compose
```
version: "3.9"

  

services:

test_db:

container_name: test_db

restart: always

build:

context: .

dockerfile: ./docker/Dockerfile-DB

environment:

# Explicitly passing required environment variables to container

MYSQL_DATABASE: test_db

MYSQL_USER: admin

MYSQL_ROOT_PASSWORD: password

MYSQL_PASSWORD: password

command: --max_allowed_packet=90000000

ports:

- '3309:3306'

healthcheck:

# wait for database availability

test: /etc/init.d/mysql status

interval: 1s

timeout: 30s

retries: 120

  

app_server:

container_name: app_server

tty: true

restart: on-failure

build:

context: .

dockerfile: ./docker/Dockerfile-TEST

env_file:

- ./docker/compose.env

ports:

- 8080:8080

depends_on:

- test_db
```
2. Set up environmental variables
```
version: "3.9"

  

services:

test_db:

container_name: test_db

restart: always

build:

context: .

dockerfile: ./docker/Dockerfile-DB

environment:

# Explicitly passing required environment variables to container

MYSQL_DATABASE: test_db

MYSQL_USER: admin

MYSQL_ROOT_PASSWORD: password

MYSQL_PASSWORD: password

command: --max_allowed_packet=90000000

ports:

- '3309:3306'

healthcheck:

# wait for database availability

test: /etc/init.d/mysql status

interval: 1s

timeout: 30s

retries: 120

  

app_server:

container_name: app_server

tty: true

restart: on-failure

build:

context: .

dockerfile: ./docker/Dockerfile-TEST

environment:

# Explicitly passing required environment variables to container

DATABASE_DIALECT: mysql+pymysql

DATABASE_HOST: test_db

DATABASE_PORT: 3306

MASTER_USERNAME: root

MASTER_PASSWORD: password

DATABASE_NAME: test_db

env_file:

- ./docker/compose.env

ports:

- 8080:8080

depends_on:

- test_db
```
3. Test connection
```
 docker exec -t app_server poetry run pytest -v
```
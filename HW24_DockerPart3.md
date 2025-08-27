# Docker и контейнеризация №3
> Цель: получить практический опыт работы c Docker-compose

## Task 1 Запустить приложение golang-app_01
> Склонируем репу `git clone https://gitlab.com/dos-29-onl/docker-apps.git`
> Выведем дерево проекта
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_golang-app_01$ tree -a
.
├── compose.yml
└── golang-app_01
    ├── Dockerfile
    ├── .env
    ├── go.mod
    ├── go.sum
    ├── main.go
    └── README.md
```
> Выведем содержимое compose.yml
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_golang-app_01$ cat compose.yml
services:
  database:
    image: postgres:17
    container_name: golang_postgres
    restart: always
    volumes:
      - pg_data:/var/lib/postgresql/data
    env_file:
      - ./golang-app_01/.env

  app:
    build:
      context: ./golang-app_01
      dockerfile: Dockerfile
    container_name: golang_app
    restart: always
    env_file:
      - ./golang-app_01/.env
    depends_on:
      - database
    ports:
      - "5000:5000"

volumes:
  pg_data:
```
> Выведем содержимое Dockerfile
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_golang-app_01$ cat golang-app_01/Dockerfile
FROM golang:1.24.4 AS builder
WORKDIR /app
COPY . .
RUN go mod tidy
RUN go build -o server .

FROM gcr.io/distroless/base-debian12
WORKDIR /app
COPY --from=builder /app/server .
ENTRYPOINT ["/app/server"]
```
> Выведем содержимое .env
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_golang-app_01$ cat golang-app_01/.env
POSTGRES_USER=goappuser
POSTGRES_PASSWORD=qwer/.,m
POSTGRES_DB=goapp
PG_HOST=golang_postgres
PG_PORT=5432
PG_DATABASE=goapp
PG_USER=goappuser
PG_PASSWORD=qwer/.,m
```
> Соберем и поднимем контейнеры
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_golang-app_01$ docker compose up -d
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_golang-app_01$ docker compose ps
NAME              IMAGE                   COMMAND                  SERVICE    CREATED          STATUS          PORTS
golang_app        prj_golang-app_01-app   "/app/server"            app        14 minutes ago   Up 14 minutes   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp
golang_postgres   postgres:17             "docker-entrypoint.s…"   database   14 minutes ago   Up 14 minutes   5432/tcp
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_golang-app_01$ curl "http://localhost:5000/generate-load?n=5000000"
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_golang-app_01$ curl http://localhost:5000/metrics
# HELP go_gc_duration_seconds A summary of the wall-time pause (stop-the-world) duration in garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0.002004964
go_gc_duration_seconds{quantile="0.25"} 0.002004964
go_gc_duration_seconds{quantile="0.5"} 0.002004964
go_gc_duration_seconds{quantile="0.75"} 0.002004964
...
много всяких метрик
...
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_golang-app_01$ curl http://localhost:5000/error
Server Error
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_golang-app_01$ curl http://localhost:5000
Main Page
```
## Task2 Запустить приложение через docker compose, которое записывает данные в Memcached и сразу же читает их обратно, демонстрируя работу с кэшированием. Каждый запрос сохраняет текущую метку времени в кэше и возвращает ее в ответе.

## Требования
1) Node.js не ниже 12
2) Memcached сервер
3) Nginx в роли reverse proxy

> Дерево проекта
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_01$ tree
.
├── compose.yml
├── nginx.conf
└── nodejs-app_01
    ├── app.js
    ├── Dockerfile
    ├── package.json
    └── README.md
```
> Выведем compose.yml
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_01$ cat compose.yml
services:
  webserver:
    image: nginx:1.24
    container_name: reverse_proxy
    ports:
      - 8080:80
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    restart: always
    depends_on:
      app:
        condition: service_started

  app:
    build:
      context: ./nodejs-app_01
      dockerfile: Dockerfile
    restart: always
    depends_on:
      - memcached

  memcached:
    image: memcached
```
> nginx.conf
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_01$ cat nginx.conf
upstream backend {
  server app:3000;
}

server {
    listen 80;
    server_name app;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    location /api {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
> Dockerfile
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_01$ cat nodejs-app_01/Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN npm install
ENTRYPOINT ["npm", "start"]
```
> В коде app.js 'localhost:11211' пришлось поменять на 'memcached:11211', так как он в отдельном контейнере. (увидел попытки соединения с 127.0.0.1 только после curl в docker compose logs app)

> Проверим работоспособность
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_01$ docker compose up -d
[+] Running 5/5
 ✔ prj_nodejs-app_01-app                    Built                                                                                                                                                      0.0s
 ✔ Network prj_nodejs-app_01_default        Created                                                                                                                                                    0.0s
 ✔ Container prj_nodejs-app_01-memcached-1  Started                                                                                                                                                    0.1s
 ✔ Container prj_nodejs-app_01-app-1        Started                                                                                                                                                    0.2s
 ✔ Container reverse_proxy                  Started
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_01$ curl http://localhost:8080
{"message":"Данные успешно записаны и прочитаны из Memcached!","cached":{"timestamp":1756206911073},"status":"success"}
```

## Task 3 Запустить приложение через docker compose, которое gозволяет добавлять и просматривать информацию о серверах в базе данных MySQL.
## Требования
1) Node.js не ниже 14
2) MySQL (mariadb)
3) Nginx в роли reverse proxy

> Дерево проекта
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_02$ tree -a
.
├── compose.yml
├── nginx.conf
└── nodejs-app_02
    ├── app.js
    ├── cmdb-data.json
    ├── Dockerfile
    ├── .env
    ├── package.json
    └── README.md
```
> Выведем compose.yml
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_02$ cat compose.yml
services:
  webserver:
    image: nginx:1.24
    container_name: reverse_proxy
    ports:
      - 8080:80
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    restart: always
    depends_on:
      app:
        condition: service_started

  app:
    build:
      context: ./nodejs-app_02
      dockerfile: Dockerfile
    restart: always
    env_file:
      - ./nodejs-app_02/.env
    depends_on:
      database:
        condition: service_healthy


  database:
    container_name: mariadb
    image: mariadb:10.9
    restart: always
    env_file:
      - ./nodejs-app_02/.env
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "mysqluser", "-pmypassword"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db_data:
```

> nginx.conf тупо скопирован с прошлого задания
> Dockerfile
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_02$ cat nodejs-app_02/Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN npm install
ENTRYPOINT ["node", "app.js"]
```
> Выведем содержимое .env
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_02$ cat nodejs-app_02/.env
      DB_HOST=database
      DB_USER=mysqluser
      DB_PASSWORD=mypassword
      DB_NAME=nodejsbd

      MARIADB_ROOT_PASSWORD=rootpassword
      MARIADB_DATABASE=nodejsbd
      MARIADB_USER=mysqluser
      MARIADB_PASSWORD=mypassword
```
> Выполним сборку
```
docker compose up -d --build
[+] Running 6/6
 ✔ prj_nodejs-app_02-app               Built                                                                                                                                                           0.0s
 ✔ Network prj_nodejs-app_02_default   Created                                                                                                                                                         0.0s
 ✔ Volume "prj_nodejs-app_02_db_data"  Created                                                                                                                                                         0.0s
 ✔ Container mariadb                   Healthy                                                                                                                                                        10.7s
 ✔ Container prj_nodejs-app_02-app-1   Started                                                                                                                                                        10.8s
 ✔ Container reverse_proxy             Started
```
> При проверке выяснили что тело файла cmdb-data.json, которое передается в POST запросе не соответствует
```
app.post('/api/servers', async (req, res) => {
  const { hostname, ip_address, role } = req.body;
```
> тут должно передаваться только 3 поля поэтому файл cmdb-data.json будет выглядеть так
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_02$ cat nodejs-app_02/cmdb-data.json
{
    "role": "Web Server 01",
    "hostname": "HP ProLiant DL380",
    "ip_address": "192.168.1.10"
  }
```
> теперь сделаем POST запрос и следом за ним GET
```
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_02$ curl -X POST -H "Content-Type: application/json" -d @nodejs-app_02/cmdb-data.json  http://localhost:8080/api/servers
{"id":1}
user@lab:~/skurat/docker/docker-apps/compose-apps/prj_nodejs-app_02$ curl -X GET http://localhost:8080/api/servers
[{"id":1,"hostname":"HP ProLiant DL380","ip_address":"192.168.1.10","role":"Web Server 01","created_at":"2025-08-27T10:31:11.000Z"}]
```
> Как видим данные в базу заносятся

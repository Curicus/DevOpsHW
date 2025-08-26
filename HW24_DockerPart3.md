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
## Task2 


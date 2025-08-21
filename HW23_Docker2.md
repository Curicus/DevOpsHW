# Docker и контейнеризация №2
> Цель: получить практический опыт работы c Docker-командами

## Task 1 Запустить приложение в контейнере php-app
> Склонируем репозиторий `user@lab:~$ git clone https://gitlab.com/dos-29-onl/docker-apps.git`

> Создадим Dockerfile
```
user@lab:~/docker-apps/php-app$ vim Dockerfile
  1 FROM php:8.2-apache
  2 COPY index.php /var/www/html
```
> Сделаем сборку образа 
```
user@lab:~/docker-apps/php-app$ docker build -t php-app .
[+] Building 13.0s (7/7) FINISHED
```
> Запустим контейнер с именем php, замапим порт 8080  
```
user@lab:~/docker-apps/php-app$ docker run -d -p 8080:80 --name php php-app
34b4ee580050dce6588d53b21234dc7f78ed19ebcfa289495dd44f4b050a0c1e
user@lab:~/docker-apps/php-app$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                     NAMES
34b4ee580050   php-app   "docker-php-entrypoi…"   14 seconds ago   Up 13 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   php
```
> Проверим работу
```
user@lab:~/docker-apps/php-app$ curl 127.0.0.1:8080
<h1>Hello, Docker and PHP!</h1><p>Сейчас: <strong>21.08.2025 08:36</strong></p><p>Ваш браузер: <strong>curl/7.81.0</strong></p><form method="POST">
    <label>Ваше имя: <input name="name" /></label>
    <button type="submit">Отправить</button>
```
<img width="1041" height="319" alt="image" src="https://github.com/user-attachments/assets/a8f62c4c-d84e-4746-a5d4-2de1b6b6af07" />

## Task 2 Запустить приложение в контейнере python-app
> Создадим Dockerfile
```
user@lab:~/docker-apps/python-app$ cat Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
ENTRYPOINT ["gunicorn", "app:app", "--bind", "0.0.0.0:5000"]
```
> Создадим отдельный файл .env в котором зададим переменные окружения и будем запускать контейнер, подсовывая этот файл
```
user@lab:~/docker-apps/python-app$ cat .env
APP_ENV=production
DEBUG=false
API_VERSION=v1
```
> Запустим контейнер подсунув файл с переменными в .env, меняя их можно влиять на поведение 
```
user@lab:~/docker-apps/python-app$ docker run --env-file .env -d -p 5000:5000 --name python python-app
05f0dca13fc0acdb1784f8ce71ace9cc94de39087cf71b9d3f1929a7ec9b2a48
user@lab:~/docker-apps/python-app$ docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                                         NAMES
05f0dca13fc0   python-app   "gunicorn app:app --…"   7 seconds ago   Up 6 seconds   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp   python
```










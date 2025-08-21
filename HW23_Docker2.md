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


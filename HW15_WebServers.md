# WebServers
> Цель: получить практический опыт установки, настройки и использования веб-сервера Nginx.

## Task 1 Установите и настройте Nginx. Создайте html-страничку, где будет указана ваша фамилия и тема урока. Настройте конфигурацию nginx для отображения страницы по ссылке http://tms.by
> Создадим каталог для нашей страницы, а в нем саму html страницу
```
sudo mkdir -p /var/www/tms.by
sudo vim /var/www/tms.by/index.html
sudo chmod -R 755 /var/www/tms.by
sudo chmod 644 /var/www/tms.by/index.html
sudo chown -R www-data:www-data /var/www/tms.by
```
```
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Моя страница</title>
</head>
<body>
    <h1>Скурат</h1>
    <p>Тема урока: Настройка Nginx</p>
</body>
</html>
```
> Создадим конфиг файл для nginx
```
sudo vim /etc/nginx/sites-available/tms.by
server {
    listen 80;
    server_name tms.by;
    root /var/www/tms.by;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

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
На клиентской машине добавим запись в файл hosts - > 192.168.1.174 tms.by
<img width="517" height="209" alt="image" src="https://github.com/user-attachments/assets/986a44ff-0b2e-43a8-947d-44f9ec27179a" />


## Task 2 additional task

**2.1 Запустить на ВМ как systemd сервис приложение на python**
> Установка Python и Flask и настройка окружения
```
sudo apt install -y python3 python3-pip python3-venv
python3 -m venv venv
source venv/bin/activate
pip install flask
```
> Скопируем код для приложения на python в файл /opt/flaskapp/app.py
> Создадим Unit файл для сервиса
```
[Unit]
Description=Flask Test App
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/opt/flaskapp
Environment="PATH=/opt/flaskapp/venv/bin"
ExecStart=/opt/flaskapp/venv/bin/python /opt/flaskapp/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```
> перечитаем и запустим
```
sudo systemctl daemon-reload
sudo systemctl enable flaskapp
sudo systemctl start flaskapp
```

**2.2 Создать директорию /var/www/static, в ней создать файлы index.html и style.js**
> Ну собственно создадим эти файлы вставим код и задидим следующие права:
```
user@Lab5:/var/www/static$sudo chown www-data:www-data index.html 
user@Lab5:/var/www/static$sudo chown www-data:www-data style.js
user@Lab5:/var/www/static$sudo chmod 755 index.html
user@Lab5:/var/www/static$sudo chmod 755 style.js
```
> А ну и скачали лого `wget -O /var/www/static/nginx-logo.png https://nginx.org/nginx.png`

**2.3 Настроим nginx для работы как proxy**
> Добавим в nginx конфиг для нашей страницы в /etc/nginx/sites-available/tms.by
```
server {
    listen 80;
    server_name tms.by;

    location /static/ {
        alias /var/www/static/;
        autoindex off;
    }

    # Проксирование всех остальных запросов на Flask
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
 }
```
> Активируем страницу добавлением семилинка
```
sudo ln -s /etc/nginx/sites-available/tms.by /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```
> В итоге у меня только JSON ответ без стилей, но для него надо как то отдельно вживлять
### Скрин 1
<img width="452" height="395" alt="image" src="https://github.com/user-attachments/assets/8eead8c2-74a6-4e6d-a709-360d11e4361c" />

### Скрин 2
<img width="409" height="301" alt="image" src="https://github.com/user-attachments/assets/ec3cec41-f42a-4cfa-b79e-b7714dde83e1" />  

### Скрин 3
<img width="434" height="156" alt="image" src="https://github.com/user-attachments/assets/e720721b-c316-446a-a327-e8206a6f9729" />  

### Скрин 4
<img width="1756" height="757" alt="image" src="https://github.com/user-attachments/assets/1f154721-b91c-4359-aa2d-856ae4806e27" />






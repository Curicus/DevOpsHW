# Операционные системы №4
## Task 1 
добавить в cron скрипт/команду,которая будет очищать кэш apt (кэшируемые пакеты,
пакеты,которые не могут быть загружены) раз в месяц в 16 часов.
### Выполнение
> Проверим объем кэша apt
```
user@Lab5:~$ sudo du -sh /var/cache/apt/archives
[sudo] password for user:
16K     /var/cache/apt/archives
```
> Добавим расписание в cron
```
user@Lab5:~$ sudo crontab -e

# Edit this file to introduce tasks to be run by cron.
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
0 16 1 * * apt clean && apt autoremove        #Данная задача запустится следующий раз 1 июля в 16.00 и будет выполняться каждый месяц 1-го числа в это время
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
```
> Запустим выполнение задачи вручную
```
user@Lab5:~$ sudo apt clean && sudo apt autoremove
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
user@Lab5:~$ sudo du -sh /var/cache/apt/archives
16K     /var/cache/apt/archives
```
> Объем занятого пространства не изменился, видимо пока чистить нечего

## Task 2 запустить демон nodejs-приложения через systemd.

### Nodejs
> Устновим node.js  ` sudo apt install node.js`
Скопируем код в файл /home/user/HW5/Nodejs






### Создаем Unit file
```
[Unit]
Description=My App on Node.js
After=network.target

[Service]
Environment=MYAPP_PORT=3000
Type=simple
User=user
Group=user
ExecStart=/usr/bin/node /home/user/HW5/nodejs
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target

```
### Запуск службы и проверка статуса
```
user@Lab5:~/HW5$ sudo systemctl daemon-reload
user@Lab5:~/HW5$ sudo systemctl start myapp
user@Lab5:~/HW5$ sudo systemctl enable myapp  # чтобы служба запускалась с запуском ОС
Created symlink /etc/systemd/system/multi-user.target.wants/myapp.service → /etc/systemd/system/myapp.service.
user@Lab5:~/HW5$ sudo systemctl status myapp
● myapp.service - My App on Node.js
     Loaded: loaded (/etc/systemd/system/myapp.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-06-16 10:37:51 UTC; 3min 31s ago
   Main PID: 2677 (node)
      Tasks: 10 (limit: 2267)
     Memory: 12.0M (peak: 15.1M)
        CPU: 446ms
     CGroup: /system.slice/myapp.service
             └─2677 /usr/bin/node /home/user/HW5/nodejs

Jun 16 10:37:51 Lab5 systemd[1]: Started myapp.service - My App on Node.js.
```
> Проверим curl запросами
```
user@Lab5:~/HW5$ curl http://localhost:3000
200 OKuser@Lab5:~/HW5$ curl -X POST http://localhost:3000
405 Method Not Alloweduser@Lab5:~/HW5$ curl http://localhost:3000/kill
curl: (52) Empty reply from server
user@Lab5:~/HW5$ sudo systemctl status myapp
● myapp.service - My App on Node.js
     Loaded: loaded (/etc/systemd/system/myapp.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-06-16 10:44:30 UTC; 15s ago
   Main PID: 2863 (node)
      Tasks: 10 (limit: 2267)
     Memory: 12.0M (peak: 14.9M)
        CPU: 465ms
     CGroup: /system.slice/myapp.service
             └─2863 /usr/bin/node /home/user/HW5/nodejs

Jun 16 10:44:27 Lab5 systemd[1]: myapp.service: Failed with result 'exit-code'.
Jun 16 10:44:30 Lab5 systemd[1]: myapp.service: Scheduled restart job, restart counter is at 1.
Jun 16 10:44:30 Lab5 systemd[1]: Started myapp.service - My App on Node.js.
```
> Как мы видим даже после kill сервис перезапустился.



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
0 16 1 * * apt clean && apt autoclean        #Данная задача запустится следующий раз 1 июля в 16.00 и будет выполняться каждый месяц 1-го числа в это время
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
> Запустим выполнение задачи вручную
```
user@Lab5:~$ sudo apt clean && sudo apt autoclean
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
user@Lab5:~$ sudo du -sh /var/cache/apt/archives
16K     /var/cache/apt/archives
```
> Объем занятого пространства не изменился, видимо пока чистить нечего

## Task 2 запустить демон nodejs-приложения через systemd.

















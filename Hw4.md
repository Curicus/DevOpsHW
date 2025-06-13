# Операционные системы № 3
Цель: получить практический опыт установки пакетов с помощью
сторонних репозиториев и пакетного менеджера. Научиться
выполнять задачи автоматизации с помощью Bash.
## Task1 Установить MongoDB
### Сделаем импорт публичного GPG ключа для MongoDB

```
user@lab4:~$ curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor
```
### Создаем файл list и добавляем туда репозиторий для MongoDB
```
user@lab4:~$ echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse
```
проверим:
```
user@lab4:~$ sudo apt update
...
Get:8 https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 InRelease [3,005 B]
Get:9 https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0/multiverse arm64 Packages [37.8 kB]
Get:10 https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0/multiverse amd64 Packages [37.7 kB]
...
```
### Произведем установку MongoDB и проверим статус
```
user@lab4:~$ sudo apt install -y mongodb-org
```
![image](https://github.com/user-attachments/assets/a0df7f2e-a5e3-428e-aa9e-a4c062fd7826)

> Включим режим аутентификации `user@lab4:~$ sudo vim /etc/mongod.conf`
```
# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo

security:
  authorization: enabled

#operationProfiling:
```
> Сделаем перезапуск службы `user@lab4:~$ sudo systemctl restart mongod`


### Работа с MongoDB
Подключение к MongoDB `user@lab4:~$ mongosh` и создадим БД
```
test> use Database
switched to db Database
```
Создадим коллекцию data
```
Database> db.createCollection("data")
{ ok: 1 }
```
Создаем пользователя manager
```
Database> db.createUser({user: "manager", pwd:"parol",roles:[{role:"read", db:"Database"}]})
{ ok: 1 }
```
*Проверим права пользователя manager*
>залогинимся и попробуем внести изменения в БД
```
user@lab4:~$ mongosh -u manager -p parol --authenticationDatabase Database
test> use Database
Database> db.data.insertOne({proverka: 2})
MongoServerError[Unauthorized]: not authorized on Database to execute command { insert: "data", documents: [ { proverka: 2, _id: ObjectId('684ac15293489c95ce69e328') } ], ordered: true, lsid: { id: UUID("bdcfd982-ace7-4707-827d-7f0d0fa7b0eb") }, $db: "Database" }
Database>
```
>Получили ошибку.

## Task 3 Bash script №1
Содержание скрипта: замена существующего расширения в имени файла на заданное. Исходное имя файла и новое расширение передаются скрипту в
качестве параметров. Основное средство: нестандартное раскрытие переменных. Усложнение: предусмотреть штатную реакцию на отсутствие
расширения в исходном имени файла.
### Содержание скрипта
```
#! /usr/bin/env bash

filename="$1"
extention="$2"

#проверка на существование файла
if [ ! -e "$filename" ]; then
    echo "File $filename does not exist"
    exit 1
fi

#проверка наличия расширения в файле
name="${filename%.*}"

if [ "$filename" != "$name" ]; then
    echo "Filename without extension: "$name""
else
    echo " $filename has no extension"
    exit 0
fi

new_filename="${name}.${extention}"
mv "$filename" "$new_filename"
echo "Old filename: "$filename"  New filename: "$new_filename" "
```
###  Результаты исполнения
>вводная информация
```
user@lab4:~/bash$ ls -l
total 4
-rw-rw-r-- 1 user user   0 Jun 13 09:26 filefortest
-rw-rw-r-- 1 user user   0 Jun 13 08:40 filefortest.gz
-rwxrw-r-- 1 user user 553 Jun 13 09:06 script.sh
```
> Проверка на отсутствие файла
```
user@lab4:~/bash$ sudo ./script.sh file.gz ppt
File file.gz does not exist
```
> Проверка на отсутствие расширения
```
user@lab4:~/bash$ sudo ./script.sh filefortest ppt
 filefortest has no extension
```
> Полная работа
```
user@lab4:~/bash$ sudo ./script.sh filefortest.gz ppt
Filename without extension: filefortest
Old filename: filefortest.gz  New filename: filefortest.ppt
```

## Task4 Bash script №2

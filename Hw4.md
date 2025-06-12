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

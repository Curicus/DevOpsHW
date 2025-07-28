# Базы данных №1
> Цель - поулчить практический опыт написания SQL-запросов
## Task 1 Создание бызы Medcenter и сделать выборку:
Формулировка: вывести название и цену для всех анализов, которые продавались 5 февраля 2020 и всю следующую неделю.
> Войдем в оснастку MariaDB `sudo mysql -u root -p${DB_PASS}`
> Создадим базу данных 
MariaDB [(none)]> create database medcenter;
MariaDB [(none)]> use medcenter;

> Создадим таблицы и наполним их данными
```
MariaDB [medcenter]> CREATE TABLE Analysis (an_id INT PRIMARY KEY, an_name VARCHAR(255) NOT NULL, an_cost DECIMAL(10, 2) NOT NULL, an_price DECIMAL(10, 2) NOT NULL, an_group INT,FOREIGN KEY (an_group) REFERENCES Groups(gr_id));
MariaDB [medcenter]> CREATE TABLE Groups (gr_id INT PRIMARY KEY, gr_name VARCHAR(255) NOT NULL, gr_temp VARCHAR(50) NOT NULL);
MariaDB [medcenter]> CREATE TABLE Orders (ord_id INT PRIMARY KEY, ord_datetime DATETIME NOT NULL, ord_an INT, FOREIGN KEY (ord_an) REFERENCES Analysis(an_id));
MariaDB [medcenter]> INSERT INTO Analysis (an_id, an_name, an_cost, an_price, an_group) VALUES (1, 'Анализ 1', 100.00, 150.00, 1), (2, 'Анализ 2', 200.00, 250.00, 2), (3, 'Анализ 3', 50.00, 75.00, 3), (4, 'Анализ 4', 300.00, 350.00, 1), (5, 'Анализ 5', 400.00, 450.00, 2);
MariaDB [medcenter]> INSERT INTO Groups (gr_id, gr_name, gr_temp) VALUES (1, 'Группа 1', '+4°C'), (2, 'Группа 2', '+20°C'), (3, 'Группа 3', '+20°C');
MariaDB [medcenter]> INSERT INTO Orders (ord_id, ord_datetime, ord_an) VALUES (1, '2020-02-05 10:00:00', 1), (2, '2020-02-06 12:00:00', 2), (3, '2020-02-07 14:00:00', 3), (4, '2020-02-08 16:00:00', 1), (5, '2020-02-09 18:00:00', 4), (6, '2020-02-10 20:00:00', 5), (7, '2020-02-11 22:00:00', 2), (8, '2020-02-12 00:00:00', 3);
MariaDB [medcenter]> select A.an_name, A.an_price from Analysis A JOIN Orders O on A.an_id = O.ord_an where O.ord_datetime between '2020-02-05 00:00:00' and '2020-02-12 23:59:59';
+----------------+----------+
| an_name        | an_price |
+----------------+----------+
| Анализ 1       |   150.00 |
| Анализ 2       |   250.00 |
| Анализ 3       |    75.00 |
| Анализ 1       |   150.00 |
| Анализ 4       |   350.00 |
| Анализ 5       |   450.00 |
| Анализ 2       |   250.00 |
| Анализ 3       |    75.00 |
+----------------+----------+
```

## Task 2 
Шаги:
1. Создайте бэкап базы данных. Для этого используйте команду "mysqldump" для создания полного дампа базы данных. Сохраните файл дампа в безопасном месте, таком как внешнией жесткий диск или облачное хранилище.
2.  Измените какие-либо данные в базе данных, например, добавьте новую таблицу или обновите информацию в существующей таблице.
3.  Восстановите базу данных из бэкапа, чтобы вернуть ее в исходное состояние. Для этого используйте команду "mysql" и укажите имя базы данных и файл дампа для восстановления.
4.  Убедитесь, что база данных была восстановлена успешно, проверив данные и таблицы в базе данных.
5.  Создайте скрипт, который будет автоматически создавать бэкап базы данных и отправлять его на удаленный сервер для хранения. Например, вы можете использовать инструмент "cron" Для регулярного создания бэкапов и передачи их на удаленный сервер по расписанию.

> Сделаем бэкап базы 
```
user@Lab5  /tmp  mysqldump -u user -p${DB_PASS} medcenter > /tmp/medcenter_2025_07_28.sql`
user@Lab5  /tmp  rclone copy /tmp/medcenter_2025-07-28.sql gdrive:backup
```
> Наш дамп появился на гугл диске
<img width="522" height="400" alt="image" src="https://github.com/user-attachments/assets/d2bf19dd-7c7e-4d7e-b8da-c91a2fc0d8eb" />
> Удалим таблицу Orders в БД
```
MariaDB [medcenter]> show tables;
+---------------------+
| Tables_in_medcenter |
+---------------------+
| Analysis            |
| Groups              |
| Orders              |
+---------------------+
3 rows in set (0.001 sec)

MariaDB [medcenter]> drop table Orders;
Query OK, 0 rows affected (0.003 sec)

MariaDB [medcenter]> show tables;
+---------------------+
| Tables_in_medcenter |
+---------------------+
| Analysis            |
| Groups              |
+---------------------+
2 rows in set (0.001 sec)

MariaDB [medcenter]>


> Теперь восстановим базу из дампа
 user@Lab5  /tmp  mysql -u user -p${DB_PASS} medcenter < /tmp/medcenter_2025-07-28.sql
MariaDB [medcenter]> show tables;
+---------------------+
| Tables_in_medcenter |
+---------------------+
| Analysis            |
| Groups              |
| Orders              |
+---------------------+
3 rows in set (0.000 sec)

> Как видим таблица Order вернулась обратно
MariaDB [medcenter]> describe Orders;
+--------------+----------+------+-----+---------+-------+
| Field        | Type     | Null | Key | Default | Extra |
+--------------+----------+------+-----+---------+-------+
| ord_id       | int(11)  | NO   | PRI | NULL    |       |
| ord_datetime | datetime | NO   |     | NULL    |       |
| ord_an       | int(11)  | YES  | MUL | NULL    |       |
+--------------+----------+------+-----+---------+-------+
3 rows in set (0.002 sec)
> структура тоже в порядке

MariaDB [medcenter]> select * from Orders;
+--------+---------------------+--------+
| ord_id | ord_datetime        | ord_an |
+--------+---------------------+--------+
|      1 | 2020-02-05 10:00:00 |      1 |
|      2 | 2020-02-06 12:00:00 |      2 |
|      3 | 2020-02-07 14:00:00 |      3 |
|      4 | 2020-02-08 16:00:00 |      1 |
|      5 | 2020-02-09 18:00:00 |      4 |
|      6 | 2020-02-10 20:00:00 |      5 |
|      7 | 2020-02-11 22:00:00 |      2 |
|      8 | 2020-02-12 00:00:00 |      3 |
+--------+---------------------+--------+
8 rows in set (0.002 sec)

> Ну и данные на месте

> Скрипт для создания бэкапа
  1 #!/bin/bash
  2 DATE=$(date +%F_%H_%M)
  3 DUMP="/tmp/db_$DATE.sql"
  4 mysqldump -u user -p${DB_PASS} medcenter > "$DUMP"
  5 rclone copy "$DUMP" gdrive:backup

> Добавим в `sudo crontab -e` строчку с расписанием `0 23 * * * /home/user/skurat/DB_backup.sh`
будем снимать дамп каждый день в 23:00:00


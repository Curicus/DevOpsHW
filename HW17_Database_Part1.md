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


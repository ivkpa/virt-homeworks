# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.  
`docker run --name virt-postgres -e POSTGRES_PASSWORD=123 -v "$(pwd)/postgres-data":/var/lib/postgresql/data -d postgres`  

Подключитесь к БД PostgreSQL используя `psql`.  
`docker exec -it <container_id> psql`  

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД  
`\l`  
- подключения к БД  
`\c postgres`  
- вывода списка таблиц  
`\d`  
- вывода описания содержимого таблиц  
`\d <table>`  
- выхода из psql  
`\q`  

## Задача 2

Используя `psql` создайте БД `test_database`.  
`create database test_database;`  

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.  
`psql postgresql://postgres:123@localhost/test_database < backup.sql`  

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.  


**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.
`select tablename, attname, avg_width from pg_stats where tablename = 'orders';`  
```
 tablename | attname | avg_width
-----------+---------+-----------
 orders    | id      |         4
 orders    | title   |        16
 orders    | price   |         4
(3 rows)
```

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.  
```
CREATE TABLE orders_1 (LIKE orders);
INSERT INTO orders_1 SELECT * FROM orders WHERE price > 499;
DELETE FROM orders WHERE price > 499;

CREATE TABLE orders_2 (LIKE orders);
INSERT INTO orders_2 SELECT * FROM orders WHERE price <= 499;
DELETE FROM orders WHERE price > 499;
```

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?  
Да, через PARTITION.

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.  
`pg_dump postgresql://postgres:123@localhost/test_database > backup.sql`  
Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?  
`title character varying(80) NOT NULL -> title character varying(80) NOT NULL UNIQUE`

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

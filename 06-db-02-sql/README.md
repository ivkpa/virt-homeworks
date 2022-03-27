# Домашнее задание к занятию "6.2. SQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

```yml

version: "3.7"
services:
  postgres:
    image: postgres:12
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
      PGDATA: "/var/lib/postgresql/data/pgdata"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./backup:/backup
    ports:
      - "5432:5432"
    networks:
      - postgres
    restart: unless-stopped

volumes:
  postgres-data:

networks:
  postgres:

```

## Задача 2

В БД из задачи 1:  
`docker exec -it -u postgres <container_id> psql` 
- создайте пользователя test-admin-user и БД test_db   
`CREATE USER "test-admin-user"; CREATE USER "test_db";`  
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)  
`\c test_db;`  
```
CREATE TABLE orders (
	id serial PRIMARY KEY,
	name VARCHAR (128),
	price INT
);

CREATE TABLE clients (
	id serial PRIMARY KEY,
	lastname VARCHAR (128),
	country VARCHAR (128),
        fk_order INT REFERENCES orders(id)
);

CREATE INDEX idx_client_country ON clients(country);
```
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db  
`GRANT ALL ON DATABASE test_db TO "test-admin-user";`  
- создайте пользователя test-simple-user  
`CREATE USER "test-simple-user";`  
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db  
`GRANT SELECT,INSERT,UPDATE,DELETE ON ALL TABLES IN SCHEMA public TO "test-simple-user";`  
`ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT,INSERT,UPDATE,DELETE ON TABLES TO "test-simple-user";`  

Таблица orders:
- id (serial primary key)
- наименование (string)
- цена (integer)

Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:
- итоговый список БД после выполнения пунктов выше  
```
                                     List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |       Access privileges
-----------+----------+----------+------------+------------+--------------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres                  +
           |          |          |            |            | postgres=CTc/postgres         +
           |          |          |            |            | "test-admin-user"=CTc/postgres
(4 rows)

```
- описание таблиц (describe)  
```
                                     Table "public.clients"
  Column  |          Type          | Collation | Nullable |               Default
----------+------------------------+-----------+----------+-------------------------------------
 id       | integer                |           | not null | nextval('clients_id_seq'::regclass)
 lastname | character varying(128) |           |          |
 country  | character varying(128) |           |          |
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "idx_client_country" btree (country)
Foreign-key constraints:
    "fk_order" FOREIGN KEY (id) REFERENCES orders(id)

                                    Table "public.orders"
 Column |          Type          | Collation | Nullable |              Default
--------+------------------------+-----------+----------+------------------------------------
 id     | integer                |           | not null | nextval('orders_id_seq'::regclass)
 name   | character varying(128) |           |          |
 price  | integer                |           |          |
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "fk_order" FOREIGN KEY (id) REFERENCES orders(id)


```
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db  
`SELECT * FROM information_schema.role_table_grants WHERE table_catalog='test_db';`  

- список пользователей с правами над таблицами test_db
```
test_db=# \l
                                     List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |       Access privileges
-----------+----------+----------+------------+------------+--------------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres                  +
           |          |          |            |            | postgres=CTc/postgres         +
           |          |          |            |            | "test-admin-user"=CTc/postgres
(4 rows)

test_db=# \z clients
                                     Access privileges
 Schema |  Name   | Type  |        Access privileges         | Column privileges | Policies
--------+---------+-------+----------------------------------+-------------------+----------
 public | clients | table | postgres=arwdDxt/postgres       +|                   |
        |         |       | "test-simple-user"=arwd/postgres |                   |
(1 row)

test_db=# \z orders
                                     Access privileges
 Schema |  Name  | Type  |        Access privileges         | Column privileges | Policies
--------+--------+-------+----------------------------------+-------------------+----------
 public | orders | table | postgres=arwdDxt/postgres       +|                   |
        |        |       | "test-simple-user"=arwd/postgres |                   |
(1 row)
```
## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы 
- приведите в ответе:
    - запросы 
    - результаты их выполнения.  
    
```
test_db=# INSERT INTO orders(name, price) VALUES('Шоколад', 10),('Принтер',3000),('Книга',500),('Монитор',7000),('Гитара',4000);
INSERT 0 5
test_db=# INSERT INTO clients(lastname, country, fk_order) VALUES('Иванов Иван Иванович', 'USA', NULL),('Петров Петр Петрович', 'Canada', NULL),
test_db-# ('Иоганн Себастьян Бах', 'Japan', NULL),('Ронни Джеймс Дио', 'Russia', NULL),('Ritchie Blackmore', 'Russia', NULL);
INSERT 0 5
test_db=# SELECT count(*) FROM clients;
 count
-------
     5
(1 row)

test_db=# SELECT count(*) FROM orders;
 count
-------
     5
(1 row)

```
## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
 
Подсказк - используйте директиву `UPDATE`.
```
test_db=# UPDATE clients SET fk_order=(SELECT id FROM orders WHERE name='Книга') WHERE lastname='Иванов Иван Иванович';
UPDATE 1
test_db=# UPDATE clients SET fk_order=(SELECT id FROM orders WHERE name='Монитор') WHERE lastname='Петров Петр Петрович';
UPDATE 1
test_db=# UPDATE clients SET fk_order=(SELECT id FROM orders WHERE name='Гитара') WHERE lastname='Иоганн Себастьян Бах';
UPDATE 1
test_db=# SELECT * FROM clients WHERE fk_order IS NOT NULL;
 id |       lastname       | country | fk_order
----+----------------------+---------+----------
  1 | Иванов Иван Иванович | USA     |        3
  2 | Петров Петр Петрович | Canada  |        4
  3 | Иоганн Себастьян Бах | Japan   |        5
(3 rows)

```
## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

```
test_db=# EXPLAIN SELECT * FROM clients WHERE fk_order IS NOT NULL;
                       QUERY PLAN
--------------------------------------------------------
 Seq Scan on clients  (cost=0.00..1.05 rows=3 width=47)
   Filter: (fk_order IS NOT NULL)
(2 rows)
```  
<b>Получившийся результат значит, что будет выполнено последовательное сканирование с фильтрацией каждой строки по признаку fk_order IS NOT NULL. Числа в скобках означают:  
1.Приблизительная стоимость запуска. Это время, которое проходит, прежде чем начнётся этап вывода данных, например для сортирующего узла это время сортировки.  
2. Приблизительная общая стоимость. Она вычисляется в предположении, что узел плана выполняется до конца, то есть возвращает все доступные строки. На практике родительский узел может досрочно прекратить чтение строк дочернего (см. приведённый ниже пример с LIMIT).  
3. Ожидаемое число строк, которое должен вывести этот узел плана. При этом так же предполагается, что узел выполняется до конца.  
4. Ожидаемый средний размер строк, выводимых этим узлом плана (в байтах).  </b>

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).  
`docker exec -it -u postgres <container_id> bash`  
`pg_dump test_db > /backup/test.sql`  
Остановите контейнер с PostgreSQL (но не удаляйте volumes).  
`docker-compose down`  
Поднимите новый пустой контейнер с PostgreSQL.  
`docker-compose up -d`  
Восстановите БД test_db в новом контейнере.  
`docker exec -it -u postgres <container_id> bash`  
`createdb test_db`  
`psql test_db < /backup/test.sql`  
Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

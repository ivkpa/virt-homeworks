# Домашнее задание к занятию "6.3. MySQL"

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.  
`Server version:         8.0.28 MySQL Community Server - GPL`  
Подключитесь к восстановленной БД и получите список таблиц из этой БД.  
`use test_db; show tables;`  
**Приведите в ответе** количество записей с `price` > 300.  
```
mysql> select * from orders where price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)
```
В следующих заданиях мы будем продолжать работу с данным контейнером.

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней 
- количество попыток авторизации - 3 
- максимальное количество запросов в час - 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"

```
CREATE USER 'test'@'localhost'
  IDENTIFIED WITH mysql_native_password BY 'test-pass'
  WITH MAX_QUERIES_PER_HOUR 100
  PASSWORD EXPIRE INTERVAL 180 DAY
  FAILED_LOGIN_ATTEMPTS 3 ATTRIBUTE '{"lastname": "Pretty", "name": "James"}';
```
Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.  
`GRANT SELECT ON test_db.* TO test@localhost;`  
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и 
**приведите в ответе к задаче**.  
```
mysql> select * from INFORMATION_SCHEMA.USER_ATTRIBUTES where user='test';
+------+-----------+-----------------------------------------+
| USER | HOST      | ATTRIBUTE                               |
+------+-----------+-----------------------------------------+
| test | localhost | {"name": "James", "lastname": "Pretty"} |
+------+-----------+-----------------------------------------+
1 row in set (0.00 sec)
```

## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.
```
mysql> SELECT TABLE_NAME, ENGINE FROM   information_schema.TABLES WHERE  TABLE_SCHEMA = 'test_db';  
+------------+--------+
| TABLE_NAME | ENGINE |
+------------+--------+
| orders     | InnoDB |
+------------+--------+
1 row in set (0.00 sec)
```  

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
- на `InnoDB`  

```
mysql> ALTER TABLE orders ENGINE = MyISAM;
Query OK, 5 rows affected (0.08 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW PROFILE;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000088 |
| Executing hook on transaction  | 0.000007 |
| starting                       | 0.000021 |
| checking permissions           | 0.000006 |
| checking permissions           | 0.000004 |
| init                           | 0.000011 |
| Opening tables                 | 0.000310 |
| setup                          | 0.000107 |
| creating table                 | 0.000708 |
| waiting for handler commit     | 0.000022 |
| waiting for handler commit     | 0.006132 |
| After create                   | 0.000322 |
| System lock                    | 0.000010 |
| copy to tmp table              | 0.000187 |
| waiting for handler commit     | 0.000009 |
| waiting for handler commit     | 0.000041 |
| waiting for handler commit     | 0.000036 |
| rename result table            | 0.000056 |
| waiting for handler commit     | 0.021057 |
| waiting for handler commit     | 0.000021 |
| waiting for handler commit     | 0.005636 |
| waiting for handler commit     | 0.000010 |
| waiting for handler commit     | 0.020385 |
| waiting for handler commit     | 0.000019 |
| waiting for handler commit     | 0.010584 |
| end                            | 0.012938 |
| query end                      | 0.004283 |
| closing tables                 | 0.000022 |
| waiting for handler commit     | 0.000021 |
| freeing items                  | 0.003941 |
| cleaning up                    | 0.000019 |
+--------------------------------+----------+
31 rows in set, 1 warning (0.00 sec)

mysql> ALTER TABLE orders ENGINE = InnoDB;
Query OK, 5 rows affected (0.10 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW PROFILE;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000067 |
| Executing hook on transaction  | 0.000006 |
| starting                       | 0.000023 |
| checking permissions           | 0.000006 |
| checking permissions           | 0.000004 |
| init                           | 0.000012 |
| Opening tables                 | 0.000198 |
| setup                          | 0.000052 |
| creating table                 | 0.000082 |
| After create                   | 0.045114 |
| System lock                    | 0.000022 |
| copy to tmp table              | 0.000164 |
| rename result table            | 0.000797 |
| waiting for handler commit     | 0.000012 |
| waiting for handler commit     | 0.005818 |
| waiting for handler commit     | 0.000009 |
| waiting for handler commit     | 0.027361 |
| waiting for handler commit     | 0.000020 |
| waiting for handler commit     | 0.006094 |
| waiting for handler commit     | 0.000010 |
| waiting for handler commit     | 0.005099 |
| end                            | 0.000779 |
| query end                      | 0.004629 |
| closing tables                 | 0.000015 |
| waiting for handler commit     | 0.001403 |
| freeing items                  | 0.004438 |
| cleaning up                    | 0.000028 |
+--------------------------------+----------+
27 rows in set, 1 warning (0.00 sec)

mysql> SHOW PROFILES;
+----------+------------+--------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                      |
+----------+------------+--------------------------------------------------------------------------------------------+
|        1 | 0.00112850 | SELECT TABLE_NAME, ENGINE FROM   information_schema.TABLES WHERE  TABLE_SCHEMA = 'test_db' |
|        2 | 0.08700975 | ALTER TABLE orders ENGINE = MyISAM                                                         |
|        3 | 0.10226200 | ALTER TABLE orders ENGINE = InnoDB                                                         |
+----------+------------+--------------------------------------------------------------------------------------------+
3 rows in set, 1 warning (0.00 sec)

```
## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных  
`innodb_flush_log_at_trx_commit = 2`  
- Нужна компрессия таблиц для экономии места на диске  
`innodb_file_per_table = 1`  
- Размер буффера с незакомиченными транзакциями 1 Мб  
`innodb_log_buffer_size = 1M`  
- Буффер кеширования 30% от ОЗУ  
`innodb_buffer_pool_size = 5G`  
- Размер файла логов операций 100 Мб  
`innodb_log_file_size = 100M`

Приведите в ответе измененный файл `my.cnf`.
```
[mysqld]
innodb_flush_log_at_trx_commit = 2
innodb_file_per_table = 1
innodb_log_buffer_size = 1M
innodb_buffer_pool_size = 5G
innodb_log_file_size = 100M
```

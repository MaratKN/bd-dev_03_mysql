# Домашнее задание к занятию 3. «MySQL»

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/blob/virt-11/additional/README.md).

## Задача 1

Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h`, получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с этим контейнером.



**Решение:**

Создаю Volume 
```bash
sudo docker volume create mysql_data
```
Запускаю MySQL-контейнер
```bash
sudo docker run --name mysql-instance -e MYSQL_ROOT_PASSWORD=Password -v mysql_data:/var/lib/mysql -d -p 3306:3306 mysql:8.0
```
Скачиваю backup
```bash
sudo wget https://raw.githubusercontent.com/netology-code/virt-homeworks/virt-11/06-db-03-mysql/test_data/test_dump.sql
```
![alt text](https://github.com/MaratKN/bd-dev_03_mysql/blob/main/1.png)
Подключаюсь к MySQL-контейнеру
```bash
sudo docker exec -it mysql-instance mysql -u root -pPassword
```
В интерактивном режиме MySQL создаю БД
```bash
CREATE DATABASE test_db;
```
Выхожу из интерактивного режиме MySQL и восстанавливаю из backup
```bash
sudo docker exec -i mysql-instance mysql -u root -pPassword test_db < test_dump.sql
```
![alt text](https://github.com/MaratKN/bd-dev_03_mysql/blob/main/2.png)
Снова подключаюсь к MySQL-контейнеру
```bash
sudo docker exec -it mysql-instance mysql -u root -pPassword
```
В интерактивном режиме MySQL переключаюсь на test_db
```bash
USE test_db;
```
Получаю статус БД
```bash
status;
```
Получаю список таблиц в БД
```bash
SHOW TABLES;
```
![alt text](https://github.com/MaratKN/bd-dev_03_mysql/blob/main/3.png)
Вывожу список записей с price > 300
```bash
SELECT * FROM orders WHERE price > 300;
```
![alt text](https://github.com/MaratKN/bd-dev_03_mysql/blob/main/4.png)



## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:

- плагин авторизации mysql_native_password
- срок истечения пароля — 180 дней 
- количество попыток авторизации — 3 
- максимальное количество запросов в час — 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James".

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test` и 
**приведите в ответе к задаче**.



**Решение:**

Создаю пользователя с паролем
```bash
CREATE USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY 'test-pass' PASSWORD EXPIRE INTERVAL 180 DAY FAILED_LOGIN_ATTEMPTS 3 ATTRIBUTE '{"name": "James", "last_name": "Pretty"}';
```
Предоставляю привелегии пользователю test на операции SELECT базы test_db
```bash
GRANT SELECT ON test_db.* TO 'test'@'%';
```
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получаю данные по пользователю test
```bash
SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER='test';
```
![alt text](https://github.com/MaratKN/bd-dev_03_mysql/blob/main/5.png)



## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`,
- на `InnoDB`.



**Решение:**

Подключаюсь к MySQL и активирую профилирование:
```bash
SET profiling = 1;
```

В таблице БД test_db используется engine InnoDB
```bash
SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES where TABLE_SCHEMA = 'test_db';
```
![alt text](https://github.com/MaratKN/bd-dev_03_mysql/blob/main/6.png)

Изменяю engine на MyISAM/InnoDB и вывожу время выполнения
```bash
ALTER TABLE orders ENGINE = MyISAM;
```
```bash
ALTER TABLE orders ENGINE = InnoDB;
```
![alt text](https://github.com/MaratKN/bd-dev_03_mysql/blob/main/7.png)


## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.



**Решение:**

Было:
```bash
bash-5.1# cat /etc/my.cnf

# For advice on how to change settings please see

# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]

#

# Remove leading # and set to the amount of RAM for the most important data

# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.

# innodb_buffer_pool_size = 128M

#

# Remove leading # to turn on a very important data integrity option: logging

# changes to the binary log between backups.

# log_bin

#

# Remove leading # to set options mainly useful for reporting servers.

# The server defaults are faster for transactions and fast SELECTs.

# Adjust sizes as needed, experiment to find the optimal values.

# join_buffer_size = 128M

# sort_buffer_size = 2M

# read_rnd_buffer_size = 2M

# Remove leading # to revert to previous value for default_authentication_plugin,

# this will increase compatibility with older clients. For background, see:

# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin

# default-authentication-plugin=mysql_native_password

skip-host-cache

skip-name-resolve

datadir=/var/lib/mysql

socket=/var/run/mysqld/mysqld.sock

secure-file-priv=/var/lib/mysql-files

user=mysql

pid-file=/var/run/mysqld/mysqld.pid

[client]

socket=/var/run/mysqld/mysqld.sock

!includedir /etc/mysql/conf.d/
```


Стало:

```bash
bash-5.1# cat /etc/my.cnf

# For advice on how to change settings please see

# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]

#

# Remove leading # and set to the amount of RAM for the most important data

# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.

# innodb_buffer_pool_size = 128M

#

# Remove leading # to turn on a very important data integrity option: logging

# changes to the binary log between backups.

# log_bin

#

# Remove leading # to set options mainly useful for reporting servers.

# The server defaults are faster for transactions and fast SELECTs.

# Adjust sizes as needed, experiment to find the optimal values.

# join_buffer_size = 128M

# sort_buffer_size = 2M

# read_rnd_buffer_size = 2M

# Remove leading # to revert to previous value for default_authentication_plugin,

# this will increase compatibility with older clients. For background, see:

# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin

# default-authentication-plugin=mysql_native_password

skip-host-cache

skip-name-resolve

datadir=/var/lib/mysql

socket=/var/run/mysqld/mysqld.sock

secure-file-priv=/var/lib/mysql-files

user=mysql

pid-file=/var/run/mysqld/mysqld.pid

[client]

socket=/var/run/mysqld/mysqld.sock

!includedir /etc/mysql/conf.d/

# Установка движка по умолчанию

default-storage-engine = InnoDB

# Компрессия таблиц для экономии места на диске

innodb_file_format = Barracuda

innodb_file_per_table = 1

innodb_compression_level = 6

# Размер буфера с незакомиченными транзакциями 1 Мб

innodb_log_buffer_size = 1M

# Буфер кеширования 30% от ОЗУ

innodb_buffer_pool_size = 2.457G

# Размер файла логов операций 100 Мб

innodb_log_file_size = 100M
```
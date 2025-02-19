### Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

### Ответ

---

### Задание 2

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

### Решение

Задание выполнено с использованием Docker.
Содержимое compose.yml:
```
version: '3.7'

services:
  mysql-master:
    image: 'mysql:8.0'
    container_name: mysql-master
    env_file: .env
    volumes:
      - ./master/my.cnf:/etc/my.cnf
      - ./master/master.sql:/docker-entrypoint-initdb.d/start.sql
    environment:
      - MYSQL_ROOT_PASSWORD:${MYSQL_ROOT_PASSWORD}
    ports:
      - 3306:3306

  mysql-slave:
    image: 'mysql:8.0'
    container_name: mysql-slave
    env_file: .env
    volumes:
      - ./slave/my.cnf:/etc/my.cnf
      - ./slave/slave.sql:/docker-entrypoint-initdb.d/start.sql
    environment:
      - MYSQL_ROOT_PASSWORD:${MYSQL_ROOT_PASSWORD}
    ports:
      - 3307:3306
    depends_on:
      - mysql-master
```

Содержимое master/my.cnf:
```
[mysqld]
server-id=1
binlog_format=ROW
log-bin
```

Содержимое slave/my.cnf:
```
[mysqld]
server-id=2
```

Содержимое master/master.sql
```sql
CREATE USER repl@'%' IDENTIFIED WITH mysql_native_password BY 'slavepass';
GRANT REPLICATION SLAVE ON *.* TO repl@'%';
```

Содержимое slave/slave.sql
```sql
CHANGE MASTER TO 
MASTER_HOST='mysql-master',
MASTER_USER='repl',
MASTER_PASSWORD='slavepass';
```

Поднимаю 2 контейнера с mysql:
```
ubuntu@vm2:~/mysqlrepl$ docker compose up -d
WARN[0000] /home/ubuntu/mysqlrepl/compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] Running 2/2
 ✔ Container mysql-master  Started                                                                                                                    
 ✔ Container mysql-slave   Started
ubuntu@vm2:~/mysqlrepl$ docker ps -a
CONTAINER ID   IMAGE       COMMAND                  CREATED        STATUS              PORTS                                                    NAMES
778f45e76956   mysql:8.0   "docker-entrypoint.s…"   22 hours ago   Up About a minute   33060/tcp, 0.0.0.0:3307->3306/tcp, [::]:3307->3306/tcp   mysql-slave
ced8d5d15fb9   mysql:8.0   "docker-entrypoint.s…"   22 hours ago   Up About a minute   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp     mysql-master
```

Проверяю настройки репликации на Master:
![alt text](https://github.com/masterchoo495/Repl_p1/blob/main/001.png)

Проверяю настройки репликации на Slave:
![alt text](https://github.com/masterchoo495/Repl_p1/blob/main/002.png)

Проверяю работоспособность репликации путем создания тестовой БД на Master:
![alt text](https://github.com/masterchoo495/Repl_p1/blob/main/003.png)

Проверяю наличие на Slave ранее созданной БД на Master. База данных появилась, репликация работает:
![alt text](https://github.com/masterchoo495/Repl_p1/blob/main/004.png)

---

## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

---

### Задание 3* 

Выполните конфигурацию master-master репликации. Произведите проверку.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*
